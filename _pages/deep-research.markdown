---
layout: page
title: Deep Research 101
permalink: /deep-research/
includelink: false
comment: false
---

Everything I have learnt about deep research are from the following resources:

1️⃣ [LangChain's deep research from scratch](https://github.com/langchain-ai/deep_research_from_scratch)

2️⃣ [LangChain's deep agents from scratch](https://github.com/langchain-ai/deep-agents-from-scratch)

3️⃣ [renschi's](https://github.com/renschni) [Manus agent report](https://gist.github.com/renschni/4fbc70b31bad8dd57f3370239dccd58f)

4️⃣ [Deep Agents by Harrison Chase](https://www.langchain.com/blog/deep-agents) — the pattern below, given a name and formalised into an open-source harness

5️⃣ [Deep Agents vs LangChain vs LangGraph](https://www.langchain.com/blog/deep-agents-vs-langchain-vs-langgraph) — where each of these layers actually fits

6️⃣ [Organizing Context in a Multi-Agent Harness](https://www.langchain.com/blog/organizing-context-in-a-multi-agent-harness) — current thinking on what to share between the orchestrator and its sub-agents

> 📌 This page is deliberately **conceptual, not a tutorial**. Everything below is pseudocode — the goal is to understand the shape of the pattern, not to copy-paste a working agent. Translate it to whatever framework or language you use.

**The core idea is to:**

✅ Have a "main" agent, that is provided with a bigger goal which it needs to achieve using specialised sub-agents

```
define deep_research_agent:
  model      = llm()
  tools      = [ sub_agent_selection_tool ]
  output     = CustomStructuredOutput
  return agent(model, tools, output)
```

✅ A set of sub-agents who have smaller goals and tools they need, to help them achieve their individual goals - These sub-agents are used by the main agent as tools

```
define sub_agent(prompt, tools):
  return agent(model=llm(), tools=tools, prompt=prompt)

sub_agents = {
  "sub_agent_1": sub_agent(SUB_AGENT_1_PROMPT, [...]),  # specialist in task A
  "sub_agent_2": sub_agent(SUB_AGENT_2_PROMPT, [...]),  # specialist in task B
  "sub_agent_3": sub_agent(SUB_AGENT_3_PROMPT, [...]),  # specialist in task C
  "sub_agent_4": sub_agent(SUB_AGENT_4_PROMPT, [...]),  # specialist in task D
  "sub_agent_5": sub_agent(SUB_AGENT_5_PROMPT, [...]),  # specialist in task E
}
```

✅ Breakdown the bigger goal into multiple smaller tasks - A list of ToDos or plans or task items. The main agent is tasked to think and breakdown the bigger goals into specific (nearly) atomic tasks that can be executed independently. Give the agent a dedicated **planning tool** rather than expecting it to hold the plan purely in its head - each ToDo gets a status (`pending` / `in_progress` / `completed`), and the agent is nudged to re-read (**recite**) its own plan every few steps so it doesn't drift off track on a long-running task

```
tool write_todos(todos):
  # todos: list of { task, status }
  state.todos = todos
  return "todos updated"

tool read_todos():
  return state.todos

# the agent re-reads its own plan periodically -
# this "recitation" is what keeps a long task on track
loop:
  plan = read_todos()
  next_task = pick(plan, status="pending")
  act_on(next_task)
```

✅ Having access to a file system tool that can read-write the ToDos on demand rather than putting everything in prompt - As the complexity grows, the information you want to put in the prompt also grows - This is exactly we need to keep check of

✅ Provide the main agent with a sub-agent selection tool to help determine which sub-agent is good at which task - The trick is to treat the response from the sub-agents as a **tool result** rather than an **assistant message**. So when the work done by the sub-agents are sent back to the main agent, to the main agent they are simply output from tool calls, not something someone "said"

```
tool sub_agent_selection_tool(content, which_sub_agent):
  # content         : atleast 5 sentences, extracted from the input, that this
  #                   sub-agent needs to do its job (more the better)
  # which_sub_agent : exactly one registered sub-agent name to delegate to

  sub_agent = sub_agents[which_sub_agent]
  result    = sub_agent.invoke(content)

  # return the result as a TOOL message, not an AI message -
  # this is what keeps the sub-agent's context isolated from the main agent's
  return tool_result(result)
```

✅ Cleverly written [**ReAct** prompts](https://smith.langchain.com/hub/hwchase17/react) and efficient context engineering [**read this**](https://simonwillison.net/2025/jun/27/context-engineering/), [**this**](https://www.philschmid.de/context-engineering), and [**this**](https://www.langchain.com/blog/organizing-context-in-a-multi-agent-harness) for the multi-agent specific version of it - Create as much context for individual agents, remove anything not required

✅ ToDos can be read from file system and put into context as required, RAG/knowledge graph based data retrieval also helps to identify key information required for the particular agent in action and providing it the particular information it needs (nearly zero-noise contenxt - as LLMs hallucinate when they have to juggle with a lot of different information). [More on ToDos.. the LangChain way](https://www.youtube.com/watch?v=dwvhZ1z_Pas).

✅ Let the agent keep an **internal monologue** - a scratchpad of "thinking out loud" that is never shown to the user, kept separate from the final answer. This is where the agent reasons about tradeoffs, notices contradictions in what sub-agents returned, and decides what to do next - the report is only written at the very end, from this scratchpad, not turn by turn

```
tool think(scratch_note):
  # a no-op tool - writing here changes nothing in the world,
  # it just gives the model a place to reason before acting
  state.scratchpad.append(scratch_note)
  return "noted"
```

✅ Build in **self-correction** - before the main agent accepts a sub-agent's result (or its own draft), have it check the result against the original goal and the ToDo it was meant to satisfy. If it doesn't hold up, don't just retry blindly - fix the specific gap and try again

```
result = sub_agent.invoke(content)

if not satisfies(result, goal=task):
  gap    = diagnose(result, task)
  result = sub_agent.invoke(content, hint=gap)   # retry, but smarter this time

mark_todo_complete(task)
```

✅ **Prune bad paths early.** Not every sub-agent call or research direction pays off. Have the main agent track a lightweight confidence/success signal per branch, and abandon a branch the moment it stops being productive instead of pouring more context and tool calls into a dead end - the failed attempt is worth a one-line note in the scratchpad ("tried X, didn't work, moving on"), not paragraphs of unusable output carried forward into the final report

```
for task in todos:
  attempt = 0
  while attempt < max_attempts:
    result = sub_agent.invoke(task)
    if is_promising(result):
      accept(result)
      break
    attempt += 1
  else:
    prune(task)   # give up on this path, note it, move to the next task
```

✅ The "main" agent finally aggregates all the results from all the sub-agents and creates the final research report in one shot

```
DEEP_RESEARCH_PROMPT:
  """
  You are an expert in this and that. You have access to specialist sub-agents
  to delegate your work. Each sub-agent will need specific sections from the
  input provided to you - extract those sections and hand them off during
  delegation.

  Available agents for delegation:
  - sub_agent_1: expert in task A
  - sub_agent_2: expert in task B
  - sub_agent_3: expert in task C
  - sub_agent_4: expert in task D
  - sub_agent_5: expert in task E

  Input:
  <input>{input}</input>

  Your goal:
  ...

  Your tasks:
  1. Delegate work to each sub-agent based on its specialisation - think and
     extract (atleast 5 sentences, more the better) the relevant section for it
  2. Do task A / task B / task C / task D / task E
  3. Combine output exactly as returned by all the sub-agents into one report
  4. Add a summary, your recommendation and commentary at the end
  5. Add a note that this report is AI-generated and must be verified by a human

  Output only the final report - no follow-up questions, no extra commentary.
  """

response = deep_research_agent.invoke(DEEP_RESEARCH_PROMPT(input))
```

**A few mechanisms mature systems tend to layer on top of the above:**

✅ **Skills** - reusable, named behaviours the agent can load on demand, instead of stuffing every capability it might ever need into one giant system prompt

✅ **Human-in-the-loop** - certain steps (an expensive tool call, a delegation with real-world side effects) are worth pausing on and letting a human approve, edit or reject before the agent proceeds - see my [HITL with LangGraph](/2025/12/03/hitl-langgraph/) post for one way to implement this

✅ **MCP as the default way to plug in tools** - instead of writing every tool by hand, agents increasingly reach for tools already exposed by [MCP](https://www.anthropic.com/news/model-context-protocol) servers

✅ **Persistent, cross-session memory** - the ToDo-and-file-system trick above is scoped to a single run; production systems persist a slice of that memory across sessions too, so the agent doesn't start from zero every time

None of this needs to be hand-rolled from scratch - [`deepagents`](https://github.com/langchain-ai/deepagents) is an open-source harness that packages the entire pattern above (planning tool, sub-agent delegation, virtual file system, context management) out of the box, and is a good reference implementation to study alongside this page.



I have used this pattern for a quite complicated research-like use case (with GPT 4.1 mini, and later on Claude Sonnet 4.6)... 

The results were amazing! 

🔥 On top of this core idea of deep research, I had used periodic summarization of messages exchanged, context tailoring and evaluation of report using the LLM-as-judge technique.

🔥 LLM-as-judge could simply be done by having a specialist agent with "evaluator" role that knows your evaluation criteria (may be with examples), and it evaluates the content provided with discrete labels like POOR, AVERAGE, GOOD or VERY GOOD. Then your conditional acceptance may go ahead with GOOD and VERY GOOD reports, and send POOR and GOOD reports back to the main agent to re-do the research. The trick is to configure your evaluator agent to write a feedback for substandard reports, which you can put into the context of the main or sub-agents in the next pass.

Give it a try.
