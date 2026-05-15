# Session 19.1 — Foundations: What APIs Are & How to Call Them 🌐

## Session Overview

| Detail | Info |
|---|---|
| **Session** | 19.1 (Part 1 of 3) |
| **Topic** | What an API is, HTTP basics, JSON, the `requests` library, API keys |
| **Duration** | ~3 hours (teaching) + 15 min break + 20 min Q&A |
| **Audience** | Students who completed the OOP arc (16.1 → 18.3) |
| **Previous Module** | OOP — Classes, Inheritance, Decorators |
| **Next Session** | 19.2 — Talking to LLMs Properly |

## Learning Objectives

By the end of this session, you will be able to:

- ✅ Explain what an API is, in plain English
- ✅ Read an HTTP request and response — methods, headers, body, status code
- ✅ Recognize JSON and convert between it and Python dicts
- ✅ Use the `requests` library to make GET and POST calls
- ✅ Authenticate to an LLM API using an API key
- ✅ Store secrets safely with `.env` files (never hardcoded)
- ✅ Make your first real call to an LLM API and parse the response

## Why This Module Exists

You've spent 6+ sessions learning OOP — classes, methods, inheritance, decorators. That's the *internal* shape of code: how to organize logic inside one program.

Now we step outside. **APIs are how programs talk to other programs across the internet.** When your agent asks Claude a question, it's an API call. When your code searches the web, hits a database in the cloud, or sends an email — API call. Every modern AI system you'll build is a chain of API calls glued together.

So before we can build an agent, we have to learn how programs talk over the network. That's this module.

---

## Session Timeline

| Time | Topic | Duration |
|---|---|---|
| 0:00 - 0:25 | What Is an API? (no Python — analogies & diagrams) | 25 min |
| 0:25 - 0:55 | HTTP — Methods, Status Codes, Headers, Body | 30 min |
| 0:55 - 1:20 | JSON — The Lingua Franca of APIs | 25 min |
| 1:20 - 1:35 | ☕ Break | 15 min |
| 1:35 - 2:05 | Calling APIs in Python — the `requests` Library | 30 min |
| 2:05 - 2:30 | API Keys & `.env` Files | 25 min |
| 2:30 - 2:55 | Your First LLM API Call (real, working code) | 25 min |
| 2:55 - 3:00 | Recap & What's Next | 5 min |
| 3:00 - 3:20 | Q&A / Doubt Clearing | 20 min |

---

## Part 1: What Is an API? (25 min)

### 🤔 The Problem Before APIs

Imagine you're building a flight-booking app. To show flight prices, you need data from IndiGo, Air India, and Vistara. How do you get it?

You have two terrible choices:

1. **Beg each airline to email you a CSV every hour.** Stale data. Manual work. Scales to zero airlines beyond the first few.
2. **Each airline gives you direct access to their internal database.** They will absolutely not do this — it's a security catastrophe.

What we want is something in the middle: **a controlled doorway** through which any external program can ask "what's the price of Mumbai → Delhi tomorrow?" and get a clean, machine-readable answer back. The airline keeps full control over what data you can see and how often you can ask.

That doorway is called an **API** — Application Programming Interface.

### 🍽️ The Restaurant Analogy

The clearest mental model: an API is like the menu + waiter at a restaurant.

```
   YOU (your program)              KITCHEN (the other system)
        │                                │
        │   "I'd like #14 with no salt"  │
        ├─────── waiter ─────────────────►
        │                                │
        │                                ▼
        │                           [cooks it]
        │                                │
        ◄────── waiter ──────────────────┤
        │   here's your dish (or:        │
        │   "we're out of that")         │
        │                                │
```

You don't walk into the kitchen. You don't see how it's made. You don't even know if there are 3 chefs or 30. You just:
1. Look at the menu (the **API documentation**) — what's available, how to order it.
2. Tell the waiter (send a **request**).
3. Get either the dish or an error (the **response**).

The kitchen could rebuild itself entirely tomorrow — different chefs, different stove, different country — and as long as the menu stays the same, your ordering process doesn't change. **That's the power of APIs: they decouple the caller from the implementation.**

### What "API" Really Means in 2026

In modern software, "API" almost always means **HTTP API** — a service you talk to over the internet using HTTP. When someone says "OpenAI's API" or "the GitHub API," they mean: a set of URLs you can send requests to, and rules about how to format those requests and what comes back.

So really, "calling an API" = "sending an HTTP request to a URL and reading the response."

### What This Looks Like for AI

Every line below is a real API call:

| You write | What's actually happening |
|---|---|
| `client.chat.completions.create(...)` | POST to `https://api.openai.com/v1/chat/completions` |
| `Anthropic().messages.create(...)` | POST to `https://api.anthropic.com/v1/messages` |
| `bedrock.invoke_model(...)` | POST to an AWS Bedrock URL |
| `requests.get("https://...")` | GET to that URL |

The fancy SDKs you'll see (`openai`, `anthropic`, `boto3`) are just **convenience wrappers** around HTTP calls. Once you understand HTTP, every AI service in the world becomes accessible — even ones with no SDK.

> 💡 **Key Idea**: An API is a controlled doorway between programs. In modern AI work, "API" means "HTTP endpoint" 99% of the time. Master HTTP and you can talk to anything.

---

## Part 2: HTTP — Methods, Status Codes, Headers, Body (30 min)

HTTP (HyperText Transfer Protocol) is the language two programs use over the network. Every API call is built from four parts:

```
REQUEST                                  RESPONSE
─────────                                ─────────
1. Method (verb)        ─────────►       1. Status code
2. URL                                   2. Headers
3. Headers              ◄─────────       3. Body
4. Body
```

Let's break each part down with examples.

### Methods — What Verb Are You Using?

The method tells the server *what kind of action* you want. There are a few you'll meet constantly:

| Method | Meaning | Example use |
|---|---|---|
| `GET` | "Read this. I'm not changing anything." | Fetch a model's info, list files, search |
| `POST` | "Create something / do an action." | Send a chat message to an LLM, upload a file |
| `PUT` | "Replace this entire resource with what I'm sending." | Update a full record |
| `PATCH` | "Update part of this resource." | Change one field |
| `DELETE` | "Delete this resource." | Remove a file |

For LLM APIs, you'll be sending `POST` requests 95% of the time — every chat completion is a POST. The other 5% is `GET` for things like "list available models."

> 📝 **Note**: `GET` requests are supposed to be *safe* (read-only) and *idempotent* (same result every time). `POST` is for changes and side effects. LLM calls use `POST` because they're not idempotent — the model returns different text each time, and they cost money.

### URL — Where Are You Sending It?

A URL has structure:

```
https://api.openai.com/v1/chat/completions?model=gpt-4
└─┬─┘   └────────┬───────┘└───────┬───────┘└────┬────┘
scheme    host (server)      path           query string
```

- **scheme** — `https` (encrypted) or `http` (not). Always use `https` in production.
- **host** — the server you're talking to.
- **path** — what specific resource on that server.
- **query string** — optional `?key=value&key=value` parameters, mostly for `GET`.

### Headers — Metadata About the Request

Headers are key-value pairs that go *with* every request, separate from the body. Think of them as the envelope on a letter — they describe the message without being the message itself.

```
Authorization: Bearer sk-abc123...           ← who's calling (auth)
Content-Type: application/json               ← what format the body is in
Accept: application/json                     ← what format we want back
User-Agent: my-agent/1.0                     ← who I am (optional)
```

For AI APIs, the two headers you'll always set are:
- **`Authorization`** — proves you're allowed to make this call (your API key).
- **`Content-Type: application/json`** — tells the server "the body I'm sending is JSON."

### Body — The Actual Data

The body is the payload — the actual content you're sending. For `POST` requests to LLM APIs, the body is a JSON object describing what you want:

```json
{
  "model": "gpt-4",
  "messages": [
    {"role": "user", "content": "What is HTTP?"}
  ]
}
```

`GET` requests usually don't have a body — they put their parameters in the URL query string instead.

### Status Codes — Did It Work?

Every response comes back with a 3-digit status code. The first digit tells you the category:

| Range | Meaning | Common ones |
|---|---|---|
| `2xx` | ✅ Success | `200 OK`, `201 Created` |
| `3xx` | ↪️ Redirect | `301 Moved`, `304 Not Modified` |
| `4xx` | ❌ Client error (your fault) | `400 Bad Request`, `401 Unauthorized`, `404 Not Found`, `429 Too Many Requests` |
| `5xx` | 💥 Server error (their fault) | `500 Internal Server Error`, `503 Service Unavailable` |

The ones you'll meet most often when building agents:

| Code | Meaning | What you should do |
|---|---|---|
| `200` | Success | Use the response |
| `400` | Your request was malformed | Fix your code; don't retry |
| `401` | Missing or wrong API key | Fix your auth; don't retry |
| `403` | Authenticated but not allowed | Don't retry — your account doesn't have access |
| `404` | URL doesn't exist | Check the endpoint URL |
| `429` | Rate limit hit — too many requests | Wait, then retry |
| `500/502/503/504` | Server problem on their end | Wait, then retry |

> 💡 **Key Idea**: `4xx` = "you did something wrong, fix it" (don't retry blindly). `5xx` and `429` = "transient problem, retrying might help."

### Putting It Together — An Example Request/Response

A real `POST` to OpenAI's chat completions endpoint:

```
POST /v1/chat/completions HTTP/1.1
Host: api.openai.com
Authorization: Bearer sk-abc123...
Content-Type: application/json

{"model": "gpt-4", "messages": [{"role": "user", "content": "Hi!"}]}
```

And the response:

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "chatcmpl-...",
  "choices": [{
    "message": {"role": "assistant", "content": "Hello! How can I help?"}
  }]
}
```

Status code `200`, JSON body containing the assistant's reply. That's an LLM call. Stripped of all SDK magic, it's literally just this.

---

## Part 3: JSON — The Lingua Franca of APIs (25 min)

### Why JSON?

Every API in our world (OpenAI, Anthropic, Bedrock, GitHub, Stripe, you name it) sends and receives **JSON** in the body. Not XML. Not CSV. JSON.

JSON (JavaScript Object Notation) is a simple text format for structured data. It's the lowest-common-denominator way to represent dicts, lists, strings, numbers, booleans, and `null`. Every programming language can produce and parse it, which is why APIs picked it.

### JSON ↔ Python

The mapping is almost 1:1 — that's why JSON feels natural in Python:

| JSON | Python |
|---|---|
| `{}` | `dict` |
| `[]` | `list` |
| `"hello"` | `str` |
| `42` / `3.14` | `int` / `float` |
| `true` / `false` | `True` / `False` |
| `null` | `None` |

A JSON document for an LLM request:

```json
{
  "model": "gpt-4",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "What is HTTP?"}
  ],
  "temperature": 0.7,
  "stream": false
}
```

The same thing in Python:

```python
{
    "model": "gpt-4",
    "messages": [
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is HTTP?"},
    ],
    "temperature": 0.7,
    "stream": False,
}
```

Notice the differences are tiny: `true` → `True`, `false` → `False`, `null` → `None`. Otherwise it's identical.

### Converting Between Them — `json.dumps` and `json.loads`

Python's standard library gives you two functions you'll use thousands of times:

```python
import json

# Python dict → JSON string (for sending in a request body)
payload = {"model": "gpt-4", "temperature": 0.7}
body = json.dumps(payload)
print(body)
# '{"model": "gpt-4", "temperature": 0.7}'

# JSON string → Python dict (for parsing a response body)
response_text = '{"id": "abc", "choices": [{"text": "Hello"}]}'
data = json.loads(response_text)
print(data["choices"][0]["text"])
# Hello
```

Mnemonic: `dumps` = "**dump** to **s**tring", `loads` = "**load** from **s**tring". (`dump`/`load` without the `s` work directly with files.)

### One Gotcha — Trailing Commas Are Illegal in JSON

```json
{"a": 1, "b": 2,}    ← ❌ Invalid JSON. Last comma kills it.
```

Python is forgiving about trailing commas in dicts/lists. JSON is not. If you're hand-writing JSON, double-check.

### Why You'll Rarely Touch `json.dumps` Directly

In a moment we'll see the `requests` library, which has a shortcut: pass `json=...` and it serializes for you. But knowing what's happening underneath matters — when something breaks, you'll need to know that "the request body is just a JSON string."

> 💡 **Key Idea**: JSON is text. Networks send text. Python objects can be turned into JSON text and back. Every API call boils down to "serialize a dict to JSON, send it, parse the JSON response back into a dict."

---

## ☕ Break (15 min)

---

## Part 4: Calling APIs in Python — the `requests` Library (30 min)

### Why `requests`?

Python's standard library has `urllib`, but no one uses it for HTTP — it's painful. Everyone uses `requests`, the de-facto standard:

```bash
pip install requests
```

`requests` makes HTTP calls feel like function calls. One line per request.

### A Simple GET — Hello, Internet

Let's start with the simplest thing — a public API that doesn't require auth. The free `httpbin.org` echoes whatever you send it, useful for learning.

```python
import requests

response = requests.get("https://httpbin.org/get")

print(response.status_code)    # 200
print(response.headers)        # {'Content-Type': 'application/json', ...}
print(response.text)           # raw JSON string
print(response.json())         # parsed into a Python dict
```

Read it carefully. Four things you almost always inspect on a response:

| Attribute | What it gives you |
|---|---|
| `response.status_code` | The HTTP status code (200, 429, 500, ...) |
| `response.headers` | Response headers (a dict-like object) |
| `response.text` | The raw body as a string |
| `response.json()` | The body parsed as JSON (calls `json.loads` for you) |

### POST With a JSON Body

For LLM calls, you send `POST` requests with JSON bodies:

```python
import requests

response = requests.post(
    "https://httpbin.org/post",
    json={"hello": "world", "n": 42},   # ← `requests` JSON-serializes this for you
    headers={"X-Custom": "demo"},       # ← any extra headers
)

print(response.status_code)            # 200
print(response.json())
# {
#   "args": {},
#   "data": '{"hello": "world", "n": 42}',
#   "headers": {"X-Custom": "demo", "Content-Type": "application/json", ...},
#   ...
# }
```

The magic: passing `json=...` does three things at once:
1. Calls `json.dumps(...)` on your dict.
2. Sets the body to the resulting string.
3. Adds the header `Content-Type: application/json`.

That's why we love `requests`.

### Query Parameters — `params=`

For `GET` calls that need parameters in the URL query string, use `params=`:

```python
response = requests.get(
    "https://httpbin.org/get",
    params={"q": "agents", "limit": 10},
)

print(response.url)
# https://httpbin.org/get?q=agents&limit=10
```

`requests` does the URL-encoding for you (handles spaces, special characters, etc.).

### Timeouts — Always Set Them

Here's a thing junior code always forgets: **`requests` will wait forever by default** if the server stops responding. In production this means a stuck process. Always pass `timeout=`:

```python
response = requests.get("https://httpbin.org/delay/2", timeout=5)   # 5 sec max
```

If the server doesn't respond in time, you get a `requests.exceptions.Timeout` exception. We'll cover handling that properly in 19.3.

### A Tiny Utility Function — Setting the Pattern

For LLM work, you'll write the same shape of call over and over:

```python
import requests

def post_json(url: str, body: dict, headers: dict | None = None, timeout: int = 30) -> dict:
    response = requests.post(url, json=body, headers=headers or {}, timeout=timeout)
    response.raise_for_status()         # turns 4xx/5xx into exceptions
    return response.json()
```

`response.raise_for_status()` is a friend — it raises an exception on `4xx`/`5xx` so you don't silently process a 401 as if it were valid.

We'll grow this tiny function into a full `LLMClient` class over the next two sessions. **This is the seed of `v2_oop/llm_client.py`.**

> 💡 **Key Idea**: `requests.get(url)` and `requests.post(url, json=body, headers=...)` cover ~90% of what you'll ever need. Always set a `timeout`. Always check `status_code` (or call `raise_for_status()`).

---

## Part 5: API Keys & `.env` Files (25 min)

### 🤔 The Problem — Your First Real LLM Call Will Be Rejected

Try calling Anthropic's API without authentication:

```python
import requests

r = requests.post(
    "https://api.anthropic.com/v1/messages",
    json={"model": "claude-3-5-sonnet-20241022", "messages": [{"role": "user", "content": "Hi"}]},
)
print(r.status_code, r.text)
# 401 {"error": {"type": "authentication_error", "message": "..."}}
```

`401 Unauthorized`. The API has no idea who you are. Why would it spend compute on a stranger?

That's where **API keys** come in. An API key is a long secret string the provider gives you when you sign up. You include it on every request, and the server uses it to:
1. Know who you are (so they can bill you).
2. Check you're allowed to call this endpoint.
3. Track your rate limits and usage.

### The `Authorization: Bearer ...` Pattern

Almost every modern API uses the same auth header pattern:

```
Authorization: Bearer sk-ant-api03-abc123...
```

`Bearer` literally means "the holder of this token is allowed in." Whoever has the key can use the API as you. **That's why protecting it is critical.**

In Python:

```python
headers = {
    "Authorization": f"Bearer {api_key}",
    "Content-Type": "application/json",
}
```

Anthropic uses a slightly different header (`x-api-key` instead of `Authorization`). OpenAI uses the standard `Authorization: Bearer`. Bedrock uses AWS signing (more complex — that's why we use `boto3` for it). The SDK quietly handles whichever style — under the hood it's just headers.

### 🚨 The Security Problem — Don't Hardcode Keys

The naive approach:

```python
api_key = "sk-ant-api03-abcdef123456..."   # ❌ NEVER DO THIS
```

Why this is dangerous:
1. **Git history is forever.** The moment you commit a key, it's in the repo's history. Even if you delete it later, anyone with access to the repo can pull the old commit.
2. **Public repos = scanned in seconds.** Bots constantly scan GitHub for leaked keys. There are documented cases of keys being abused within minutes of being pushed.
3. **You can't share code.** Every collaborator would see your key.
4. **You can't rotate easily.** Changing the key means editing every file that hardcodes it.

The fix: keep secrets **outside** the codebase, in environment variables.

### Environment Variables & `.env` Files

An environment variable is a key=value pair set on your computer (or server), readable from any program. Python reads them via `os.getenv`:

```python
import os
api_key = os.getenv("ANTHROPIC_API_KEY")
```

You could set them in your shell every time:

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
```

But typing that for every project is annoying. The standard pattern is a `.env` file at the project root:

**.env** (this file is git-ignored — never committed)
```
ANTHROPIC_API_KEY=sk-ant-api03-abc123...
OPENAI_API_KEY=sk-...
```

**.gitignore**
```
.env
```

Then load it in Python with `python-dotenv`:

```bash
pip install python-dotenv
```

```python
from dotenv import load_dotenv
import os

load_dotenv()                                  # reads .env into the environment
api_key = os.getenv("ANTHROPIC_API_KEY")       # now this works
```

That's the whole pattern. Used in essentially every Python AI project.

### Checklist — Secrets Hygiene

| ✅ Do | ❌ Don't |
|---|---|
| Store keys in `.env` | Hardcode keys in source files |
| Add `.env` to `.gitignore` | Commit `.env` |
| Use `os.getenv("KEY")` | Use `KEY = "sk-..."` |
| Rotate keys if they leak | Push and hope no one noticed |
| Use a separate key per project | Reuse one key everywhere |

> 🚨 **Production Warning**: If you EVER accidentally commit a key, **immediately revoke it** in the provider's dashboard and generate a new one. Don't try to "remove" it from git history — assume it's already been scraped.

> 💡 **Key Idea**: Code goes in git. Secrets go in `.env`. Never the twain shall meet. Read keys via `os.getenv(...)` so the same code runs in dev, test, and production with different keys.

---

## Part 6: Your First LLM API Call (25 min)

Putting it all together — let's make a real call to Anthropic's Claude. (You can do the same with OpenAI; the shape is identical.)

### Setup

1. Sign up at [console.anthropic.com](https://console.anthropic.com), create an API key.
2. Save it to `.env` as `ANTHROPIC_API_KEY=...`.
3. Add `.env` to your `.gitignore`.
4. `pip install requests python-dotenv`.

### The Code

```python
import os
import requests
from dotenv import load_dotenv

load_dotenv()
api_key = os.getenv("ANTHROPIC_API_KEY")

if not api_key:
    raise RuntimeError("ANTHROPIC_API_KEY not set. Check your .env file.")

url = "https://api.anthropic.com/v1/messages"

headers = {
    "x-api-key": api_key,
    "anthropic-version": "2023-06-01",
    "content-type": "application/json",
}

body = {
    "model": "claude-3-5-sonnet-20241022",
    "max_tokens": 1024,
    "messages": [
        {"role": "user", "content": "Explain APIs in one sentence."}
    ],
}

response = requests.post(url, headers=headers, json=body, timeout=30)
response.raise_for_status()

data = response.json()
print(data["content"][0]["text"])
```

Run it. You'll see something like:

```
APIs are structured doorways that let one program request data or actions from another program over the network.
```

### Walk Through What Happened

1. **Loaded the API key** from `.env` via `load_dotenv()` + `os.getenv(...)`.
2. **Set the URL** to Anthropic's `/v1/messages` endpoint.
3. **Set headers** — auth (`x-api-key`), API version, content type.
4. **Built the body** — model ID, max tokens, list of messages.
5. **Sent a POST** with `requests.post(...)`. Always with a timeout.
6. **Checked status** with `raise_for_status()` — explodes if anything went wrong.
7. **Parsed JSON** with `response.json()`, then dug into the structure to get the text.

That's it. **That is an LLM API call.** The fancy `anthropic` Python SDK does exactly the same thing under the hood; you've just removed the wrapper.

### What If It Failed?

Try things that go wrong on purpose. Real production code has to handle them:

```python
# Wrong key
headers["x-api-key"] = "wrong"
# → 401 Unauthorized

# Bad model name
body["model"] = "claude-9000-ultra"
# → 400 Bad Request

# No internet (turn off WiFi)
# → requests.exceptions.ConnectionError

# Server is slow (use a tiny timeout)
requests.post(url, headers=..., json=..., timeout=0.001)
# → requests.exceptions.Timeout

# You hit your rate limit (call it 100x in a loop)
# → 429 Too Many Requests
```

Right now we'd just crash on any of these. In **19.3** we'll build proper handling — retries, custom exceptions, rate limiting. Don't worry about it yet; just notice that a real client has to plan for all of these.

### 🔗 Looking Back at the OOP Module

Notice the small problem already creeping in — we have:
- An API key (config)
- A model name (config)
- Headers (built from config)
- A function that uses all of these

If we wanted to make 5 different calls in a script, we'd be passing `api_key, model, headers` around everywhere. Sound familiar? It's the same pain that motivated `__init__` back in 16.1: "store this once, use it many times." That's why we'll wrap all of this in an `LLMClient` class in 19.3 — and now you understand exactly why classes exist for this kind of code.

> 💡 **Key Idea**: An LLM call is just a `POST` to a URL with a JSON body and an auth header. SDKs wrap this for convenience, but the underlying mechanism is what you just wrote.

---

## 🧪 Try It

1. **Different model.** Change the model to `claude-3-5-haiku-20241022` and rerun. (Haiku is faster and cheaper — useful for quick experiments.)

2. **System prompt.** Add a `"system"` field at the top level of `body`: `"system": "You are a sarcastic assistant."` Rerun. (We'll dive deep on system prompts in 19.2.)

3. **OpenAI variant.** Repeat the call against OpenAI. URL: `https://api.openai.com/v1/chat/completions`. Auth header: `Authorization: Bearer ${OPENAI_API_KEY}`. Body shape is similar but the response structure differs. Find where the text is in the response — that's a key skill: navigating JSON responses.

4. **Status-code logger.** Wrap the call in a `try/except`. Print the status code and a short message for each kind of failure (`401`, `429`, `5xx`, `Timeout`, `ConnectionError`).

5. **Echo server.** Make a `POST` to `https://httpbin.org/post` with any JSON. Inspect `response.json()` — it tells you exactly what your request looked like. Useful for debugging.

---

## Recap

- **An API is a controlled doorway** — programs talk to programs without exposing internals.
- **HTTP** is the protocol. Every call has a method, URL, headers, and (sometimes) body. Every response has a status code, headers, and body.
- **JSON** is the data format. `json.dumps` to serialize, `json.loads` to parse. Python dicts and JSON objects map ~1:1.
- **`requests`** is the Python library. `requests.get(url)` and `requests.post(url, json=..., headers=..., timeout=...)` are the bread and butter.
- **API keys** authenticate you. Never hardcode — use `.env` + `os.getenv()`. Add `.env` to `.gitignore`.
- **An LLM call is just a POST** to the provider's URL with a JSON body and an auth header. SDKs are convenience wrappers around this.

> ⚠️ **Common Mistakes**
> 1. No `timeout=` → process hangs forever when a server stalls.
> 2. Hardcoding API keys → leak, abuse, key revocation, panic.
> 3. Not checking `response.status_code` → silently processing error responses as data.
> 4. Forgetting `Content-Type: application/json` → server thinks you're sending form data.
> 5. Trailing commas in hand-written JSON → parser error.
> 6. Confusing `4xx` (your fault, fix it) and `5xx` (their fault, retry).

> 💡 **The One Thing to Remember**: An API call is `requests.post(url, json=body, headers=auth, timeout=N)`. Everything else — SDKs, retries, streaming, agents — is built on top of that one line.

---

## 🔗 What's Next — Session 19.2

We can now talk to an LLM. But our message structure was minimal — one `user` message and a reply. Real agents need much more:

- **System prompts** that define the assistant's role and behavior.
- **Multi-turn conversations** where the model remembers previous turns.
- **Structured output** so the model returns clean JSON we can parse, not free-form prose.
- **Streaming** so the user sees tokens appear as they're generated, instead of waiting 30 seconds for a full reply.
- **Knobs** — `temperature`, `max_tokens`, `top_p` — that control the model's behavior.

In **19.2 — Talking to LLMs Properly**, we'll cover all of these. By the end, your code will look and feel like a real chat app — not just a one-shot API ping.

See you in **19.2**.

---

## Quick Reference Card

```python
# ── HTTP method cheat sheet ──────────────────────
# GET    → read (no body)            → list models, search
# POST   → create / action            → send chat, upload file
# PUT    → replace whole resource     → update full record
# PATCH  → modify part                → change one field
# DELETE → remove resource            → delete file

# ── Status code categories ───────────────────────
# 2xx → success     | 200 OK
# 4xx → your fault  | 400, 401, 403, 404, 429
# 5xx → their fault | 500, 502, 503, 504

# ── JSON ↔ Python ────────────────────────────────
import json
body = json.dumps({"a": 1})        # dict  → JSON string
data = json.loads('{"a": 1}')      # JSON  → dict

# ── requests basics ──────────────────────────────
import requests

requests.get(url, params={...}, headers={...}, timeout=30)
requests.post(url, json={...}, headers={...}, timeout=30)

response.status_code        # int
response.headers            # dict-like
response.text               # raw string
response.json()             # parsed dict / list
response.raise_for_status() # raises on 4xx/5xx

# ── Secrets ──────────────────────────────────────
# .env file (git-ignored):
#    ANTHROPIC_API_KEY=sk-ant-...
#    OPENAI_API_KEY=sk-...

from dotenv import load_dotenv
import os
load_dotenv()
key = os.getenv("ANTHROPIC_API_KEY")

# ── LLM call template ────────────────────────────
import requests, os
from dotenv import load_dotenv
load_dotenv()

url = "https://api.anthropic.com/v1/messages"
headers = {
    "x-api-key": os.getenv("ANTHROPIC_API_KEY"),
    "anthropic-version": "2023-06-01",
    "content-type": "application/json",
}
body = {
    "model": "claude-3-5-sonnet-20241022",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello!"}],
}

r = requests.post(url, headers=headers, json=body, timeout=30)
r.raise_for_status()
print(r.json()["content"][0]["text"])
```
