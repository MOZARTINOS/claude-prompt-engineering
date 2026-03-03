---
name: claude-prompt-engineering
description: "Expert prompt engineering skill for creating optimized prompts for any task, model, and use case. Use when asked to write, create, or improve prompts, system prompts, or AI instructions."
---

# Claude Prompt Engineering — Expert Prompt Craft

> **Sources**: Anthropic [Interactive Tutorial](https://github.com/anthropics/prompt-eng-interactive-tutorial) (9 chapters + 3 appendices), [Claude Prompting Best Practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices), [Prompting Tools](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-tools).
>
> **Note**: Model-specific advice reflects Claude 4.x (2025–2026). Check [Anthropic docs](https://platform.claude.com/docs) for current API parameters. Framework synthesized from official Anthropic materials.

## When This Skill Activates

Trigger phrases (any language):
- "write/create/make/build a prompt", "prompt for...", "prompt template"
- "system prompt for...", "instructions for AI/bot/agent"
- "промпт", "промт", "напиши промпт", "сделай промпт", "инструкция для бота"
- "GPT prompt", "Claude prompt", "LLM prompt"
- "chatbot instructions", "AI instructions"
- Any request that implies creating text to instruct an AI model

## Prompt Creation Protocol

### Phase 0: Context Gathering (AUTOMATIC — run silently before anything else)

When this skill activates, **immediately and in parallel** gather context from the current session. Do NOT ask the user for this — extract it yourself from what's already available.

#### What to detect

| Signal | Where to look | How it affects the prompt |
|--------|---------------|--------------------------|
| **Project type** | `package.json`, `requirements.txt`, `go.mod`, `*.py`, `*.ts`, CLAUDE.md | Determines domain language, code style, relevant APIs |
| **Tech stack** | Dependencies, imports, framework configs | Informs which tools/integrations to reference |
| **Current task** | Conversation history, recent tool calls, user's last messages | Defines the GOAL the prompt should serve |
| **Target platform** | Mentioned in chat: "n8n", "API", "chatbot", "Telegram bot", "web app" | Determines prompt format (system/user, webhook, agent) |
| **User's language** | Last 3 messages from user — what language are they in? | Prompt instructions should match user's language |
| **Existing prompts** | CLAUDE.md, `.cursorrules`, system prompts in codebase | Match existing style, avoid contradictions |
| **Data examples** | Recent file reads, API responses, DB schemas in context | Use as real few-shot examples instead of placeholders |

#### Detection protocol

```
1. Check conversation history for:
   - What the user is building (project goal)
   - What problem they're solving (immediate task)
   - What they've already tried (avoid duplication)
   - Any constraints mentioned (model, budget, latency)

2. If in a code project, scan for:
   - CLAUDE.md / .cursorrules (existing AI instructions)
   - Main config files (package.json, pyproject.toml, etc.)
   - Any existing prompt files or system prompts in the codebase

3. Infer from context (don't ask if obvious):
   - Target model: if they're in Claude Code → Claude; if code mentions openai → GPT
   - Format: if building n8n workflow → Code node format; if API → system+user
   - Persona: match the domain of the project (fintech → financial analyst, etc.)
   - Output format: if downstream is JSON API → JSON; if human-facing → prose
```

#### Context injection rules

- **DO** use real data from the session as few-shot examples when available
- **DO** match the coding style and naming conventions of the project
- **DO** reference specific APIs, tools, or schemas from the codebase
- **DO** always write the prompt body (instructions, structure, XML tags) in **English** — all models perform best with English instructions. If the user needs the AI's *response* in another language, append `Respond in [language].` at the end of the prompt
- **DON'T** ask questions you can answer from the session context
- **DON'T** use generic placeholders when real data is available
- **DON'T** ignore existing CLAUDE.md rules — the new prompt must be compatible

### Phase 1: Clarify (ASK only what can't be inferred)

After Phase 0, check what's still unknown. Only ask about gaps that can't be inferred:

| Question | Ask only if... |
|----------|----------------|
| **What model?** | Not obvious from project context (no SDK imports, no CLAUDE.md) |
| **What task?** | User request is genuinely ambiguous |
| **Where will it run?** | Multiple valid platforms possible |
| **Who is the end user?** | Could be developer OR end-user facing |
| **What format is expected?** | Multiple formats make sense for the task |
| **Are there examples?** | None found in context AND task is complex |

If everything is clear from context — **skip Phase 1 entirely and go straight to building**.

### Phase 2: Build Using the 10-Part Framework

Apply ALL relevant elements (not every prompt needs all 10):

```
┌─────────────────────────────────────────────────┐
│  1. ROLE / PERSONA                              │
│     System prompt. Who is the AI?               │
│     "You are a senior tax accountant with       │
│      15 years of experience in US tax law."     │
├─────────────────────────────────────────────────┤
│  2. TASK CONTEXT                                │
│     Why does this task exist? Background.        │
│     "Users submit quarterly expense reports.     │
│      You help categorize and flag anomalies."   │
├─────────────────────────────────────────────────┤
│  3. TONE / STYLE                                │
│     Communication style constraints.            │
│     "Professional but approachable.             │
│      Avoid jargon. Explain like to a client."   │
├─────────────────────────────────────────────────┤
│  4. DETAILED INSTRUCTIONS & RULES               │
│     Step-by-step behavior. Edge cases.          │
│     Numbered list when order matters.           │
│     "If the expense is over $10,000, flag it.   │
│      If category is ambiguous, ask the user."   │
├─────────────────────────────────────────────────┤
│  5. EXAMPLES (Few-Shot)                          │
│     3-5 input/output pairs in <example> tags.   │
│     Cover: typical case, edge case, refusal.    │
│     THE SINGLE MOST EFFECTIVE TECHNIQUE.        │
├─────────────────────────────────────────────────┤
│  6. INPUT DATA                                   │
│     Variable content in XML tags.               │
│     <document>{{CONTENT}}</document>            │
│     <user_query>{{QUERY}}</user_query>          │
├─────────────────────────────────────────────────┤
│  7. IMMEDIATE TASK (reiteration)                 │
│     Restate the exact task near the end.        │
│     Critical for long prompts (>2K tokens).     │
├─────────────────────────────────────────────────┤
│  8. THINKING / REASONING                         │
│     "Think step by step before answering."      │
│     Use <thinking> tags to separate reasoning.  │
│     Place BEFORE the final task instruction.    │
├─────────────────────────────────────────────────┤
│  9. OUTPUT FORMAT                                │
│     Exact format spec. XML tags, JSON schema,   │
│     markdown structure, length constraints.     │
│     Place near the end of the prompt.           │
├─────────────────────────────────────────────────┤
│ 10. PREFILL / STRUCTURED OUTPUT                  │
│     For Claude 4.x: use Structured Outputs      │
│     or tool calling with enum fields.           │
│     For older models: prefill assistant turn.   │
│     For GPT: use response_format / json_schema. │
└─────────────────────────────────────────────────┘
```

### Phase 3: Apply Techniques by Task Type

#### Classification / Labeling
```xml
<system>
You are a [domain] classifier. Classify the input into exactly one of these categories: [list].
</system>

<examples>
<example>
<input>[sample input 1]</input>
<output>[category]</output>
</example>
<example>
<input>[edge case input]</input>
<output>[category with brief reasoning]</output>
</example>
</examples>

Classify the following:
<input>{{USER_INPUT}}</input>

Respond with ONLY the category name.
```

#### Information Extraction
```xml
<system>
You are a data extraction specialist. Extract structured information from unstructured text.
</system>

Extract the following fields from the document below:
- Name
- Date
- Amount
- Status

<document>{{DOCUMENT}}</document>

First, find relevant quotes from the document in <quotes> tags.
Then provide the extracted data in <result> tags as JSON.
```

#### Creative Generation
```xml
<system>
You are a [creative role] known for [specific style traits].
</system>

Write a [format] about [topic].

Requirements:
- Length: [specific word/paragraph count]
- Tone: [specific tone]
- Must include: [required elements]
- Must avoid: [exclusions]

<examples>
<example>
[One complete example in the target style]
</example>
</examples>
```

#### Chatbot / Conversational Agent
```xml
<system>
You are [persona with detailed background].

Your task: [primary purpose].

Rules:
1. [Behavior rule — what to do]
2. [Boundary — what NOT to do]
3. [Fallback — what to do when unsure]
4. [Tone — how to communicate]

When you don't know the answer, say: "[specific fallback phrase]"
Never: [hard constraints]
</system>
```

#### Code Generation
```xml
<system>
You are an expert [language] developer specializing in [domain].
</system>

Write [what] that [does what].

Requirements:
- Language: [language + version]
- Style: [coding conventions]
- Error handling: [strategy]
- Must include: [specific features]

<context>
[Existing code or API docs the solution must integrate with]
</context>

<examples>
<example>
<input>[sample request]</input>
<output>[complete code solution]</output>
</example>
</examples>
```

#### Analysis / Reasoning
```xml
<system>
You are an analytical expert in [domain].
</system>

Analyze the following [data type]:
<data>{{DATA}}</data>

Before answering:
1. Identify key patterns in <analysis> tags
2. List supporting evidence in <evidence> tags
3. Note any limitations in <caveats> tags

Then provide your conclusion in <conclusion> tags.
```

#### Multi-Document / RAG
```xml
<documents>
<document index="1">
<source>{{SOURCE_1_NAME}}</source>
<document_content>{{DOCUMENT_1}}</document_content>
</document>
<document index="2">
<source>{{SOURCE_2_NAME}}</source>
<document_content>{{DOCUMENT_2}}</document_content>
</document>
</documents>

Based ONLY on the documents above, answer the following question.
First, extract relevant quotes in <quotes> tags.
If the documents don't contain enough information, say "The provided documents don't contain sufficient information to answer this question."

Question: {{QUESTION}}
```

## Core Techniques Reference

### 1. XML Tags — Structure Everything

Claude is specifically trained to recognize XML tags. Use them for:
- **Separating data from instructions**: `<input>`, `<document>`, `<context>`
- **Structuring output**: `<answer>`, `<reasoning>`, `<result>`
- **Wrapping examples**: `<examples><example>...</example></examples>`
- **Variables**: `<text>{{variable_name}}</text>`

Tag names should be descriptive. Consistent across prompts.

### 2. Few-Shot Examples — Most Effective Technique

| Shots | When |
|-------|------|
| 0 (zero-shot) | Simple, well-defined tasks |
| 1 (one-shot) | When format demonstration suffices |
| 3-5 (few-shot) | Complex format, specific tone, edge cases |

Rules for examples:
- **Relevant**: mirror the real use case
- **Diverse**: cover typical + edge + refusal cases
- **Structured**: always in `<example>` tags
- Ask Claude to generate additional examples if needed

### 3. Role Prompting — More Detail = Better

| Bad | Good |
|-----|------|
| "You are a doctor." | "You are a board-certified cardiologist with 20 years of experience, known for explaining complex conditions in plain language." |
| "You are a writer." | "You are a tech journalist who writes for Wired, known for sharp wit and deep technical accuracy." |

### 4. Chain of Thought — Verbalize Reasoning

- **MUST be verbalized** — silent thinking doesn't help (except with extended thinking API)
- Use `<thinking>` tags to separate reasoning from output
- Place "think step by step" BEFORE the final task instruction
- Corrects ordering bias in comparisons

For Claude 4.x API:
```python
thinking={"type": "adaptive"}  # Claude decides when to think
output_config={"effort": "high"}  # Controls depth
```

### 5. Anti-Hallucination Techniques

1. **Give an out**: "If you're not sure, say 'I don't know'"
2. **Evidence-first**: "First extract relevant quotes, then answer based ONLY on those quotes"
3. **Investigate before answering**: "Never speculate about code/data you haven't read"
4. **Lower temperature**: 0 = deterministic, conservative; higher = creative, risky

### 6. Prompt Chaining — Multi-Step Pipelines

Pattern: Output of prompt N → Input of prompt N+1

Common chains:
- **Generate → Review → Refine** (self-correction)
- **Extract → Classify → Summarize** (pipeline)
- **Draft → Critique → Improve** (quality loop)

### 7. Variable Templates

Use `{{DOUBLE_BRACKETS}}` for dynamic content:
```
Translate the following text to {{TARGET_LANGUAGE}}:
<text>{{SOURCE_TEXT}}</text>
```

Separates fixed instructions from variable data. Enables reuse.

## Model-Specific Adjustments

### Claude 4.x (Opus 4.6, Sonnet 4.6, Haiku 4.5)

- **No prefill on last assistant turn** — use structured outputs or explicit instructions
- **No aggressive tool language** — "Use this tool when..." NOT "CRITICAL: You MUST use..."
- **Adaptive thinking** — replaces budget_tokens
- **More proactive** — dials back "above and beyond" prompting; be specific instead
- **Tell what TO DO** not what NOT to do: "Write in flowing prose" NOT "Don't use markdown"
- **Provide context for rules** — explain WHY, not just WHAT

### GPT-4 / GPT-4o

- Use `response_format: { type: "json_object" }` for JSON
- System prompt is strongly respected
- Few-shot examples work similarly
- No XML tag training — but XML still works as structure; JSON schema preferred

### Gemini

- System instructions are separate from conversation
- Supports structured output via response schema
- Few-shot examples effective
- Grounding with Google Search available

### Local / Open Source (Llama, Mistral, etc.)

- System prompt support varies by model
- Keep prompts simpler — less instruction following capacity
- More examples needed (5-10 vs 3-5)
- JSON mode may require explicit schema in prompt
- Temperature control more important

## Quality Checklist

Before delivering any prompt, verify:

- [ ] **Clear role** defined (with enough detail)
- [ ] **Task is unambiguous** — a colleague could follow it
- [ ] **XML tags** separate data from instructions
- [ ] **Examples provided** (3-5 for complex tasks)
- [ ] **Output format specified** explicitly
- [ ] **Edge cases covered** — what to do when uncertain
- [ ] **Fallback behavior defined** — "If X, then Y"
- [ ] **No conflicting instructions** — review for contradictions
- [ ] **Tested mentally** — walk through with sample input
- [ ] **Variables marked** with `{{BRACKETS}}` for dynamic content
- [ ] **Model-appropriate** — techniques match target model

## Anti-Patterns (NEVER Do)

| Anti-Pattern | Why It Fails | Fix |
|---|---|---|
| Vague instructions ("make it good") | No success criteria | Specify exact requirements |
| Negative-only rules ("don't use X") | Model doesn't know what TO do | Say what to do instead + explain why |
| No examples for complex format | Model guesses structure | Add 3-5 diverse examples |
| Data mixed with instructions | Model confuses input with command | Wrap data in XML tags |
| "CRITICAL: YOU MUST..." | Overtriggers Claude 4.x | Normal language: "Use X when..." |
| Asking to think silently | Removes reasoning benefit | Let reasoning be visible in tags |
| No fallback for uncertainty | Model hallucinates | Add "If unsure, say..." |
| Long doc at the bottom | Significant quality loss ([Anthropic docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)) | Documents at TOP, query at BOTTOM |
| One giant paragraph of instructions | Hard to parse | Use numbered lists, XML sections |
| Assuming model knows your context | Missing context = bad output | Provide background, explain why |

## Output Format for Created Prompts

When delivering a prompt to the user, always provide:

1. **The prompt itself** — ready to copy-paste, properly formatted
2. **Variables list** — all `{{VARIABLES}}` that need to be filled
3. **Where to place it** — system prompt vs user message vs both
4. **Model recommendation** — which model works best for this task
5. **Usage example** — one concrete input/output demonstration
6. **Tuning tips** — what to adjust if results aren't perfect

## Iterative Improvement Protocol

If the user says the prompt isn't working well:

1. Ask for a **concrete example** of bad output
2. Identify which technique is missing or misconfigured
3. Apply the fix (usually: add examples, add constraints, or restructure)
4. Test mentally with the failing input
5. Deliver the improved version with explanation of what changed
