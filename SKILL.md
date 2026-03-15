---
name: arrow-kt-skills
description: Guide users in writing idiomatic Arrow-kt code. Trigger whenever the user asks about Arrow, functional Kotlin, type-safe error handling, Either/Validated/Option types, async validation, composing operations, resilience patterns, or wants to refactor Kotlin code to be more functional. Include this skill when helping with validation, error handling, result composition, parsing, effects management, or any functional programming pattern in Kotlin. Always suggest Arrow patterns when the user is dealing with operations that can fail or need to compose multiple steps.
---

# Arrow-kt Best Practices & Patterns

Arrow is a functional programming framework for Kotlin. This skill helps you write idiomatic, composable, type-safe code using Arrow abstractions.

## When to Use Arrow (Not Just Validation)

Arrow isn't just for validation. Use it when you need to:
- **Compose operations that can fail** — use `Either`/`Validated`
- **Handle nullable values functionally** — use `Option`
- **Manage side effects** — use `IO`/`Effect`
- **Run operations in parallel** — use `parTraverse`, `parZip`
- **Deep-update nested immutable data** — use Optics/Lenses
- **Parse or transform structured data** — use Parser Combinators
- **Retry failures, circuit breakers** — use Resilience patterns
- **Write generic code over abstractions** — use Type Classes

## Core Types: Quick Reference

### Either (Fail-fast, single error)
Use when you want to **stop at the first error**.

```kotlin
Either<Error, Success>
// Represents: this succeeded OR it failed with this error
```

**When to use:**
- One error per operation is enough
- You want to stop processing on first failure
- Mapping between error types is rare

**Pattern:**
```kotlin
fun validateUser(user: User): Either<UserError, User> = either {
    validateName(user.name).bind()
    validateAge(user.age).bind()
    validateEmail(user.email).bind()
    user
}
```

### Validated (Accumulate errors)
Use when you want to **collect ALL errors at once**.

```kotlin
Validated<E, A>
// Can accumulate multiple errors via Semigroup
```

**When to use:**
- Form validation (show all field errors at once)
- You want ALL problems reported simultaneously
- Errors are of the same type or combine naturally

**Pattern:**
```kotlin
fun validateUser(user: User): Validated<Nel<UserError>, User> = mapN(
    validateName(user.name),
    validateAge(user.age),
    validateEmail(user.email)
) { name, age, email -> User(name, age, email) }
```

### Option (Nullable values functionally)
Use instead of nullable types for **optional values**.

```kotlin
Some(value) | None  // No null pointer exceptions
```

**When to use:**
- Instead of `T?` (nullable types)
- Chains of operations where any step might be missing
- Want functional operations on optional values

**Pattern:**
```kotlin
val email = getUser(id)
    .flatMap { user -> getEmail(user.id) }
    .map { email -> email.toLowerCase() }
    .getOrNull()  // Null only if explicitly converted
```

### IO (Deferred side effects)
Use to **describe computations without running them**.

```kotlin
IO<A>  // A computation that produces A, but doesn't run yet
```

**When to use:**
- Testing (run effects only in production)
- Parallel execution (compose multiple IOs, run together)
- Pure functional code (side effects deferred to edge)

**Pattern:**
```kotlin
fun getUser(id: String): IO<User> = IO {
    // This code doesn't run until unsafeRunAsync()
    api.call("/users/$id").toUser()
}

// Compose
fun getUserAndPosts(id: String): IO<Pair<User, Posts>> = IO {
    val user = getUser(id).unsafeRun()
    val posts = getPosts(id).unsafeRun()
    user to posts
}
```

---

## Essential Patterns

### Pattern 1: Fail-Fast Chaining (Either + either {})

```kotlin
// BAD: Imperative, exception-based
fun processOrder(request: OrderRequest): Order {
    try {
        request.validate()
        val items = request.items.map { fetchItem(it) }
        val total = items.sumOf { it.price }
        return Order(items, total)
    } catch (e: Exception) {
        throw RuntimeException("Failed", e)
    }
}

// GOOD: Functional, explicit error handling
sealed class OrderError {
    data class ValidationFailed(val msg: String) : OrderError()
    object ItemNotFound : OrderError()
}

fun processOrder(request: OrderRequest): Either<OrderError, Order> = either {
    validateRequest(request).bind()
    val items = request.items.map { fetchItem(it).bind() }
    val total = items.sumOf { it.price }
    Order(items, total)
}
```

**Key points:**
- `.bind()` automatically unwraps on success, stops on first failure
- Type signature documents what can fail
- No try/catch clutter

---

### Pattern 2: Collect All Errors (Validated + mapN)

```kotlin
// GOOD for forms: show ALL errors at once
sealed class FormError {
    object NameBlank : FormError()
    object EmailInvalid : FormError()
    object PasswordWeak : FormError()
}

fun validateSignup(name: String, email: String, password: String): 
    Validated<Nel<FormError>, SignupData> = mapN(
    validateName(name),
    validateEmail(email),
    validatePassword(password)
) { n, e, p -> SignupData(n, e, p) }

// Usage
when (val result = validateSignup(name, email, pwd)) {
    is Valid -> submit(result.value)
    is Invalid -> result.value.forEach { error ->  // All errors shown
        when (error) {
            FormError.NameBlank -> showNameError()
            FormError.EmailInvalid -> showEmailError()
            FormError.PasswordWeak -> showPasswordError()
        }
    }
}
```

---

### Pattern 3: Conditional/Optional Steps

```kotlin
// Use Option for truly optional data
fun loadUserData(userId: String): Option<User> = Option.fromNullable(
    database.getUser(userId)
)

// Use Option.flatMap to chain optional operations
fun getUserPreferences(userId: String): Option<Preferences> {
    return loadUserData(userId)
        .flatMap { user -> loadPreferences(user.id) }
        .filter { it.isValid() }
}

// For conditional validation logic, use Either.mapLeft + ensure
fun validateAge(age: Int): Either<AgeError, Int> = either {
    ensure(age >= 0) { AgeError.Negative }
    ensure(age <= 150) { AgeError.Unrealistic }
    age
}
```

---

### Pattern 4: Async With Suspend (coroutine-friendly)

```kotlin
// Arrow suspend builders work seamlessly with async code
suspend fun registerUser(request: SignupRequest): Either<SignupError, User> = either {
    validateRequest(request).bind()
    
    val existing = userRepository.findByEmail(request.email)
        .mapLeft { SignupError.DatabaseError }
        .bind()
    
    ensure(existing == null) { SignupError.EmailExists }
    
    val user = User.fromRequest(request)
    userRepository.save(user)
        .mapLeft { SignupError.DatabaseError }
        .bind()
    
    user
}

// Calling code
launch {
    when (val result = registerUser(request)) {
        is Either.Right -> navigateToHome()
        is Either.Left -> showError(result.value)
    }
}
```

---

### Pattern 5: Parallel Execution (parTraverse, parZip)

```kotlin
// Run multiple async operations in parallel
data class UserProfile(val user: User, val posts: List<Post>, val friends: List<User>)

sealed class ProfileError {
    object UserNotFound : ProfileError()
    object PostsUnavailable : ProfileError()
    object FriendsUnavailable : ProfileError()
}

// Async operation returning IO (deferred computation)
fun getUserIO(id: String): IO<Either<ProfileError, User>> = IO {
    api.getUser(id)  // Expensive operation
}

fun getPostsIO(userId: String): IO<Either<ProfileError, List<Post>>> = IO {
    api.getPosts(userId)  // Expensive operation
}

fun getFriendsIO(userId: String): IO<Either<ProfileError, List<User>>> = IO {
    api.getFriends(userId)  // Expensive operation
}

// RUN ALL 3 IN PARALLEL automatically
fun loadUserProfile(userId: String): IO<Either<ProfileError, UserProfile>> {
    return parZip(
        { getUserIO(userId) },      // Fetch user
        { getPostsIO(userId) },     // Fetch posts (in parallel!)
        { getFriendsIO(userId) }    // Fetch friends (in parallel!)
    ) { userResult, postsResult, friendsResult ->
        // Combine results: if any failed, Either propagates the failure
        either {
            val user = userResult.bind()
            val posts = postsResult.bind()
            val friends = friendsResult.bind()
            UserProfile(user, posts, friends)
        }
    }
}

// Usage - everything runs in parallel
val profile = loadUserProfile("user123").unsafeRunAsync()

// Result: all 3 API calls happen at the same time (not sequentially)
// If any fails, we get that error. If all succeed, we get the combined result.
```

**Key points:**
- `parZip` runs operations **in parallel**, not sequentially
- All 3 API calls happen at the same time
- If any operation fails, that error is returned
- Much faster than `.flatMap` chaining (which runs sequentially)

---

### Pattern 6: Retry Logic

```kotlin
sealed class ApiError {
    object Timeout : ApiError()
    object ServerError : ApiError()
}

fun callApiWithRetry(url: String): Either<ApiError, Response> = either {
    retryWithBackoff(
        maxRetries = 3,
        initialDelay = 100.milliseconds,
        maxDelay = 5.seconds,
        backoffMultiplier = 2.0
    ) {
        callApi(url).mapLeft { ApiError.Timeout }
    }.bind()
}
```

---

### Pattern 7: Transform Errors Across Layers

```kotlin
// Domain layer
sealed class DomainError {
    object UserNotFound : DomainError()
}

// API layer
sealed class ApiError {
    data class Http404(val msg: String) : ApiError()
}

// Transformation at boundary
fun getUserDomain(id: String): Either<DomainError, User> {
    return getUser(id)
        .mapLeft { apiError ->
            when (apiError) {
                is ApiError.Http404 -> DomainError.UserNotFound
            }
        }
}
```

---

## Real-World Example: Payment Processing with Arrow

This is how you'd structure a complete payment processing flow with Arrow:

```kotlin
// Define semantic domain errors
sealed class PaymentError {
    data class ValidationFailed(val reason: String) : PaymentError()
    data class InsufficientFunds(val available: Double, val requested: Double) : PaymentError()
    data class ProcessorError(val code: String) : PaymentError()
    data class DatabaseError(val msg: String) : PaymentError()
}

data class PaymentReceipt(val transactionId: String, val amount: Double, val timestamp: Long)

// Service layer - pure business logic
class PaymentService(
    val cardProcessor: CardProcessor,
    val accountRepository: AccountRepository,
    val transactionRepository: TransactionRepository
) {
    fun processPayment(request: PaymentRequest): Either<PaymentError, PaymentReceipt> = either {
        // Validate request
        validatePaymentRequest(request)
            .mapLeft { PaymentError.ValidationFailed(it) }
            .bind()
        
        // Check balance
        val account = accountRepository.getAccount(request.accountId)
            .mapLeft { PaymentError.DatabaseError(it.message ?: "Unknown") }
            .bind()
        
        // Ensure sufficient funds
        ensure(account.balance >= request.amount) {
            PaymentError.InsufficientFunds(account.balance, request.amount)
        }
        
        // Charge the card
        val transaction = cardProcessor.charge(request.cardToken, request.amount)
            .mapLeft { PaymentError.ProcessorError(it.code) }
            .bind()
        
        // Save to database
        transactionRepository.save(transaction)
            .mapLeft { PaymentError.DatabaseError(it.message ?: "Unknown") }
            .bind()
        
        PaymentReceipt(transaction.id, transaction.amount, System.currentTimeMillis())
    }
}

// Controller layer - handle different error types appropriately
@RestController
@RequestMapping("/payments")
class PaymentController(val paymentService: PaymentService) {
    @PostMapping
    suspend fun processPayment(@RequestBody request: PaymentRequest): ResponseEntity<*> {
        return when (val result = paymentService.processPayment(request)) {
            is Either.Right -> {
                logger.info("Payment processed: ${result.value.transactionId}")
                ResponseEntity.ok(result.value)
            }
            is Either.Left -> when (val error = result.value) {
                is PaymentError.ValidationFailed -> {
                    ResponseEntity.badRequest().body(mapOf("error" to error.reason))
                }
                is PaymentError.InsufficientFunds -> {
                    ResponseEntity.status(402)  // Payment Required
                        .body(mapOf("error" to "Insufficient funds. Need \$${error.requested - error.available} more"))
                }
                is PaymentError.ProcessorError -> {
                    logger.error("Card processor error: ${error.code}")
                    // Implement retry logic here
                    ResponseEntity.status(503).body(mapOf("error" to "Payment processing unavailable, please try again"))
                }
                is PaymentError.DatabaseError -> {
                    logger.error("Database error: ${error.msg}")
                    ResponseEntity.status(500).body(mapOf("error" to "Server error"))
                }
            }
        }
    }
}

// Key patterns shown:
// 1. Sealed class errors for semantic error handling
// 2. mapLeft for error transformation across layers
// 3. ensure() for validation within Either
// 4. when() to pattern-match on error types
// 5. Different HTTP responses based on error type
// 6. Logging at appropriate levels
```

---

## Common Mistakes & Fixes

### ❌ Mistake 1: Forgetting `.bind()` in either {}

```kotlin
// WRONG: returns nested Either
fun process(): Either<Error, Result> = either {
    validateInput(data)  // Returns Either, not unwrapped!
    data
}

// RIGHT: use .bind()
fun process(): Either<Error, Result> = either {
    validateInput(data).bind()  // Unwrapped
    data
}
```

### ❌ Mistake 2: Using Either when you need Validated

```kotlin
// WRONG for forms: only shows first field error
fun validateForm(name: String, email: String): Either<Error, FormData> = either {
    validateName(name).bind()  // Stops here if invalid
    validateEmail(email).bind()
    FormData(name, email)
}

// RIGHT for forms: shows all field errors
fun validateForm(name: String, email: String): 
    Validated<Nel<Error>, FormData> = mapN(
    validateName(name),
    validateEmail(email)
) { n, e -> FormData(n, e) }
```

### ❌ Mistake 3: Throwing exceptions when you have Either

```kotlin
// WRONG: loses the Either semantics
fun process(): Either<Error, Result> = either {
    val user = getUser(id).bind()
    if (user.age < 18) throw IllegalArgumentException("Too young")  // ❌
    user
}

// RIGHT: use ensure
fun process(): Either<Error, Result> = either {
    val user = getUser(id).bind()
    ensure(user.age >= 18) { AgeError.TooYoung }
    user
}
```

### ❌ Mistake 4: Mixing error types carelessly

```kotlin
// WRONG: Either<Exception, T> is too generic
fun process(): Either<Exception, Result> = TODO()

// RIGHT: Use sealed class for semantic errors
sealed class ProcessError {
    data class Invalid(val reason: String) : ProcessError()
    object PermissionDenied : ProcessError()
}

fun process(): Either<ProcessError, Result> = TODO()
```

---

## When to Use Arrow vs Alternatives

| Scenario | Use Arrow | Why |
|----------|-----------|-----|
| **Input validation** | `Validated` | Collect all errors at once |
| **Multi-step operation** | `Either` + `either {}` | Clean chaining, fail-fast |
| **Optional data** | `Option` | Better than nullable types |
| **Async operations** | `Either` + `suspend` | Natural with coroutines |
| **Side effects** | `IO` | Purely functional, testable |
| **Parallel execution** | `parTraverse`, `parZip` | Automatic parallelism |
| **Recursive data** | Recursion schemes | Elegant tree walking |
| **Deep updates** | Optics/Lenses | Type-safe nested updates |

---

## Common Built-in Functions

```kotlin
// Either operations
either<E, A> { /* code */ }          // Suspend builder
result.bind()                         // Unwrap or short-circuit
result.map { }                        // Transform success
result.mapLeft { }                    // Transform error
result.fold(ifLeft, ifRight)          // Extract value
result.getOrNull()                    // Convert to nullable
result.getOrElse { default }          // Provide default
result.recover { error -> ... }       // Recover from error

// Validated operations
mapN(v1, v2, v3) { a, b, c -> ... }  // Combine validations
valid<E, A>(a)                        // Success
invalid<E, A>(e)                      // Failure
result.fold(ifInvalid, ifValid)       // Extract value

// Option operations
Option.fromNullable(value)            // Convert nullable
option.flatMap { }                    // Chain operations
option.getOrNull()                    // Extract or null
option.getOrElse { default }          // Provide default
```

---

## Best Practices Summary

1. **Use sealed classes for errors** — enables compiler to check exhaustiveness
   ```kotlin
   sealed class MyError { object A : MyError(); object B : MyError() }
   when (error) { is MyError.A -> ...; is MyError.B -> ... }  // Compiler checks completeness
   ```

2. **Make errors explicit in function signatures** — type signature documents failure modes
   ```kotlin
   fun process(data: Data): Either<ProcessError, Result>  // Clear what can fail
   ```

3. **Use `either {}` for readable chaining** — reads like imperative but is functional
   ```kotlin
   either {
       step1().bind()
       step2().bind()
       step3()
   }
   ```

4. **Use `Validated` for forms** — show all validation errors at once
   ```kotlin
   mapN(validateField1(), validateField2(), ...) { f1, f2, ... -> ... }
   ```

5. **Prefer `Option` over nullable types** — functional operations on optional values
   ```kotlin
   Option.fromNullable(value).map { ... }.flatMap { ... }
   ```

6. **Transform errors at boundaries** — domain layer shouldn't know about HTTP errors
   ```kotlin
   getUser().mapLeft { apiError -> convertToDomainError(apiError) }
   ```

7. **Compose effects functionally** — use `parTraverse`, `parZip` for parallelism
   ```kotlin
   listOf(id1, id2, id3).parTraverse { getUser(it) }  // Runs in parallel
   ```

---

## Integration Examples

### With Spring Boot (Detailed)

**Service layer returns Either:**
```kotlin
sealed class UserError {
    object NotFound : UserError()
    object EmailAlreadyExists : UserError()
    object InvalidEmail : UserError()
}

@Service
class UserService(val userRepository: UserRepository) {
    fun createUser(request: CreateUserRequest): Either<UserError, User> = either {
        // Validate
        ensure(request.email.contains("@")) { UserError.InvalidEmail }
        
        // Check if exists
        val existing = userRepository.findByEmail(request.email)
        ensure(existing == null) { UserError.EmailAlreadyExists }
        
        // Create and save
        val user = User(request.email, request.name)
        userRepository.save(user)
        user
    }
}
```

**Controller handles Either, transforms to HTTP:**
```kotlin
@RestController
@RequestMapping("/users")
class UserController(val userService: UserService) {
    @PostMapping
    suspend fun createUser(@RequestBody request: CreateUserRequest): ResponseEntity<UserResponse> {
        return when (val result = userService.createUser(request)) {
            is Either.Right -> ResponseEntity
                .status(HttpStatus.CREATED)
                .body(result.value.toResponse())
            
            is Either.Left -> when (val error = result.value) {
                UserError.InvalidEmail -> ResponseEntity
                    .badRequest()
                    .body(UserResponse.Error("Invalid email format"))
                
                UserError.EmailAlreadyExists -> ResponseEntity
                    .status(HttpStatus.CONFLICT)
                    .body(UserResponse.Error("Email already registered"))
                
                UserError.NotFound -> ResponseEntity
                    .notFound()
                    .build()
            }
        }
    }
}
```

**Or use @ControllerAdvice for global error handling:**
```kotlin
@ControllerAdvice
class ErrorHandler {
    @ExceptionHandler
    fun handleUserErrors(error: UserError): ResponseEntity<ErrorResponse> {
        return when (error) {
            UserError.InvalidEmail -> ResponseEntity
                .badRequest()
                .body(ErrorResponse("Invalid email", "INVALID_EMAIL"))
            
            UserError.EmailAlreadyExists -> ResponseEntity
                .status(HttpStatus.CONFLICT)
                .body(ErrorResponse("Email exists", "EMAIL_EXISTS"))
            
            UserError.NotFound -> ResponseEntity
                .notFound()
                .build()
        }
    }
}
```

### With Ktor
```kotlin
post("/users") {
    val request = call.receive<CreateUserRequest>()
    when (val result = userService.create(request)) {
        is Either.Right -> call.respond(HttpStatusCode.Created, result.value)
        is Either.Left -> call.respond(HttpStatusCode.BadRequest, result.value)
    }
}
```

### With Testing
```kotlin
@Test
fun testValidation() {
    val invalid = User("", -5)
    val result = validateUser(invalid)
    
    assertTrue(result is Either.Left)
    result.fold(
        ifLeft = { error -> assertTrue(error is UserError.Invalid) },
        ifRight = { fail("Should not be valid") }
    )
}
```

---

## Reference

- **Docs:** https://arrow-kt.io/docs/
- **GitHub:** https://github.com/arrow-kt/arrow-kt
- **Slack Community:** https://kotlinlang.slack.com (arrow-kt channel)
