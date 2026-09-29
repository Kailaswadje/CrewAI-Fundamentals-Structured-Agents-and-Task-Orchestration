# 👨‍🍳 CrewAI Fundamentals — Agent, Task, and Crew as First-Class Objects (Agentic AI #4)

A first introduction to **CrewAI**, a multi-agent framework built around a deliberately different abstraction than AutoGen (used throughout the earlier notebooks in this series): instead of one agent defined by a single `system_message` string, CrewAI decomposes an agent's identity into **role, goal, and backstory**, and separates **what needs doing** (a `Task`) from **who does it** (an `Agent`) as genuinely distinct objects, bundled together and run by a `Crew`.

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python&logoColor=white)
![CrewAI](https://img.shields.io/badge/CrewAI-Multi--Agent%20Framework-FF6F00)
![OpenAI](https://img.shields.io/badge/OpenAI-gpt--4o--mini-412991?logo=openai&logoColor=white)
![Agentic AI](https://img.shields.io/badge/Series-Agentic%20AI%20%2304-8A2BE2)

---

## 📌 Overview

Parts 1–3 of this series built increasingly sophisticated multi-agent systems entirely on **AutoGen**: `ConversableAgent`, `GroupChat`, state machines, nested delegation. This notebook deliberately switches frameworks to **CrewAI**, whose four core building blocks are spelled out directly in the notebook itself:

> **Agent** — who performs the task. **Task** — the work to be done. **Crew** — which agents need to work together. **LLM** — to get a response.

Building the same *kind* of system on a second framework is what actually reveals which ideas are fundamental to agentic AI, and which are just a particular library's API shape.

---

## 🏗️ The Complete Example — A Recipe Generator Crew

```
main_ingredient = "tomato"
dietary_restrictions = "sugar"
        │
        ▼
recipe_assistant (Agent: role + goal + backstory)
        │
        ▼
Task 1: find_and_filter_recipes ──► Task 2: guide_recipe_steps
        │                                    │
        └──────────── both assigned to recipe_assistant
        │
        ▼
Crew(agents=[recipe_assistant], tasks=[...], planning=True)
        │
        ▼
crew.kickoff()  /  await crew.kickoff_async()
        │
        ▼
Final recipe + step-by-step instructions
```

---

## 🔬 Part 1 — Defining an Agent: Role, Goal, and Backstory

```python
recipe_assistant = Agent(
    role="recipe assistant",
    goal="Find recipes, filter them to meet dietary preferences, and guide user through recipe steps.",
    backstory="An experienced recipe_assistant skilled in finding and tailoring recipes based on "
               "ingredients and dietary needs, and providing clear, step-by-step cooking instructions",
    llm=llm,
    verbose=True,
)
```

This is the clearest structural contrast with AutoGen's `ConversableAgent`, which packs persona and instructions into a single free-text `system_message`. CrewAI splits the same information into three named fields:

| Field | Purpose |
|---|---|
| `role` | A short label for what the agent *is* |
| `goal` | What the agent is explicitly trying to achieve |
| `backstory` | Context that shapes *how* the agent behaves — tone, experience, approach |

Structuring persona this way makes an agent's definition more consistent and easier to audit across a larger crew than one unstructured prompt string per agent.

---

## 🔬 Part 2 — Tasks Are Independent of the Agent That Runs Them

```python
find_and_filter_recipes = Task(
    description=f"Find recipes that use the ingredient: {main_ingredient} and filter them "
                 f"to meet dietary restrictions: {dietary_restrictions}.",
    expected_output=f"One recipe using {main_ingredient} and matching {dietary_restrictions} restrictions.",
    agent=recipe_assistant,
)

guide_recipe_steps = Task(
    description="Provide step-by-step instructions for the selected recipe.",
    expected_output="Step-by-step cooking instructions for the chosen recipe.",
    agent=recipe_assistant,
)
```

A `Task` is a genuinely separate object from the `Agent`, with its own `description` and — notably — an explicit `expected_output`. Declaring what a *successful* result looks like, as a first-class field rather than an implicit assumption, is a structural nudge toward more predictable, checkable agent output. The same agent is assigned two sequential tasks here, but the object model doesn't require that — a different task could just as easily be handed to a different agent.

---

## 🔬 Part 3 — The `Crew`: Bundling Agents and Tasks, With Planning

```python
crew = Crew(
    agents=[recipe_assistant],
    tasks=[find_and_filter_recipes, guide_recipe_steps],
    planning=True,
)

result = crew.kickoff()
```

`Crew` is the orchestrator — it owns the full list of agents and the full list of tasks to execute. `planning=True` is a CrewAI-specific feature: before actually executing the tasks, the crew produces a plan for *how* to accomplish them, an extra reasoning layer AutoGen's more conversational model doesn't have as a built-in toggle.

### Synchronous and Asynchronous Execution
```python
crew.kickoff()                    # blocking call
result = await crew.kickoff_async()  # non-blocking, awaitable
```
Both execution modes are demonstrated — `kickoff_async()` is the version that fits naturally into a larger async application (e.g. a web backend handling multiple crew runs concurrently) rather than a single blocking script.

---

## 💡 A Detail Worth Noting: `SERPER_API_KEY`

```python
SERPER_API_KEY = getpass("Enter your serper api key here :")
os.environ['SERPER_API_KEY'] = SERPER_API_KEY
```
A [Serper](https://serper.dev) API key (used for web search tools in CrewAI) is collected and set as an environment variable, but **no tool actually uses it in this particular notebook** — `recipe_assistant` has no attached search tool. This is set up in preparation for giving agents real web-search capability, the natural next step once static, hardcoded ingredients (`"tomato"`, `"sugar"`) aren't enough for a genuinely dynamic recipe search.

---

## 🗂️ Repository Structure

```
crewai-fundamentals-recipe-crew/
├── AgenticAI_04_CrewAI_01.ipynb   # Main notebook
├── requirements.txt                 # Dependencies
├── .gitignore                       # Keeps secrets out of git
├── .env.example                     # Template for required environment variables
└── README.md                        # This documentation
```

---

## 🚀 Getting Started

### Prerequisites
- Python 3.9+
- An [OpenAI API key](https://platform.openai.com/api-keys)
- A [Serper API key](https://serper.dev) (free tier available) — collected by this notebook but not yet used by any tool

### Installation

```bash
git clone https://github.com/Kailaswadje/crewai-fundamentals-recipe-crew.git
cd crewai-fundamentals-recipe-crew

pip install -r requirements.txt

jupyter notebook AgenticAI_04_CrewAI_01.ipynb
```

> ⚠️ The notebook already uses `getpass()` for both API keys — good practice, keep it. Clear notebook outputs before pushing.

---

## 🧠 Key Takeaways

- **CrewAI structures persona; AutoGen leaves it free-text** — `role`/`goal`/`backstory` vs. a single `system_message` string is the clearest framework-level contrast in this series so far
- **A `Task` with an explicit `expected_output` is a small but real quality-control idea** — declaring success criteria as data, not just implying them in a prompt
- **`Crew` is the orchestrator that owns both agents and tasks together** — a different mental model from AutoGen's conversation-centric `initiate_chat`/`GroupChat` pattern
- **`planning=True` adds an explicit reasoning-before-execution step** — a CrewAI-native feature with no direct AutoGen equivalent covered so far in this series
- **Sync (`kickoff()`) and async (`kickoff_async()`) execution are both first-class** — relevant the moment a crew needs to run inside a larger asynchronous application
- Comparing this notebook against the AutoGen-based Parts 1–3 builds genuine framework-agnostic agentic AI literacy — directly useful when choosing the right orchestration layer for my dissertation's agentic intelligence platform

---

## 📚 Agentic AI Series Context

| Part | Project | Framework | Focus |
|---|---|---|---|
| 01 | [AutoGen Agent Fundamentals](https://github.com/Kailaswadje/agentic-ai-autogen-introduction) | AutoGen | `ConversableAgent`, peer-to-peer negotiation |
| 02 | [UserProxyAgent & Sequential Chat](https://github.com/Kailaswadje/agentic-ai-userproxyagent-sequential-chat) | AutoGen | Human-facing coordination, pipeline handoffs |
| 03 | [Group Chat, State Flow & Nested Chat](https://github.com/Kailaswadje/agentic-ai-group-chat-state-flow-nested-chat) | AutoGen | Multi-agent teams, deterministic orchestration |
| **04 (this repo)** | CrewAI Fundamentals | **CrewAI** | Structured agents, task/agent separation, planning |

---

## 🔮 Possible Extensions

- [ ] Attach a Serper-powered web search tool to `recipe_assistant`, replacing the hardcoded ingredient/restriction with a genuine search
- [ ] Add a second, specialised agent (e.g. a nutrition-checking agent) and split the two tasks across them
- [ ] Compare `planning=True` vs `planning=False` output on the same task set
- [ ] Rebuild Part 1's marketing-strategist scenario in CrewAI for a direct side-by-side framework comparison

---

## 👤 Author

**Kailas Wadje**
MSc Data Science & AI, University of Liverpool

- GitHub: [@Kailaswadje](https://github.com/Kailaswadje)
- LinkedIn: [linkedin.com/in/kwadaje](https://www.linkedin.com/in/kwadaje/)

---

⭐ If seeing agentic AI from a second framework's perspective sharpened your understanding, consider giving it a star!
