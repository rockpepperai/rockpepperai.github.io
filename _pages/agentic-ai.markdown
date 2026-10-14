---
layout: page
title: Roadmap for Agentic AI
permalink: /agentic-ai/
includelink: false
comment: false
---

### 🆕 Developing agentic software?

I am posting content here that will help you get idea on building agentic software systems.

Before you proceed, you need to remember...

**Machines don't learn or think like humans do! It's all math.**

<a href="/2025/04/24/smart-machines/">Know more...</a>

In my opinion, crafting agentic software depends mainly on five maturity areas...

1️⃣ Understanding the complexity of the problem and how agents can help (will it really benefit with such a solution?)

2️⃣ Understanding the level of accuracy and human intervention required (beware, LLMs are probabilistic software!)

3️⃣ Defining agent personas with clear objectives: <a href="/2025/03/16/llm-bayes-theorem/">Writing clever prompts</a> and using different prompt engineering techniques

4️⃣ Thinking simple workflows: Restrospecting the output at every step, understand what can improve accuracy

5️⃣ Understanding implications on security, privacy and compliance - <a href="/2026/08/19/owasp-top-10/">OWASP Top 10s</a> is a good place to start

**To build capabilities in agentic AI, you could use this three step process.**

I followed this three step process myself.

**✨ Step 1: Be an efficient software craftsman**

✅ <a href="/2025/07/22/gen-ai-benefits/">Stop thinking that AI is the answer to all your problems</a>

✅ Be part of, or develop an excellent engineering culture

✅ Collaborate, experiment and learn; understand what others are doing

✅ Learn Python, async programming, API development and a database with vector support (pgvector is an excellent choice)

**Why Python?** You can develop agentic software using Java, JS or any programming language, but Python's community support and strong-establishment in the AI/ML and now in GenAI and Agentic AI space is tremendous. Even the libraries and frameworks used in GenAI/Agentic AI usually come for Python first. 

Python perhaps is easier to learn, but you need to know how to do async programming, multiprocessing, create modular and manageable systems, clean coding, unit/integration testing, logging, monitoring, etc, (like any other programming language) in order to build usable and maintainable software systems.

✅ Read why LLMs are called probabilistic systems, use ChatGPT or Gemini and experiment with prompts

**✨ Step 2: Understand the building blocks**

✅ Start small; build a simple chatbot and experiment with different prompt techniques like, one-shot, few-shot, and advanced techniques like ReAct, CoT, etc

✅ Learn vector databases, tokens, text embedding, chunks, etc - Learn how an embedding model works, why text is converted into a vector of numbers and how it helps

✅ <a href="https://www.youtube.com/watch?v=qN_2fnOPY-M">Build a RAG workflow</a>, connect it with the chatbot, add <a href="/2026/05/30/memory-architecture/">memory</a> - Try <a href="https://www.ibm.com/think/topics/agentic-rag">Agentic RAG</a>!

✅ Learn different RAG techniques and simple tool usage with LLMs - Try advanced RAG techniqies like Corrective and Adaptive RAG, and once you outgrow toy datasets, see how to do <a href="/2026/02/13/RAG-on-millions-of-documents/">hybrid RAG at scale</a>

✅ Understand the economics of cloud computing (there is no free lunch - There are many open source tools available in this space to build things, but when you run things, it costs money, and it's very easy to overspend!)

**✨ Step 3: Develop agents**

✅ Build a system with two agents that interact with each other, define agent goals, criteria, expected results, etc - you don't have to do this fully from scratch either, frameworks like LangGraph, Microsoft Agent Framework, OpenAI's Agents SDK and Google's ADK all give you this scaffolding out of the box

✅ Optimize token usage by <a href="/2025/10/04/prompt-vs-context/">engineering agent context</a>!

✅ Learn structured inputs, structured outputs and guardrails

✅ Learn agentic workflow patterns (like supervisor-worker, chaining, parallelising, evaluator-optimizer, human-in-the-loop, etc), add multiple agents - see <a href="/2025/12/03/hitl-langgraph/">Human In the Loop using LangGraph</a> for how to pause an agent for a human decision

✅ Once a single agent isn't enough, look at the "harness" pattern - planning, sub-agents, a shared file system and context isolation, the same shape used by Claude Code, Manus and most production deep-research systems. I've written about this in detail in <a href="/2026/03/22/agent-harness/">Agent Harness</a> and <a href="/deep-research/">Deep Research 101</a>

✅ Provide more tools to the agent, learn how agent picks the best tool. <a href="https://modelcontextprotocol.io/">Model Context Protocol (MCP)</a> is now the default way agents discover and call tools - <a href="/2025/12/06/fastmcp/">FastMCP</a> is a great way to build your own MCP server. Pair it with **Skills** - reusable, named behaviours the agent loads on demand instead of stuffing every capability into one system prompt

✅ Logging, monitoring, LLMOps and deployment (Check tools like LangSmith, Arize, etc)

✅ Dive deep into evals (Evaluation of how the agents are performing!) - LLM as judge is a popular technique done using the evaluator-optimizer pattern,.
