# AI Projects - Personal Notes

Quick notes on this repo and the LangChain flow.

## What this repo is

- A notebook-based learning project for LangChain and generative AI.
- Shows how to use LLMs, tools, agents, structured output, memory, and middleware.
- Not production code, mostly experiments and examples.

## Notebook map

- `generativeai/1-langchainintro.ipynb`: basic LangChain usage and model calls
- `generativeai/2-tools.ipynb`: define tools and expose them to the model
- `generativeai/3-agentintro.ipynb`: build an agent and run tool loops automatically
- `generativeai/4-agentwithmultipletools.ipynb`: multiple tools, tool selection, routing
- `generativeai/5-messages.ipynb`: message roles, chat history, conversation state
- `generativeai/6-structuredoutput.ipynb`: enforce JSON/schema outputs from the model
- `generativeai/7-memory.ipynb`: memory storage across turns
- `generativeai/8-middleware.ipynb`: intercepting requests/responses, custom hooks

## Core concepts

### LLM basics
- LLMs predict text based on input tokens.
- They do not remember past calls unless you pass history.
- Prompt engineering + good context matters.
- `.invoke()` = normal call, `.stream()` = streaming tokens/chunks, `.batch()` = parallel prompts.

### LangChain
- Wrapper around model calls, prompts, tools, and agents.
- Makes flows explicit and reusable.
- Useful for orchestration when model needs to interact with external data.

### Tools
- `@tool` makes a Python function callable by the model.
- Each tool has a schema: name, args, description.
- The model uses tool calls when it needs external data or deterministic operations.
- Tool execution cycle:
  1. model decides to call a tool
  2. program runs the Python function
  3. result is added back to chat history
  4. model continues reasoning with tool output

### Agents
- Agents automate the tool execution loop.
- They plan, choose tools, execute them, observe results, and continue.
- Good for multi-step problems and decision-making.
- `create_agent()` is the usual entrypoint here.

### Messages
- Roles matter: `SystemMessage`, `HumanMessage`, `AIMessage`, `ToolMessage`.
- Use messages to build the chat state.
- System prompts define behavior; user prompts are actual queries.
- AI and tool messages carry the model output and tool results.

### Structured output
- Use schemas to get predictable outputs.
- Common pattern: Pydantic model or typed schema.
- Structured output is more reliable than free-form text for extraction tasks.

### Memory
- Memory persists details across turns.
- Useful for preferences, context, and longer conversations.
- Not automatic: you choose what to store and recall.

### Middleware
- Middleware intercepts requests/responses.
- Can modify prompts, log events, validate outputs.
- Useful for custom behavior without changing core logic.

## Setup notes

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

If using `uv`:

```bash
uv sync
```

.env keys used:

- `GROQ_API_KEY`
- `GEMINI_API_KEY`
- `TAVILY_API_KEY`

## Quick snippets

### Basic model call

```python
from langchain.chat_models import ChatOpenAI
from langchain.schema import HumanMessage

model = ChatOpenAI(model_name="gpt-4o-mini")
response = model.invoke([HumanMessage(content="Write a short Python function to reverse a string.")])
print(response.content)
```

### Define a tool

```python
from langchain.tools import tool

@tool
def get_weather(location: str) -> str:
    return f"The weather in {location} is sunny and 24°C."
```

### Bind tool to model

```python
model = ChatOpenAI(model_name="gpt-4o-mini")
model = model.bind_tools([get_weather])
response = model.invoke([HumanMessage(content="What is the weather in Paris today?")])
print(response.content)
```

### Simple agent

```python
from langchain.agents import create_agent
from langchain.schema import SystemMessage, HumanMessage

agent = create_agent(
    model=model,
    tools=[get_weather],
    system_prompt="You are a helpful assistant that can use tools when necessary.",
)
result = agent.invoke([HumanMessage(content="Please tell me the weather in London.")])
print(result.content)
```

### Structured output example

```python
from pydantic import BaseModel, Field
from langchain.output_parsers import StructuredOutputParser
from langchain.chat_models import ChatOpenAI
from langchain.schema import HumanMessage

class EventSchema(BaseModel):
    name: str = Field(..., description="Event title")
    date: str = Field(..., description="Event date in YYYY-MM-DD format")

parser = StructuredOutputParser(pydantic_object=EventSchema)
response = model.invoke([
    HumanMessage(content="Extract event name and date from: 'Team meeting on 2026-06-01'."),
])
print(parser.parse(response.content))
```

## Notes

- `main.py` is just a stub: prints a greeting.
- `pyproject.toml` contains dependencies for LangChain, providers, `python-dotenv`, and `numexpr`.
- This repo is a place to experiment with ideas, not a production app.
