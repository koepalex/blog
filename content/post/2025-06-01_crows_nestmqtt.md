---
title:       "Crow's NestMQTT"
date:        2025-05-31
tags:        ["MQTT", "IoT", "Vibe Engineering"]
categories:  ["AI/ML", "miscellaneous", "dotnet"]
---

# Crow's NestMQTT and the Vibe Engineering Adventure

In the world of (Industrial) Internet of Things (IIoT), MQTT is a widely adopted messaging protocol. Whether you’re debugging a flaky device or trying to understand system-wide message flow, a good MQTT client is a lifesaver.

There are many excellent MQTT clients available—like MqttExplorer, MQTTX, mqttui, and the classic `mosquitto_sub`. However, none quite met **my specific needs**. That kicked off a wild journey: building my own client, **Crow's NestMQTT**, while exploring how **Generative AI can accelerate software development**.

This post shares how I turned a gap in tooling into a productive side quest with the help of LLMs.

---

## 🕵️ First Clue: Designing with AI

Before writing any code, I started with a **product definition prompt** from [Harper Reed’s LLM Codegen Workflow](https://harper.blog/2025/02/16/my-llm-codegen-workflow-atm/), using **OpenAPI o3-mini** in chat mode:

> Ask me one question at a time so we can develop a thorough, step-by-step spec for this idea. Each question should build on  my previous answers, 
> and our end goal is to have a detailed specification I can hand off to a developer. Let’s do this iteratively and dig into every relevant detail. Remember, > only one question at a time.
> Here's the idea:  
> I need an MQTT client that runs on Windows, Linux, macOS, displays all message metadata (including correlation-data, response-topic, user-properties), handles thousands of messages per second, supports filtering/search/export, and can be controlled via keyboard shortcuts.

That 15-minute chat felt like brainstorming with a colleague. Surprisingly, the conversation started with **UI framework selection**, not protocol handling. We chose:

- **AvaloniaUI** for cross-platform desktop UI  
- **.NET (C#)** for the frontend and backend  
- **MQTTnet** as the MQTT library  
- A **fixed UI refresh interval** (1 second)  
- A **ring buffer** to handle high-volume messaging with predictable memory usage

We decided to **defer** support for TLS, websockets, advanced authentication, and publishing until later iterations—starting with read-only and anonymous connections.

To wrap the planning phase, I used Harper’s second prompt:

> Now that we’ve wrapped up the brainstorming process, can you compile our findings into a comprehensive, developer-ready specification? 
> Include all relevant requirements, architecture choices, data handling details, error handling strategies, and a testing plan so a developer can
> immediately begin implementation.

In hindsight, I’d now lean on an AI coder's architecture tools like:

- [aider.chat architect mode](https://aider.chat/2024/09/26/architect.html)  
- [Cline Plan mode](https://docs.cline.bot/features/plan-and-act)  
- [RooCode’s Boomerang tasks](https://docs.roocode.com/features/boomerang-tasks/)  

paired with [Sequential Thinking's MCP server](https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking) to maintain session context and step-by-step progress.

---

## 🧭 Follow the Lead: Blueprint to Prompts

Following Harper’s next guidance, I asked o3-mini-high to generate a development plan:

> Draft a detailed, step-by-step blueprint for building this project. Then, once you have a solid plan, break it down into small, iterative chunks that build
> on each other. Look at these chunks and then go another round to break it into small steps. Review the results and make sure that the steps are small
> enough to be implemented safely with strong testing, but big enough to move the project forward. Iterate until you feel that the steps are right sized for
> this project.
> 
> From here you should have the foundation to provide a series of prompts for a code-generation LLM that will implement each step in a test-driven manner.
> Prioritize best practices, incremental progress, and early testing, ensuring no big jumps in complexity at any stage. Make sure that each prompt builds on
> the previous prompts and ends with wiring things together. There should be no hanging or orphaned code that isn't integrated into a previous step.
> Make sure and separate each prompt section. Use markdown. Each prompt should be tagged as text using code tags. The goal is to output prompts, but context,
> etc is important as well.
>
> <SPEC>


The result was fantastic—each chunk clear enough to serve as a standalone **codegen prompt**.

However, I quickly hit friction in the next step: **actually generating code**. What was missing?

- A defined **project structure**  
- Explicit **library versions**  
- Style rules (.editorconfig, naming conventions)  
- Coding guidelines (e.g., DI pattern, method length, comment style)  
- Guardrails for AI coders 
  - [aider.chat conventions](https://aider.chat/docs/usage/conventions.html)
  - [Cline rules](https://docs.cline.bot/features/cline-rules)
  - [RooCode custom instructions](https://docs.roocode.com/features/custom-instructions/)

Without that, AI-generated code was... *inconsistent at best*.

To fix this, I started using **memory banks** to persist critical project metadata:

- [Cline memory bank](https://docs.cline.bot/prompting/cline-memory-bank)  
- [RooCode memory bank](https://github.com/GreatScottyMac/roo-code-memory-bank)  
- [Graphiti MCP server](https://github.com/getzep/graphiti/blob/main/mcp_server/README.md) for structured, queryable memory graphs

These helped AI tools stop asking the same questions repeatedly.

---

## 🔍 Digging for Treasure: Coding with Context

Even with a solid spec and step plan, things didn’t *just work*. AI coders kept:

- Repeating the same errors  
- Breaking existing features  
- Downgrading library versions (!)

After testing many LLM + tool combinations, I had the best experience with:

- **RooCode + Gemini 2.5 Pro**

But results were only acceptable after **prompt refinement** and adding structural constraints:

- Define interfaces explicitly  
- Refactor any class >1000 LOC  
- Limit method complexity (e.g., cyclomatic complexity ≤ 20)  
- Use DI wherever possible

Things got dramatically better when I integrated [Context7 MCP server](https://github.com/upstash/context7). It pulls in:

- Latest docs
- Code snippets
- API usage examples

This was perfect for **AvaloniaUI**—not so much for **MQTTnet**. The library wasn’t indexed on [context7.com](https://context7.com/?q=MQTTnet), and its GitHub repo mostly contained commented C# samples instead of `.md` docs.

To work around that, I:

1. Used `uithub.com/dotnet/MQTTnet` (not a typo—an alt frontend for GitHub content)
2. Selected `.cs` files only from the `/Samples` folder
3. Stored those locally for reference during prompt crafting

---

## 💰 Opening the Chest: Results

Was it perfect? No. But here’s what **really matters**:

- The client works ✅  
- I use it every day ✅  
- Time spent from idea to usable tool: **~10 hours** ✅  
- Cost if built manually: likely much higher ❌

Sure, I could build a better client alone—but this one gets the job done, **faster**, and lets me focus on things that pay the bills.

If you're facing similar MQTT challenges, you can check out my work:

👉 **Crow's NestMQTT**: https://github.com/koepalex/Crow-s-Nest-MQTT

---

## ⚓ Final Thoughts

Generative AI won't magically do all the work. But with the right architecture planning, persistent context, and clear prompts, it can become an effective **pair programmer**—one that doesn't get tired or annoyed by vague requirements.

If you're engineering tools for your own workflow, especially in IoT or high-throughput systems, give LLM-aided development a try. But remember: **vibe engineering needs structure too** and it will lead you to have developer friendly documentation in place.

