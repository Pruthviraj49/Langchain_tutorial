# AI Projects - Generative AI Learning Path

Welcome to the AI Projects repository! This project serves as a comprehensive collection of Jupyter Notebooks mapping out core concepts in building applications with Large Language Models (LLMs) using the LangChain framework. 

## 🛠️ Setup Instructions

This project utilizes `uv` and `pyproject.toml` for Python dependency management.

**Prerequisites:**
- Python `>=3.12`
- API Keys for the respective AI Models (e.g., Groq, Google GenAI, Tavily Search, etc.) populated in a `.env` file. (Refer to the `.env.example` if applicable, or create an `.env` with `GROQ_API_KEY`, `GEMINI_API_KEY`, `TAVILY_API_KEY`).

**Installation:**
```bash
# If using uv for fast package management
uv sync

# Alternatively, using standard pip with requirements.txt or pyproject
pip install -r requirements.txt
# or
pip install -e .
```

---

## 📝 Detailed Short Notes (Generative AI section)

The `/generativeai` directory contains a step-by-step introduction to working with LangChain. Below are the key takeaways and notes from each phase of the journey.

### 1. LangChain Intro (`1-langchainintro.ipynb`)
- **Initialization:** Learn to initialize models gracefully using `init_chat_model` (e.g., `google_genai:gemini-2.5-flash-lite`, `groq:llama-3.1-8b-instant`).
- **Modes of Interaction:**
  - **Standard Invoke:** Send a single text prompt and receive a full unified output.
  - **Streaming:** Process outputs dynamically using `.stream()`, presenting chunks of text as they are produced—crucial for real-time perception in chatbots.
  - **Batching:** Execute multiple prompts concurrently utilizing `.batch()` and setting `max_concurrency` config.

### 2. Tools (`2-tools.ipynb`)
- **Connecting LLMs to the World:** Models can fetch real-time data or run code by executing "Tools".
- **Tool Creation:** Custom tools are crafted using the `@tool` decorator over Python functions equipped with type hints and docstrings (which become the schema sent to the model).
- **Binding & Execution Loop:** Tools are attached to a model using `.bind_tools([my_tool])`.
- **Manual Tool Loop:** Demonstrates the manual loop of observing an `aimessage.tool_calls`, executing the respective Python function, and returning a `ToolMessage` to the LLM to get a final natural language answer.

### 3. Agent Intro (`3-agentintro.ipynb`)
- **Abstraction with Agents:** Instead of manually handling the tool execution loop, `create_agent` from `langchain.agents` builds a CompiledStateGraph (from LangGraph) to automate this.
- **Automation:** You pass the LLM, the tools, and a system prompt into the agent. Invoking the agent handles the LLM logic, tool calling, and looping behind the scenes until the final response is prepared.

### 4. Agent with Multiple Tools (`4-agentwithmultipletools.ipynb`)
- **Robust Agents:** An advanced demonstration of combining completely distinct capabilities.
- **Tavily Search:** Leveraging the `TavilySearch` tool for real-time web awareness.
- **Calculations via NumExpr:** Building a custom `@tool` calculator that securely evaluates math strings, preventing dangerous `exec()` paradigms.
- **Streaming values:** Utilizing `.stream(..., stream_mode="values")` to observe the step-by-step cognitive steps and tool selections of the agent.

### 5. Messages (`5-messages.ipynb`)
- **Core Abstractions:** Text prompts are fine for single interactions, but *Messages* are the foundation of interactive conversations requiring history.
- **Message Types:**
  - `SystemMessage`: Instruction rules for the model on how to behave.
  - `HumanMessage`: Queries representing the user.
  - `AIMessage`: Output from the model, including thinking content and potential `tool_calls`.
  - `ToolMessage`: Raw results appended after local tool execution.
- **Metadata:** Showcases how to extract detailed token counts and computation metrics via `.usage_metadata`.

### 6. Structured Output (`6-structuredoutput.ipynb`)
- **Strict Parsing:** Instead of crossing fingers for valid JSON, you can force the LLMs to abide by specific object schemas.
- **Pydantic, TypedDict, Dataclass:** Use robust Pydantic `BaseModel` classes, built-in `TypedDict`, or standard `dataclass` to define the exact fields (e.g., Movie schemas with strings, ints, nested lists).
- **`.with_structured_output(Schema)`:** Wraps the model to automatically formulate the API request explicitly for data parsing.
- **ProviderStrategy (Autoselect):** LangChain typically attempts to rely on a model provider's native schema enforcement capability (e.g., OpenAI JSON mode) since it's the safest strategy, falling back to a dummy tool call approach if needed.

---
_Generated for reference in your ongoing Generative AI learning journey._
