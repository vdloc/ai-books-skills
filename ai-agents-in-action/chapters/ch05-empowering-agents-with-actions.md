# Chapter 5: Empowering Agents with Actions

## Core Idea
Actions (OpenAI function/tool calling, and Microsoft Semantic Kernel's semantic + native functions/plugins) are how agents reach outside themselves — the LLM never executes a function itself, it only selects one and extracts parameters; your code executes it and feeds results back; Semantic Kernel formalizes this into reusable, composable plugins and lets you build a "GPT interface" (a semantic layer) over any existing API.

## Frameworks Introduced
- **OpenAI Function/Tool Calling**: register a `tools` array of function specs (name, description, JSON-schema parameters) on a chat completion request.
  - When to use: whenever an agent needs to trigger external code/data based on natural language.
  - How: LLM returns `tool_calls` (name + JSON args) — it does not execute anything; your code matches the name to a real function, executes it, appends the result as a `role: "tool"` message, and sends a second request so the LLM can phrase a natural-language reply. Simpler models (e.g., GPT-3.5) suffice for the delegation step itself.
- **Semantic Kernel (SK)**: Microsoft's orchestration layer for "semantic functions" (prompt templates, type `completion`) and "native functions" (real code, type `function`), both registrable as plugins/skills in a `Kernel`.
  - When to use: when you want cleaner, reusable, composable actions than hand-rolling the OpenAI tool-call loop — especially when combining prompt templates with code, or exposing multiple plugins to a chat/agent interface.
  - How: create a `Kernel`, add an AI service (`OpenAIChatCompletion`/`AzureChatCompletion`), define semantic functions via `PromptTemplateConfig` + `create_function_from_prompt` (with `{{$var}}` placeholders) or native functions via `@kernel_function`-decorated class methods, then `kernel.import_plugin_from_object` / `import_plugin_from_prompt_directory` to register as plugins; invoke via `kernel.invoke(...)`.
- **Semantic Service Layer / "GPT interface"**: wrap an existing REST API's endpoints as `@kernel_function`-decorated methods on a service class, turning any API into natural-language-callable plugins.
  - When to use: to expose any existing web API (e.g., TMDB) to a chat/agent interface without building new middleware from scratch.
  - How: one class per API, one method per meaningful endpoint, each decorated with `@kernel_function(description=..., name=...)`; import the class as a plugin (`import_plugin_from_object`); the kernel/LLM can now call these functions from a conversation.
- **Embedding functions inside functions**: a semantic function's prompt template can reference a native (or other semantic) function by `{{ClassName.function_name}}`.
  - When to use: to inject dynamic data (e.g., a user's seen-movies list) directly into a prompt template without manual pre-fetching in application code.
  - How: reference `{{MySeenMoviesDatabase.LoadSeenMovies}}` inside `skprompt.txt`/prompt string; only the native function needs to be registered with the kernel — the semantic function referencing it does not need separate registration to work inline.

## Key Concepts
- **Action**: this book's umbrella term for any function/skill/tool/plugin an agent can invoke — different frameworks name it differently.
- **Plugin (formerly "skill")**: SK's term for a registered, invokable function (semantic or native) exposed to a kernel/chat interface; OpenAI's plugin spec maps directly to its function-calling JSON schema.
- **Semantic function**: a prompt template (type `completion`) executed against an LLM — SK's equivalent of a Prompt Flow prompt template.
- **Native function**: real code (type `function`) that can do anything — API calls, file I/O, scraping.
- **Context/template variables**: `{{$var}}` placeholders in a prompt template, filled via `KernelArguments`.
- **`tool_choice="auto"`**: the default setting letting the LLM decide whether/which registered tool to call.
- **Two-step function-calling loop**: request 1 gets tool_calls; your code executes them; request 2 (with tool results appended as `role: "tool"` messages) gets the natural-language reply.
- **"Think semantically" principle**: return rich structured data (e.g., full JSON) from a semantic service function rather than pre-filtered plain text, so the LLM itself can filter/sort/transform — pre-truncating data denies the LLM its main strength.

## Mental Models
- Use OpenAI's raw `tools`/function-calling loop for a single quick integration; reach for Semantic Kernel once you have multiple plugins, need composition (functions calling functions), or want a reusable semantic service layer.
- Use native functions for anything that must reliably execute code/IO; use semantic functions for anything that's fundamentally "ask an LLM to produce/transform text well."
- When wrapping an API as a semantic service, return maximal structured data (JSON) from each function — let the LLM be the filter/formatter, not your code.

## Anti-patterns
- **Assuming the LLM executes the function**: it only returns a suggested function name + parsed arguments; failing to build the execution + result-feeding-back loop breaks the pattern entirely.
- **Over-filtering data before returning it from an action**: returning only movie titles (v1 `TMDbService`) instead of full JSON (v2) prevents the LLM from doing genre/overview-based filtering the user actually asked for — a direct illustration of the "think semantically" anti-pattern.
- **Assuming function *creation* implies registration**: in SK, creating a semantic function does not automatically register/expose it as a plugin — only imported/registered functions are callable as plugins from chat.
- **Using function calling with local/open-source LLMs expecting parity**: at time of writing, function calling was a commercial-LLM-only capability, not universally available for local deployments.

## Code Examples
```python
tool_calls = response_message.tool_calls
if tool_calls:
    available_functions = {"recommend": recommend}
    for tool_call in tool_calls:
        function_name = tool_call.function.name
        function_to_call = available_functions[function_name]
        function_args = json.loads(tool_call.function.arguments)
        function_response = function_to_call(
            topic=function_args.get("topic"),
            rating=function_args.get("rating"),
        )
        messages.append({
            "tool_call_id": tool_call.id,
            "role": "tool",
            "name": function_name,
            "content": function_response,
        })
    second_response = client.chat.completions.create(
        model="gpt-3.5-turbo-1106", messages=messages,
    )
    return second_response.choices[0].message.content
```
- **What it demonstrates**: the canonical two-step tool-calling loop — execute each returned tool call locally, append results as `tool`-role messages, then re-call the LLM to get a natural-language synthesis of all results (used here for 3 parallel recommendation requests in one user turn).

## Reference Tables
| Function type (SK) | `type` in config | Encapsulates | Registered how |
|---|---|---|---|
| Semantic function | `completion` | Prompt template | `create_function_from_prompt` + import to register as plugin |
| Native function | `function` | Real code | `@kernel_function` decorator + `import_plugin_from_object` |

| Version | `get_top_movies_by_genre` returns | LLM can filter/sort further? |
|---|---|---|
| v1 (`tmdb.py`) | Comma-joined titles only | No — data already collapsed |
| v2 (`tmdb_v2.py`) | Full JSON movie objects | Yes — LLM applies overview/genre filters (e.g., "movies about space") |

## Worked Example
The chapter builds a full movie-recommendation "GPT interface" over the free TMDB API: (1) a `TMDbService` class wraps endpoints like `get_movie_genre_id` and `get_top_movies_by_genre` as `@kernel_function`-decorated methods; (2) `test_tmdb_service.py` calls each function directly through the kernel to validate it; (3) `SK_service_chat.py` builds an interactive chat loop where a `ChatBot` semantic function is excluded from the plugin set (so it's the pure conversational entry point) while `TMDBService` functions are exposed as callable tools — a query like "top action and comedy movies" triggers two internal calls to `get_top_movies_by_genre` (with nested `get_movie_genre_id` lookups) and returns a merged natural-language list; (4) switching to `tmdb_v2.py` (JSON-returning) lets a follow-up query ("...only return movies about space") get correctly filtered by the LLM itself using each movie's overview text, which v1's title-only output could never support.

## Key Takeaways
1. Function/tool calling is a two-step loop: the LLM only *selects* and *parses* — your code must execute and feed results back for a final natural-language reply.
2. Simpler/cheaper models are sufficient for the tool-selection step; reserve stronger models for synthesis if cost matters.
3. Semantic Kernel cleanly separates semantic functions (prompt templates) from native functions (code), and both compose as plugins — letting you embed a native function's output directly inside a semantic prompt template.
4. Creating a function is not the same as registering it — only imported/registered functions become callable plugins.
5. A "GPT interface" (semantic service layer) can wrap any existing REST API in an afternoon: one class, one `@kernel_function`-decorated method per endpoint.
6. Design actions to return rich structured data, not pre-filtered text — this is what lets the LLM apply its own reasoning/filtering downstream ("thinking semantically").
7. Actions are the concrete mechanism behind the "actions" component of the five-part agent model from chapter 1 — every multi-agent platform in chapter 4 and chapter 7's Nexus platform builds on this same tool-calling foundation.

## Connects To
- **Ch1**: implements the "actions" component of the five-component agent model.
- **Ch4**: AutoGen/CrewAI skills and tools are built on the same function/tool-calling foundation shown here in raw form.
- **Ch8**: the native function used to load "seen movies" from a file is an early, informal preview of the memory/knowledge systems covered in depth later.
- **Ch9**: Semantic Kernel's prompt templates parallel Microsoft Prompt Flow's prompt-template-plus-evaluation approach.
