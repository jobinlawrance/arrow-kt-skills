# 🎯 Arrow-kt Claude Skill

**The definitive Arrow-kt functional programming guide for Claude AI**

A production-ready Claude skill for idiomatic Arrow-kt patterns, with 100% test coverage and +82% improvement over baseline guidance.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Tests: 100%](https://img.shields.io/badge/Tests-100%25-brightgreen)](results/iteration-1-results.md)
[![Arrow-kt](https://img.shields.io/badge/Arrow--kt-Latest-blue)](https://arrow-kt.io)
[![Kotlin](https://img.shields.io/badge/Kotlin-1.9+-purple)](https://kotlinlang.org)

---

## What This Skill Does

This skill teaches Claude to provide expert guidance on Arrow-kt, a functional programming framework for Kotlin.

**When you ask Claude about Arrow-kt, this skill ensures you get:**
- ✅ Type-safe patterns (Either, Validated, Option, IO)
- ✅ Real-world examples (payment processing, user signup, API chaining)
- ✅ Best practices and common mistakes
- ✅ Framework integration (Spring Boot, Ktor)
- ✅ Clear explanations of when to use each pattern

---

## Features

### 🎓 Comprehensive Coverage
- **7 core patterns** with complete code examples
- **9 real-world scenarios** tested and validated
- **30+ assertions** ensuring quality
- **100% pass rate** across all test cases

### 🛠 Production Examples
- Payment processing with semantic error types
- User registration with async validation
- Parallel API loading with parZip
- Spring Boot @Service + @Controller integration
- Error transformation across layers

### ⚡ Pattern Guidance
- **Either vs Validated** - When to use each
- **either { }** suspend builder for async chains
- **parZip** for parallel execution
- **ensure()** for type-safe validation
- **mapLeft()** for error transformation
- **Option** as nullable alternative

### 📚 Learning Resource
- **Quick reference** for all Arrow types
- **Common mistakes** with fixes
- **Best practices** summary
- **Integration examples** for Spring Boot and Ktor

---

## Test Results

### ✅ 100% Pass Rate

Tested on 9 real-world scenarios:

| Scenario | Pass Rate | Example |
|----------|-----------|---------|
| Form validation | 100% | Show all field errors at once |
| Async chaining | 100% | DB + API + validation chain |
| Parallel execution | 100% | Fetch 3 APIs simultaneously |
| Either vs Validated | 100% | Clear comparison & decision |
| Error refactoring | 100% | Convert try/catch to Either |
| Payment processing | 100% | Real domain scenario |
| Option type | 100% | Use instead of nullable |
| Common mistakes | 100% | Identify & fix patterns |
| Spring Boot integration | 100% | @Service + @Controller |

### 📈 Improvement Over Baseline

```
Without skill (generic advice):    18% helpful
With skill (expert guidance):     100% helpful
Improvement:                      +82% 🎯
```

[See detailed test results →](results/iteration-1-results.md)

---

## Quick Start

### For Claude.ai Users

1. **Copy the skill content:**
   ```
   Copy everything from SKILL.md
   ```

2. **Create a new Claude chat**

3. **Paste the skill at the start**

4. **Ask questions:**
   - "How do I show all form validation errors at once?"
   - "Show me payment processing with Arrow"
   - "When should I use Either vs Validated?"

### For Claude API Users

```python
import anthropic

client = anthropic.Anthropic(api_key="...")

# Load the skill
with open("SKILL.md") as f:
    skill = f.read()

# Use in your conversation
response = client.messages.create(
    model="claude-opus-4-20250514",
    max_tokens=1024,
    system=skill,  # Inject skill as system context
    messages=[
        {
            "role": "user",
            "content": "How do I validate a form with all errors shown?"
        }
    ]
)

print(response.content[0].text)
```

### For Custom Claude Setups

See [INSTALLATION.md](INSTALLATION.md) for detailed setup instructions for various environments.

---

## Files in This Repo

```
arrow-kt-skills/
├── SKILL.md                          ← The actual skill (copy this!)
├── README.md                         ← You are here
├── INSTALLATION.md                   ← Setup guide
├── LICENSE                           ← MIT License
├── results/
│   └── iteration-1-results.md        ← Full test report
├── examples/
│   ├── payment-processing.kt         ← Payment domain example
│   ├── user-signup.kt                ← User registration example
│   ├── parallel-loading.kt           ← Parallel API example
│   └── spring-boot-integration.kt    ← Spring Boot example
└── program.md                        ← Development methodology
```

---

## How It Was Built

This skill was developed using an **autonomous improvement methodology** inspired by Karpathy's autoresearch pattern:

1. **Baseline Testing** - Tested generic Kotlin advice (18% pass rate)
2. **Weak Area Analysis** - Identified gaps in coverage
3. **Content Enhancement** - Added real examples (+199 lines)
4. **Iterative Validation** - Tested improved skill (100% pass rate)
5. **Results Documentation** - Documented all findings

**Improvement:** 18% → 100% (+82%)

See [program.md](program.md) and [iteration-1-results.md](results/iteration-1-results.md) for methodology details.

---

## Example Questions to Ask

### Basic Patterns
- "What's the difference between Either and Validated?"
- "When should I use Option instead of nullable types?"
- "How does the either {} suspend builder work?"

### Practical Scenarios
- "Show me how to handle an async database call and API call together"
- "How do I fetch 3 APIs in parallel with error handling?"
- "Refactor this try/catch code to use Arrow"

### Real-World Examples
- "Show me a complete payment processing flow with Arrow"
- "How do I use Either in Spring Boot @Service methods?"
- "What's the best pattern for user registration with validation?"

### Common Problems
- "I'm throwing exceptions in either {}. What should I do?"
- "How do I show ALL form field errors instead of stopping at the first?"
- "How should I transform errors when crossing layer boundaries?"

---

## Pattern Reference

### Core Types

**Either** - Fail-fast, single error
```kotlin
fun validateUser(user: User): Either<UserError, User> = either {
    validateName(user.name).bind()
    validateAge(user.age).bind()
    user
}
```

**Validated** - Accumulate all errors
```kotlin
fun validateForm(name: String, email: String) = mapN(
    validateName(name),
    validateEmail(email)
) { n, e -> FormData(n, e) }
```

**Option** - Nullable values functionally
```kotlin
Option.fromNullable(value)
    .flatMap { data -> process(data) }
    .getOrNull()
```

**parZip** - Parallel execution
```kotlin
parZip(
    { getUser(id) },
    { getPosts(id) },
    { getFriends(id) }
) { user, posts, friends -> UserProfile(user, posts, friends) }
```

### Common Patterns

- ✅ Form validation → Use **Validated + mapN**
- ✅ Multi-step chains → Use **Either + either {}**
- ✅ Optional values → Use **Option** not nullable
- ✅ Parallel ops → Use **parZip** not flatMap
- ✅ Conditional logic → Use **ensure()** not throw
- ✅ Error mapping → Use **mapLeft()** at boundaries

---

## Who Should Use This

- ✅ Kotlin developers learning Arrow-kt
- ✅ Teams adopting functional programming
- ✅ Spring Boot developers adding functional patterns
- ✅ Anyone wanting type-safe error handling
- ✅ Developers migrating from try/catch

---

## Community & Contributing

### Found an issue?
Open an [issue](https://github.com/jobinlawrance/arrow-kt-skills/issues)

### Have improvements?
Create a [pull request](https://github.com/jobinlawrance/arrow-kt-skills/pulls)

### Questions?
Start a [discussion](https://github.com/jobinlawrance/arrow-kt-skills/discussions)

### Running your own iteration?
See [program.md](program.md) for the autonomous improvement methodology.

---

## Resources

### Official
- **Arrow-kt Docs:** https://arrow-kt.io/docs/
- **GitHub:** https://github.com/arrow-kt/arrow-kt
- **Slack:** https://kotlinlang.slack.com (arrow-kt channel)

### Community
- **Kotlin Slack:** https://kotlinlang.slack.com
- **Arrow Discourse:** https://arrow-kt.io/community/
- **Reddit:** r/Kotlin

### Related
- **Valiktor Skill:** Alternative validation library
- **Autoresearch Pattern:** [Karpathy's autonomous research](https://github.com/karpathy/autoresearch)

---

## License

MIT License - See [LICENSE](LICENSE) file for details

Free for personal and commercial use. Attribution appreciated but not required.

---

## Changelog

### Version 1.0.0 (March 2026)
- ✅ Initial release with 7 patterns
- ✅ 100% test coverage
- ✅ 9 real-world scenarios
- ✅ Spring Boot integration
- ✅ Comprehensive documentation

---

## Get Started Now

1. **Copy [SKILL.md](SKILL.md)**
2. **Paste into Claude**
3. **Ask your first question** about Arrow-kt
4. **⭐ Star this repo** if it helps!

---

**Made with ❤️ for the Kotlin community**

Questions? Open an issue or reach out!

---

## Quick Links

- [Install Guide](INSTALLATION.md)
- [Test Results](results/iteration-1-results.md)
- [Methodology](program.md)
- [Code Examples](examples/)
