# LangChain Core Concepts

A hands-on project exploring the foundational capabilities of the LangChain v1 framework, implemented as a series of Jupyter notebooks. The project covers agent construction, multi-provider model integration, tool binding, message handling, structured output, and agent middleware — built on Python 3.13 with `uv` for dependency management.

---

## Project Structure

```
LangChain/
├── LangChain/
│   ├── 1-langchainintro.ipynb       # Agent creation and invocation
│   ├── 2-modelintegration.ipynb     # Multi-provider model integration
│   ├── 3-tools.ipynb                # Tool definition and execution loops
│   ├── 4-messages.ipynb             # Message types and conversation context
│   ├── 5-structuredoutput.ipynb     # Structured output with Pydantic and TypedDict
│   └── 6-middleware.ipynb           # Agent middleware (summarization & human-in-the-loop)
├── main.py
├── pyproject.toml
└── requirements.txt
```

---

## Notebooks Overview

### 1 · Agent Creation (`1-langchainintro.ipynb`)
Demonstrates how to build and invoke an agent using `langchain.agents.create_agent`. The agent is wired to a custom `get_weather` tool and backed by a GPT-5 model. The execution trace shows the full message thread — `HumanMessage → AIMessage (tool call) → ToolMessage → AIMessage (final response)`.

### 2 · Model Integration (`2-modelintegration.ipynb`)
Covers connecting to three LLM providers through a unified interface:

| Provider | Interface |
|---|---|
| OpenAI (GPT-4.1) | `init_chat_model` / `ChatOpenAI` |
| Google Gemini 2.5 Flash | `init_chat_model` / `ChatGoogleGenerativeAI` |
| Groq (Qwen 3 32B) | `init_chat_model` / `ChatGroq` |

Also explores **streaming** (`model.stream(...)`) for progressive token output and **batch** processing (`model.batch([...])`) for running multiple requests in parallel with configurable concurrency.

### 3 · Tools (`3-tools.ipynb`)
Shows how to define callable tools using the `@tool` decorator, bind them to a model with `model.bind_tools(...)`, and implement a **tool execution loop**: the model issues a tool call → the tool executes → the result is appended to the message thread → the model produces a final response.

### 4 · Messages (`4-messages.ipynb`)
Covers LangChain's standard message types, which provide a consistent interface across all providers:

- **`SystemMessage`** — sets model behavior and persona
- **`HumanMessage`** — represents user input (supports text, images, files)
- **`AIMessage`** — model output, including tool calls and metadata
- **`ToolMessage`** — carries tool execution results back to the model

Includes examples of building multi-turn conversation histories manually, attaching metadata (`name`, `id`) to messages, and inspecting `usage_metadata` for token counts.

### 5 · Structured Output (`5-structuredoutput.ipynb`)
Demonstrates how to constrain model responses to a defined schema using `model.with_structured_output(...)`. Three schema approaches are covered:

- **Pydantic `BaseModel`** — field validation, descriptions, nested structures (e.g., `Movie`, `MovieDetails` with a nested `Actor` list)
- **`TypedDict`** — simpler alternative without runtime validation
- **`@dataclass`** — Python dataclass approach

Also shows `include_raw=True` to retrieve both the raw `AIMessage` and the parsed structured object simultaneously, and integration with `create_agent` via the `response_format` parameter.

### 6 · Middleware (`6-middleware.ipynb`)
Explores the middleware layer for controlling agent behavior at runtime:

**`SummarizationMiddleware`** — automatically compresses conversation history when it exceeds a configurable threshold (by message count, token count, or context fraction), preserving recent messages while replacing older context with a summary.

**`HumanInTheLoopMiddleware`** — intercepts specific tool calls before execution and pauses the agent, yielding an `__interrupt__` signal. Supports three resolution paths:
- `approve` — execute the tool as-is
- `edit` — modify the tool arguments before execution
- `reject` — cancel the tool call with an error result

All middleware examples use `InMemorySaver` for conversation state persistence across turns.

---

## Requirements

- Python ≥ 3.13
- [`uv`](https://docs.astral.sh/uv/) (recommended) or `pip`

Key dependencies:

```
langchain>=1.1.0
langchain-openai>=1.1.0
langchain-google-genai>=3.2.0
langchain-groq>=1.1.0
langchain-community>=0.4.1
python-dotenv>=1.2.1
ipykernel>=7.1.0
```

---

## Setup

**1. Clone and enter the project directory:**

```bash
git clone <repo-url>
cd LangChain
```

**2. Install dependencies (with `uv`):**

```bash
uv sync
```

Or with pip:

```bash
pip install -r requirements.txt
```

**3. Configure API keys:**

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your_openai_key
GOOGLE_API_KEY=your_google_key
GROQ_API_KEY=your_groq_key
```

**4. Open the notebooks:**

```bash
jupyter notebook LangChain/
```

Select the `.venv` kernel when prompted.

---

## Tech Stack

- **LangChain v1** — agent framework and model abstraction
- **LangGraph** — underlying graph engine and `InMemorySaver` checkpointer
- **OpenAI / Google Gemini / Groq** — LLM providers
- **Pydantic** — schema definition for structured output
- **`uv`** — fast Python package and project manager
