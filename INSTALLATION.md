# Installation Guide

How to install and use the Arrow-kt Claude skill in different environments.

---

## Table of Contents

1. [Claude.ai Web](#claudeai-web)
2. [Claude API (Python)](#claude-api-python)
3. [Claude API (Node.js)](#claude-api-nodejs)
4. [Custom Claude Projects](#custom-claude-projects)
5. [Troubleshooting](#troubleshooting)

---

## Claude.ai Web

The simplest way to use this skill.

### Step 1: Copy the Skill

Visit the repo and copy the content of [SKILL.md](SKILL.md):

```
https://github.com/yourusername/arrow-kt-kotlin-patterns/blob/main/SKILL.md
```

Click the copy button or select all and copy.

### Step 2: Create a New Chat

Go to [Claude.ai](https://claude.ai) and click **"New Chat"**

### Step 3: Paste the Skill

Paste the SKILL.md content at the very beginning of your message, then press Enter.

### Step 4: Start Asking

```
Now ask your Arrow-kt questions:

"How do I validate a form with all errors shown at once?"
"Show me a payment processing example with Arrow"
"When should I use Either vs Validated?"
```

**That's it!** Claude will now provide expert Arrow-kt guidance.

---

## Claude API (Python)

For programmatic access to Claude with the skill.

### Installation

```bash
# Install the Anthropic Python SDK
pip install anthropic
```

### Basic Usage

```python
import anthropic

# Initialize client
client = anthropic.Anthropic(api_key="your-api-key-here")

# Load the skill
with open("SKILL.md") as f:
    skill_content = f.read()

# Create a message with the skill
message = client.messages.create(
    model="claude-opus-4-20250514",
    max_tokens=1024,
    system=skill_content,  # Inject skill as system prompt
    messages=[
        {
            "role": "user",
            "content": "How do I validate a form with all errors shown?"
        }
    ]
)

# Print the response
print(message.content[0].text)
```

### Advanced Usage

```python
import anthropic

client = anthropic.Anthropic()

# Load skill once
with open("SKILL.md") as f:
    skill = f.read()

# Reuse in multiple conversations
def ask_about_arrow(question: str) -> str:
    response = client.messages.create(
        model="claude-opus-4-20250514",
        max_tokens=2048,
        system=skill,
        messages=[
            {"role": "user", "content": question}
        ]
    )
    return response.content[0].text

# Ask multiple questions
questions = [
    "Show me a payment processing example",
    "When should I use parZip?",
    "How do I handle errors in Spring Boot?"
]

for q in questions:
    answer = ask_about_arrow(q)
    print(f"Q: {q}")
    print(f"A: {answer}\n")
```

### With Conversation History

```python
import anthropic

client = anthropic.Anthropic()

with open("SKILL.md") as f:
    skill = f.read()

# Maintain conversation history
conversation_history = []

def chat_with_skill(user_message: str) -> str:
    """Continue a multi-turn conversation with the skill."""
    # Add user message to history
    conversation_history.append({
        "role": "user",
        "content": user_message
    })
    
    # Get response
    response = client.messages.create(
        model="claude-opus-4-20250514",
        max_tokens=2048,
        system=skill,
        messages=conversation_history
    )
    
    # Add assistant response to history
    assistant_message = response.content[0].text
    conversation_history.append({
        "role": "assistant",
        "content": assistant_message
    })
    
    return assistant_message

# Have a conversation
print(chat_with_skill("What is Validated used for?"))
print(chat_with_skill("Show me an example with 5 fields")))
print(chat_with_skill("How is this different from Either?")))
```

### Environment Variable Setup

```bash
# Set your API key as environment variable
export ANTHROPIC_API_KEY="your-api-key-here"

# Python will automatically use it
# No need to pass api_key in the code
```

---

## Claude API (Node.js)

For JavaScript/TypeScript projects.

### Installation

```bash
# Install the Anthropic SDK
npm install @anthropic-ai/sdk
```

### Basic Usage

```javascript
import Anthropic from "@anthropic-ai/sdk";
import * as fs from "fs";

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// Load the skill
const skillContent = fs.readFileSync("SKILL.md", "utf-8");

// Create a message
const message = await client.messages.create({
  model: "claude-opus-4-20250514",
  max_tokens: 1024,
  system: skillContent,
  messages: [
    {
      role: "user",
      content: "How do I validate a form with all errors shown?",
    },
  ],
});

console.log(message.content[0].type === "text" ? message.content[0].text : "");
```

### With TypeScript

```typescript
import Anthropic from "@anthropic-ai/sdk";
import * as fs from "fs";

const client = new Anthropic();
const skillContent = fs.readFileSync("SKILL.md", "utf-8");

async function askAboutArrow(question: string): Promise<string> {
  const response = await client.messages.create({
    model: "claude-opus-4-20250514",
    max_tokens: 2048,
    system: skillContent,
    messages: [
      {
        role: "user",
        content: question,
      },
    ],
  });

  const textContent = response.content.find((block) => block.type === "text");
  return textContent && textContent.type === "text" ? textContent.text : "";
}

// Usage
const answer = await askAboutArrow("Show me a payment processing example");
console.log(answer);
```

### Streaming Responses

```javascript
const stream = await client.messages.stream({
  model: "claude-opus-4-20250514",
  max_tokens: 2048,
  system: skillContent,
  messages: [
    {
      role: "user",
      content: "Explain parZip for parallel execution",
    },
  ],
});

// Stream text as it arrives
for await (const event of stream) {
  if (
    event.type === "content_block_delta" &&
    event.delta.type === "text_delta"
  ) {
    process.stdout.write(event.delta.text);
  }
}
```

---

## Custom Claude Projects

If you're using Claude in your own application or LLM framework.

### Generic Integration Pattern

```
1. Load SKILL.md as a string
2. Include it in your system prompt
3. All subsequent messages use the skill

Example:
system_prompt = load_file("SKILL.md")
response = llm.complete(
    system=system_prompt,
    messages=user_messages
)
```

### Spring Boot (Java)

```java
import software.amazon.awssdk.services.bedrockruntime.BedrockRuntimeClient;

public class ArrowKtSkillClient {
    private String skillContent;
    
    public ArrowKtSkillClient() throws IOException {
        // Load skill
        this.skillContent = Files.readString(
            Paths.get("SKILL.md")
        );
    }
    
    public String askAboutArrow(String question) {
        // Use skill in your Claude integration
        return callClaudeWithSystem(skillContent, question);
    }
    
    private String callClaudeWithSystem(String system, String userMessage) {
        // Your Claude integration here
        // (e.g., using AWS Bedrock SDK)
        return null;
    }
}
```

### Langchain (Python)

```python
from langchain.chat_models import ChatAnthropic
from langchain.prompts.chat import SystemMessagePromptTemplate

# Load skill
with open("SKILL.md") as f:
    skill = f.read()

# Create system prompt
system_message_prompt = SystemMessagePromptTemplate.from_template(skill)

# Use in chain
from langchain.chains import LLMChain
from langchain.prompts.chat import ChatPromptTemplate

chat_prompt = ChatPromptTemplate.from_messages(
    [system_message_prompt, HumanMessagePromptTemplate.from_template("{input}")]
)

llm = ChatAnthropic(model="claude-opus-4-20250514")
chain = LLMChain(llm=llm, prompt=chat_prompt)

# Ask question
result = chain.run(input="How do I use parZip?")
print(result)
```

### LangChain JS

```typescript
import { ChatAnthropic } from "@langchain/anthropic";
import { readFileSync } from "fs";

const skillContent = readFileSync("SKILL.md", "utf-8");

const chat = new ChatAnthropic({
  modelName: "claude-opus-4-20250514",
  systemPrompt: skillContent,
});

const response = await chat.call([
  {
    role: "user",
    content: "Show me a payment processing example with Arrow",
  },
]);

console.log(response.content);
```

---

## Troubleshooting

### Q: Nothing changes when I paste the skill

**A:** Make sure you're pasting the SKILL.md content at the **beginning** of your first message, not in a follow-up message.

### Q: Claude doesn't mention Arrow patterns

**A:** Try being more specific:
```
Instead of: "How do I validate?"
Try: "Show me how to use Validated from Arrow-kt to validate a form"
```

### Q: The API key isn't working

**A:** Check your API key:
```bash
# Verify it's set
echo $ANTHROPIC_API_KEY

# If not, set it
export ANTHROPIC_API_KEY="sk-ant-..."
```

### Q: I'm getting rate limits

**A:** You might be hitting rate limits. Wait a moment and retry:
```python
import time
time.sleep(2)  # Wait 2 seconds
response = client.messages.create(...)
```

### Q: The skill isn't in the latest Claude version

**A:** The skill works with any Claude version. If you want the latest Claude features, update your model name:
```python
model="claude-opus-4-20250514"  # Latest at time of publication
```

Check [Claude API docs](https://docs.anthropic.com) for current model names.

### Q: I want to modify the skill

**A:** You can! Edit SKILL.md and use your modified version. Consider:
1. Adding custom examples for your domain
2. Removing sections you don't need
3. Adding links to your internal documentation

### Q: How do I use this with my custom LLM?

**A:** The skill is just markdown/text. You can:
1. Load SKILL.md as a string
2. Include it in your system prompt
3. Use with any LLM that supports system prompts

```python
# Pseudo-code for any LLM
system_prompt = read_file("SKILL.md")
response = your_llm.complete(
    system=system_prompt,
    user_message=user_input
)
```

---

## Next Steps

- **Try it out:** Start with the Quick Start section above
- **Ask questions:** Visit the [discussions](https://github.com/yourusername/arrow-kt-kotlin-patterns/discussions)
- **Report issues:** [Open an issue](https://github.com/yourusername/arrow-kt-kotlin-patterns/issues)
- **Contribute:** [Submit a pull request](https://github.com/yourusername/arrow-kt-kotlin-patterns/pulls)

---

## More Resources

- [SKILL.md](SKILL.md) - The actual skill content
- [README.md](README.md) - Overview and features
- [Examples](examples/) - Code examples
- [Test Results](results/iteration-1-results.md) - Validation results

---

Happy coding with Arrow-kt! 🎯
