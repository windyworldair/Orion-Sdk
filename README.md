<p align="center">
  <img src="image.png" alt="Orion SDK" width="400"/>
</p>

<h1 align="center">Orion SDK</h1>

<p align="center">	
  <strong>The model-agnostic AI SDK — one interface, every provider.</strong>
</p>

<p align="center">
  <a href="https://pypi.org/project/orion-sdk/"><img src="https://img.shields.io/pypi/v/orion-sdk?style=flat-square&logo=pypi&logoColor=white&color=3775A9" alt="PyPI version"/></a>
  <a href="https://pypi.org/project/orion-sdk/"><img src="https://img.shields.io/pypi/pyversions/orion-sdk?style=flat-square&logo=python&logoColor=white&color=3776AB" alt="Python version"/></a>
  <a href="https://pypi.org/project/orion-sdk/"><img src="https://img.shields.io/pypi/dm/orion-sdk?style=flat-square&color=3775A9" alt="PyPI downloads"/></a>
  <a href="https://pypi.org/project/orion-sdk/"><img src="https://img.shields.io/pypi/status/orion-sdk?style=flat-square" alt="PyPI status"/></a>
  <a href="https://github.com/zalcus/orion-sdk/blob/main/LICENSE"><img src="https://img.shields.io/github/license/zalcus/orion-sdk?style=flat-square&color=F5A623" alt="License"/></a>
  <br/><br/>
  <img src="https://img.shields.io/badge/OpenAI-GPT--4o%20%7C%20o1%20%7C%20o3-412991?style=flat-square&logo=openai"/>
  <img src="https://img.shields.io/badge/Anthropic-Claude%20Opus%204%20%7C%20Sonnet%204-D4A574?style=flat-square&logo=anthropic"/>
  <img src="https://img.shields.io/badge/Google-Gemini%202.5%20Pro%20%7C%20Flash-4285F4?style=flat-square&logo=google"/>
  <img src="https://img.shields.io/badge/OpenRouter-21%2B%20models-7C3AED?style=flat-square"/>
  <img src="https://img.shields.io/badge/Ollama-Local%20Models-3E8C72?style=flat-square"/>
</p>

---

## Why Orion SDK?

Every major AI platform ships its own client library. OpenAI has `openai`, Anthropic has `anthropic`, Google has `google-generativeai`. Orion gives you the same thing — but **unified**.

Instead of juggling 5 different SDKs with 5 different APIs, different error formats, and different tool-calling conventions, you get **one package, one interface, one `complete()` call**.

```python
from orion_sdk import OrionClient

client = OrionClient()
client.add_provider("anthropic", api_key="sk-ant-...")
client.add_provider("openai",   api_key="sk-...")

response = client.complete("Hello, world!")
print(response.content)
```

**That's it.** No provider lock-in, no API format memorization, no 300-page docs.

> [!TIP]
> Use provider aliases to save keystrokes — `"claude"` instead of `"anthropic"`, `"gpt"` instead of `"openai"`, `"local"` instead of `"ollama"`. See the full alias list [below](#provider-aliases).

---

## Features

- **5 providers, one API** — OpenAI, Anthropic, Google, OpenRouter, Ollama
- **Automatic fallback chains** — provider A fails? Try B, then C, automatically
- **Streaming support** — async iterator of chunks for real-time output
- **Tool/function calling** — unified format, auto-converts between provider schemas
- **Rate limiting** — per-provider token bucket with configurable RPM
- **Context window validation** — catches overflow before it hits the API
- **Token counting** — tiktoken when available, smart estimation otherwise
- **Custom providers** — subclass `Provider` and register it
- **Zero-lock-in** — swap providers by changing one string, not your codebase
- **Provider aliases** — `"claude"` → `"anthropic"`, `"gpt"` → `"openai"`, `"local"` → `"ollama"`

---

## Quick Start

> [!NOTE]
> You only need `pip install orion_sdk` for the core. Provider packages (`openai`, `anthropic`, `google-generativeai`) are optional — install only the ones you actually use.

### Install

```bash
pip install orion_sdk
```

### One-liner

```python
from orion_sdk import create_client

client = create_client("anthropic", api_key="sk-ant-...")
response = client.complete("What is 2 + 2?")
print(response.content)  # "4"
```

> [!IMPORTANT]
> Never hardcode API keys in your code. Use environment variables — Orion SDK automatically picks up `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, and `OPENROUTER_API_KEY` from your environment.

### Multi-provider with fallback

```python
from orion_sdk import OrionClient

client = OrionClient()
client.add_provider("anthropic", api_key="sk-ant-...")
client.add_provider("openai",   api_key="sk-...")
client.add_provider("google",   api_key="...")

# Try Anthropic → OpenAI → Google automatically
client.set_fallback_chain("anthropic", "openai", "google")

response = client.complete("Write a Python HTTP server")
print(response.content)
print(response.model)       # "claude-sonnet-4-20250514"
print(response.provider)     # "anthropic"
print(response.usage)        # {"prompt_tokens": 42, "completion_tokens": 187}
```

### Streaming

```python
from orion_sdk import OrionClient

client = OrionClient()
client.add_provider("openai", api_key="sk-...", set_default=True)

for chunk in client.stream("Tell me a story"):
    print(chunk.content, end="", flush=True)
```

### Tool calling

```python
from orion_sdk import OrionClient, ToolDefinition

client = OrionClient()
client.add_provider("anthropic", api_key="sk-ant-...")

tools = [
    ToolDefinition(
        name="get_weather",
        description="Get the current weather for a city",
        parameters={
            "type": "object",
            "properties": {
                "city": {"type": "string", "description": "City name"},
                "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
            },
            "required": ["city"]
        }
    )
]

response = client.complete("What's the weather in Tokyo?", tools=tools)

if response.has_tool_calls:
    for call in response.tool_calls:
        print(f"Call: {call.name}({call.arguments})")
        # {"city": "Tokyo", "unit": "celsius"}
```

---

## Supported Providers

| Provider | Package | Models | API Key |
|----------|---------|--------|---------|
| **OpenAI** | `openai` | GPT-4o, o1, o3, GPT-4o-mini, GPT-4-turbo | `OPENAI_API_KEY` |
| **Anthropic** | `anthropic` | Claude Opus 4, Claude Sonnet 4, Claude 3.5, Claude 3 | `ANTHROPIC_API_KEY` |
| **Google** | `google-generativeai` | Gemini 2.5 Pro, Gemini 2.5 Flash, Gemini 2.0, Gemini 1.5 | `GEMINI_API_KEY` |
| **OpenRouter** | `openai` | 21+ models (Claude, GPT, Gemini, Grok, Llama, DeepSeek, Qwen...) | `OPENROUTER_API_KEY` |
| **Ollama** | `requests` | Llama 3.3, Mistral, Qwen 2.5, CodeLlama, Phi3, Gemma2... | *None — fully local* |

### OpenRouter Models

Anthropic Claude Opus 4, Claude Sonnet 4, Claude 3.5 Haiku · OpenAI GPT-4o, o1, o3-mini · Google Gemini 2.5 Pro/Flash · xAI Grok 3 · Meta Llama 3.1 (8B / 70B / 405B) · Mistral Large · DeepSeek Chat / R1 · Qwen 2.5 · Perplexity Sonar

### Provider Aliases

```python
# These all work
client.add_provider("claude",   ...)  # → anthropic
client.add_provider("gpt",     ...)  # → openai
client.add_provider("gemini",  ...)  # → google
client.add_provider("local",   ...)  # → ollama
```

> [!TIP]
> Aliases also work in `complete()` and `stream()` — just pass `model="claude-opus-4"` and Orion resolves the provider automatically.

---

## API Reference

### `OrionClient`

```python
client = OrionClient(config={
    "default_provider": "anthropic",
    "default_model": "claude-sonnet-4-20250514",
    "timeout": 120.0,
    "max_retries": 3,
    "rate_limits": {"anthropic": 60, "openai": 120}
})
```

| Method | Description |
|--------|-------------|
| `add_provider(name, api_key, ...)` | Register a provider |
| `remove_provider(name)` | Remove a registered provider |
| `set_fallback_chain(*providers)` | Set failover priority |
| `complete(prompt, ...)` | Synchronous completion |
| `stream(prompt, ...)` | Streaming completion (async iterator) |
| `count_tokens(text, model)` | Count tokens |
| `count_messages_tokens(messages, model)` | Count tokens for a message list |
| `get_context_limit(model)` | Get model's context window size |
| `list_providers()` | List registered providers |
| `list_models(provider)` | List available models |
| `register_custom_provider(name, cls)` | Register a custom provider class |

### Data Types

```python
# Message
msg = Message.user("Hello!")
msg = Message.system("You are helpful.")
msg.to_dict()  # {"role": "user", "content": "Hello!"}

# Response
response.content        # str
response.tool_calls      # list[ToolCall]
response.model           # str
response.provider         # str
response.usage           # {"prompt_tokens": ..., "completion_tokens": ...}
response.has_tool_calls  # bool

# StreamChunk
chunk.content        # str
chunk.finish_reason   # str | None
chunk.model           # str

# ToolDefinition — auto-converts to provider formats
tool.to_openai()       # OpenAI schema
tool.to_anthropic()    # Anthropic schema
tool.to_gemini_tools() # Gemini schema
```

### Exceptions

```python
from orion_sdk import (
    OrionError,                # Base exception
    ProviderError,             # Error from a specific provider
    ProviderNotFoundError,     # Provider not registered
    AuthenticationError,       # Invalid API key
    RateLimitError,            # Rate limit hit
    ContextOverflowError,      # Input exceeds context window
    AllProvidersFailedError,   # All fallback providers failed
    TimeoutError,              # Request timed out
    InvalidConfigError,        # Bad configuration
)

try:
    response = client.complete(prompt)
except ContextOverflowError as e:
    print(f"Too many tokens: {e.tokens} > {e.limit}")
except AllProvidersFailedError as e:
    print(f"All failed:\n{e.errors}")
```

### Rate Limiting

```python
from orion_sdk import OrionClient, RateLimiter

client = OrionClient()
client.add_provider("anthropic", api_key="sk-ant-...")
client.add_provider("openai",   api_key="sk-...")

# Built-in rate limiter (default 60 req/min per provider)
# Or configure at init:
client = OrionClient(config={
    "rate_limits": {"anthropic": 50, "openai": 100}
})
```

---

## Custom Providers

```python
from orion_sdk import OrionClient, Provider, ProviderConfig, Response, Message
from orion_sdk.providers.base import StreamChunk

class MyCustomProvider(Provider):
    NAME = "custom"
    MODELS = {"custom-model-v1": {"context": 32000, "output": 4096}}

    def __init__(self, config: ProviderConfig):
        super().__init__(config)
        # Your client setup here

    def complete(self, messages, model="", tools=None, temperature=0.7, max_tokens=4096, **kwargs):
        model = self.validate_model(model)
        # Your API call here
        return Response(content="...", model=model, provider=self.NAME)

    def stream(self, messages, model="", tools=None, temperature=0.7, max_tokens=4096, **kwargs):
        model = self.validate_model(model)
        # Your streaming implementation here
        yield StreamChunk(content="...", model=model, provider=self.NAME)

    def list_models(self):
        return [{"id": m, "name": m, **info} for m, info in self.MODELS.items()]

# Register and use
client = OrionClient()
client.register_custom_provider("custom", MyCustomProvider)
client.add_provider("custom", api_key="...", set_default=True)
response = client.complete("Hello!")
```

---

## Architecture

```
orion_sdk/
├── __init__.py              # Public API — everything re-exported here
├── client.py                # OrionClient — the unified interface
├── exceptions.py            # 9 typed exceptions
├── ratelimit.py             # Token bucket rate limiter (thread-safe)
├── tokens.py                # Token counting + context window registry
└── providers/
    ├── __init__.py          # Provider registry + aliases
    ├── base.py              # Abstract Provider + Message/Tool/Response types
    ├── openai.py            # OpenAI GPT (also base for OpenAI-compatible)
    ├── anthropropic.py       # Anthropic Claude
    ├── google.py            # Google Gemini
    ├── openrouter.py        # OpenRouter (extends OpenAI provider)
    └── ollama.py            # Ollama local models (no API key)
```

Each provider implements three methods: `complete()`, `stream()`, and `list_models()`. `OrionClient` wraps them with fallback logic, rate limiting, and token management. Adding a new provider means writing one file and calling `register_provider()`.

---

## Context Window Limits

Built-in registry for 30+ models. Checked automatically before sending:

```python
client.get_context_limit("claude-opus-4-20250514")  # 200,000
client.get_context_limit("gpt-4o")                   # 128,000
client.get_context_limit("gemini-1.5-pro")            # 2,097,152
```

Raises `ContextOverflowError` if you exceed the limit — so you never waste tokens on a call that'll fail.

> [!WARNING]
> Context limits are checked **before** the API call. If your input exceeds the window, Orion raises immediately — no tokens wasted, no unexpected charges.

---

## Installation

```bash
# Core (only requires Python stdlib)
pip install orion_sdk

# With provider packages
pip install orion_sdk openai anthropic google-generativeai requests

# With token counting
pip install orion_sdk tiktoken
```

Each provider gracefully handles missing packages — if you don't install `anthropic`, only Anthropic calls will error. Everything else works fine.

> [!NOTE]
> Install `tiktoken` for accurate token counting. Without it, Orion falls back to character-based estimation (~4 chars per token) — good enough, but not exact.

---

## License

MIT

---

<p align="center">
  <sub>Built by <a href="https://github.com/zalcus">Zalcus</a></sub>
</p>
