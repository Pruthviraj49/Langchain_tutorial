# AI Projects - Generative AI & LangChain Reference Guide

Welcome to the AI Projects repository! This repository contains a series of Jupyter Notebooks mapping out core concepts in building robust applications using Large Language Models (LLMs) and the LangChain framework. This detailed README serves as a comprehensive reference guide to the topics covered.

---

## 🛠️ Installation & Setup

This project uses modern Python packaging strategies. You can find `pyproject.toml`, `requirements.txt`, and `uv.lock` at the root.

**Prerequisites:**
- Python `>=3.12`
- API Keys for necessary AI providers (Groq, Google GenAI, Anthropic, etc.) and search providers (Tavily).

**Installation Environments:**
```bash
# Recommended: Fast installation using 'uv'
uv sync

# Standard pip installation
pip install -r requirements.txt
```

**Environment Variables (`.env`):**
Make sure to create a `.env` file in the root directory to store your API keys securely. The notebooks use `python-dotenv` to load these dynamically:
`GROQ_API_KEY=your_key_here`
`GEMINI_API_KEY=your_key_here`
`TAVILY_API_KEY=your_key_here`

---

## 📖 Detailed Learning Notes

The `/generativeai` directory contains a step-by-step introduction to working with LangChain. Below are the key takeaways and technical details from each phase of the journey.

### 1. Introduction to LangChain & Basic Interaction (`1-langchainintro.ipynb`)
- **Initialization:** Introduced `init_chat_model()` to smoothly connect to multiple LLM providers (e.g., `google_genai:gemini-2.5-flash-lite`, `groq:llama-3.1-8b-instant`). This abstraction makes it easy to swap models without changing core application logic.
- **Modes of Interaction:**
  - **`.invoke()`:** The standard synchronous call. Sends a text prompt to the model and waits for the full text response via `AIMessage`.
  - **`.stream()`:** Essential for real-time user experiences. Yields an active generator `BaseChatModel.stream` that returns text chunks iteratively, avoiding long loading times.
  - **`.batch()`:** Executes multiple distinct prompts concurrently. Utilizes `max_concurrency` via the config dictionary to throttle parallel network requests.

### 2. Giving the AI Tools (`2-tools.ipynb`)
- **Concept:** LLMs generally suffer from knowledge cut-offs and hallucinatory tendencies. *Tools* are the solution, allowing the AI to fetch external data (like calling weather APIs) or execute code strings.
- **`@tool` Decorator:** Converts standard Python functions into schemas the model can understand. Type hints (e.g., `location: str`) and docstrings are highly important, as they tell the model exactly *when* and *how* to use the function.
- **`.bind_tools()`:** Once the tools are defined, `model.bind_tools([my_tool])` binds the execution schemas to the model.
- **Manual Execution Loop:** 
  1. User asks a question (e.g., "What's the weather in Mumbai?").
  2. Model returns an `AIMessage` containing a `tool_calls` argument.
  3. The local program detects the tool call, matches the function name, and executes the python function.
  4. The result is appended to the message history and fed *back* to the model to generate the final natural language answer.

### 3. Agent Fundamentals (`3-agentintro.ipynb`)
- **What is an Agent?** Managing the manual "tool execution loop" (checking for tool calls, routing, parsing JSON, passing back) gets incredibly tedious for complex applications. An Agent is a system that automates this entire loop.
- **`create_agent()`:** Uses `langchain.agents` (and underlying `CompiledStateGraph` from LangGraph) to create an autonomous loop.
- You supply the model, the tools, and a `system_prompt`. When `.invoke()` is called, the Agent will loop recursively—using tools and feeding the answers back to itself—until it has enough context to satisfy the user's initial query.

### 4. Agents with Multiple Tools (`4-agentwithmultipletools.ipynb`)
- **Multi-Tool Routing:** Demonstrates a model intelligently deciding between totally different tools based on context.
- **Implementation Examples used:**
  - **`TavilySearch`**: A highly robust, LLM-optimized search engine tool to query live internet results.
  - **Calculator Tool (`numexpr`)**: Shows how to construct a math-parsing tool using `numexpr.evaluate()`. This is specifically highlighted as a secure alternative to the dangerous built-in `eval()` function, which can lead to code injection attacks.
- **Stream Mode Values:** Instead of just `.invoke()`, utilizing `.stream(..., stream_mode="values")` allows developers to watch the Agent's state update live as it thinks and interacts with different tools.

### 5. Managing Conversation History & Messages (`5-messages.ipynb`)
- **The Concept of Context:** Simple string inputs are referred to as *Text Prompts*. However, LLMs are stateless; to maintain chat history, we must use a queue of Message Objects.
- **Message Types:**
  - `SystemMessage`: Establishes the behavior, rules, and persona of the AI (e.g., "You are a helpful data science assistant").
  - `HumanMessage`: Represents the queries inputs from the user.
  - `AIMessage`: The generated model outputs. It includes the visible text content, reasoning content (for reasoning models), and potential `tool_calls`.
  - `ToolMessage`: Acts as the bridge that carries raw output strings from completed tool executions back into the model's chat history.
- **Metadata Management:** Accessing `response.usage_metadata` to carefully track `input_tokens`, `output_tokens`, and compute time.

### 6. Ensuring Structured Output (`6-structuredoutput.ipynb`)
- **Why Structured Output?** Often, you want an AI to do data extraction (e.g., parsing an email for names and dates) and return a strict JSON payload, rather than answering with conversational text.
- **`.with_structured_output(Schema)`:** Wraps the model to automatically enforce output formats.
- **Creating the Schemas:**
  - **Pydantic**: Use `BaseModel` and `Field` for the richest set of runtime validation, deep nested structures, and field descriptions.
  - **TypedDict / Dataclasses**: Built-in alternatives when strict runtime validation is unnecessary but you still want reliable typing.
- **ProviderStrategy vs ToolStrategy:** 
  - LangChain features an automatic mechanism. If the model theoretically supports strict, native JSON mode (like Groq's Qwen or OpenAI's newer models), LangChain automatically applies `ProviderStrategy`. 
  - If the model lacks native support, LangChain falls back to `ToolStrategy`, mimicking structured output by forcing the model to make exactly one "tool call" with arguments mapped to the requested schema.
