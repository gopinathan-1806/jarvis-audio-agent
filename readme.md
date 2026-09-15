# 🎙️ Telegram AI Voice Assistant with n8n

An AI-powered Telegram voice assistant built with **n8n**. It accepts voice or text messages, processes the request with an AI Agent, and returns the response to Telegram as text and/or audio.

## 🎯 Objective

```text
Telegram Voice Message
        ↓
Speech-to-Text
        ↓
AI Agent
        ↓
Text-to-Speech
        ↓
Telegram Audio Response
```

The workflow also supports direct text input and connects the AI Agent with memory and external tools.

---

## 🏗️ Workflow Architecture

```text
                         Telegram
                            │
                            ▼
                   ┌─────────────────┐
                   │ Telegram Trigger│
                   │    Message      │
                   └────────┬────────┘
                            │
                            ▼
                       ┌─────────┐
                       │ Switch  │
                       └───┬─┬───┘
                         Voice Text
                           │    │
             ┌─────────────┘    └─────────────┐
             ▼                                ▼
       ┌─────────────┐                  ┌────────────┐
       │  Get a File │                  │ Input_var  │
       │   Telegram  │                  │ Text Input │
       └──────┬──────┘                  └─────┬──────┘
              ▼                               │
       ┌─────────────┐                        │
       │     STT     │                        │
       │ Speech→Text │                        │
       └──────┬──────┘                        │
              ▼                               │
       ┌─────────────┐                        │
       │  audio_var  │                        │
       └──────┬──────┘                        │
              └──────────────┬────────────────┘
                             ▼
                     ┌─────────────┐
                     │   AI Agent  │
                     └──────┬──────┘
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
    OpenAI Chat Model   Simple Memory      SerpAPI
                                             │
                            ┌────────────────┼───────┐
                            ▼                ▼       ▼
                          Gmail          Calculator
                            Tools
                            │
                            ▼
                   ┌─────────────────┐
                   │ Basic LLM Chain │
                   │ Final Response  │
                   └────────┬────────┘
                            │
                   ┌────────┴────────┐
                   ▼                 ▼
              Telegram             TTS
              Text Reply      Generate Audio
                                    │
                                    ▼
                                Telegram
                                    │
                                    ▼
                              🔊 Audio Reply
```

---

## 🔄 End-to-End Flow

### 1. Telegram Trigger

The **Telegram Trigger** listens for incoming messages from the Telegram bot.

It receives the user's message and passes it to the **Switch** node.

```text
User → Telegram Bot → Telegram Trigger
```

### 2. Switch — Voice vs Text

The **Switch** determines whether the incoming message contains:

- Voice
- Text

```text
                 Telegram Message
                       │
                     Switch
                    /      \
                 Voice      Text
                   │          │
                   ▼          ▼
              Voice Flow   Input_var
```

### 3. Get a File

For voice messages, the **Get a File** Telegram node retrieves the audio file so it can be processed.

```text
Telegram Voice
      ↓
Get a File
      ↓
Audio File
```

### 4. STT — Speech-to-Text

The **STT** node converts the audio into text.

```text
Audio
  ↓
Speech-to-Text
  ↓
Transcribed User Request
```

### 5. Input Normalization

- `audio_var` prepares the transcribed voice input.
- `Input_var` prepares direct text input.

Both paths eventually reach the same AI Agent.

```text
Voice → STT → audio_var ─┐
                         ├──→ AI Agent
Text ───────→ Input_var ─┘
```

---

## 🤖 AI Agent

The **AI Agent** is the central intelligence of the workflow.

It receives the user's request, reasons about the task, and can use connected tools when required.

### Connected components

| Component | Purpose |
|---|---|
| **OpenAI Chat Model** | LLM used by the AI Agent |
| **Simple Memory** | Maintains conversational context |
| **SerpAPI** | Web search |
| **Gmail** | Gmail-related tool/action |
| **Calculator** | Mathematical calculations |

Conceptually:

```text
                  AI Agent
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
       LLM         Memory       Tools
                                 │
                       ┌─────────┼─────────┐
                       ▼         ▼         ▼
                    SerpAPI    Gmail   Calculator
```

---

## ✍️ Response Generation

The AI Agent response is passed to the **Basic LLM Chain**, which prepares the final response.

```text
AI Agent
   ↓
Basic LLM Chain
   ↓
Final Response Text
```

The response can then be sent as text and/or converted into audio.

---

## 🔊 Text-to-Speech

The **TTS** node converts the generated response into audio.

```text
AI Response
     ↓
    TTS
     ↓
Audio Response
```

The audio is then sent back to Telegram.

---

## 📤 Telegram Output

The workflow has two response paths:

### Text

```text
AI Response → Telegram → Text Message
```

### Audio

```text
AI Response → TTS → Telegram → Audio Message
```

This creates a conversational voice-assistant experience inside Telegram.

---

## 🧠 Core Architecture

The overall application can be viewed as:

```text
INPUT
  │
  ├── Voice ──→ STT ──┐
  │                    │
  └── Text ────────────┤
                       ▼
                   AI AGENT
                       │
             ┌─────────┼─────────┐
             │         │         │
            LLM      Memory     Tools
                       │
                       ▼
                  AI RESPONSE
                       │
                 ┌─────┴─────┐
                 ▼           ▼
              Telegram      TTS
               Text          │
                             ▼
                          Telegram
                           Audio
```

---

## 🛠️ Technologies Used

- **n8n** — Workflow orchestration
- **Telegram** — User interface and message delivery
- **OpenAI Chat Model** — LLM
- **STT** — Speech-to-text processing
- **TTS** — Text-to-speech generation
- **Simple Memory** — Conversation context
- **SerpAPI** — Web search tool
- **Gmail** — AI Agent tool
- **Calculator** — Calculation tool

---

## 🧪 Testing

### Test 1 — Text

Send a text message to the Telegram bot and verify:

```text
Telegram Trigger
 → Switch
 → Input_var
 → AI Agent
 → Response
```

### Test 2 — Voice

Send a voice message and verify:

```text
Telegram Trigger
 → Switch
 → Get a File
 → STT
 → audio_var
 → AI Agent
```

### Test 3 — Audio Response

Verify:

```text
AI Agent
 → Basic LLM Chain
 → TTS
 → Telegram
 → Audio received
```

### Test 4 — Tools

Test requests that require:

- Web search
- Calculation
- Gmail functionality
- Conversation memory

---

## 🔐 Credentials & Security

Depending on the configured nodes, the workflow requires credentials for:

- Telegram
- OpenAI
- SerpAPI
- Gmail

Store credentials inside n8n's credential system.

**Never commit API keys, bot tokens, OAuth secrets, or passwords to GitHub.**

---

## 🚀 Possible Improvements

### Input Guardrails

Validate and filter user input before the AI Agent.

```text
Telegram
   ↓
Input Guardrails
   ↓
AI Agent
```

### Output Guardrails

Validate the AI response before sending it to the user.

```text
AI Agent
   ↓
Output Guardrails
   ↓
TTS / Telegram
```

### Persistent Memory

Replace simple memory with a persistent database for long-term conversations.

### RAG

Add a vector database and private knowledge base:

```text
User Question
     ↓
AI Agent
     ↓
Retriever
     ↓
Knowledge Base
     ↓
AI Response
```

### Observability

Track:

- Workflow executions
- STT latency/failures
- LLM latency
- Tool failures
- TTS failures
- Telegram delivery failures

### Error Handling

Add dedicated error paths for failed audio downloads, STT, LLM, tools, TTS, and Telegram delivery.

---

## 📂 Suggested Repository Structure

```text
telegram-ai-voice-assistant/
│
├── README.md
├── workflow.json
└── screenshots/
    └── telegram-ai-voice-assistant.png
```

Export the n8n workflow as `workflow.json` to keep the workflow configuration under Git version control.

---

## 📸 Workflow Screenshot

Place the completed n8n screenshot in:

```text
screenshots/telegram-ai-voice-assistant.png
```

Then add:

```markdown
![Telegram AI Voice Assistant](screenshots/telegram-ai-voice-assistant.png)
```

---

## 🎓 Project Summary

This project demonstrates how **n8n can be used to build an AI-powered conversational voice assistant**, combining workflow automation with LLMs, tools, memory, speech processing, and Telegram.

The main architecture is:

**Telegram → Input Handling → STT → AI Agent → Tools/Memory → Response → TTS → Telegram**

The project demonstrates practical concepts including **event-driven automation, multimodal input handling, AI Agents, tool calling, conversational memory, speech-to-text, text-to-speech, and workflow orchestration**.

---

## 🔗 Skills Demonstrated

`n8n` `Generative AI` `LLM` `AI Agents` `Telegram Bot` `Speech-to-Text` `Text-to-Speech` `Tool Calling` `Memory` `SerpAPI` `Gmail Integration` `Workflow Automation`

### Learning Approach

**Build → Understand → Test → Learn → Improve → Integrate**

<img width="1212" height="432" alt="Screenshot 2026-09-15 at 3 13 38 PM" src="https://github.com/user-attachments/assets/4c485db9-1870-49b8-81a7-5a61e2c3e4f6" />

<img width="1481" height="807" alt="image" src="https://github.com/user-attachments/assets/55ed725d-357c-4529-b927-0559d6add0e1" />

