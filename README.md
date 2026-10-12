# Deploy and Host Open WebUI + LiteLLM on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/open-webui-litellm?utm_medium=integration&utm_source=button&utm_campaign=open-webui-litellm)

[Open WebUI](https://openwebui.com/) is the self-hosted ChatGPT-style interface for teams: chats, shared prompts, document and knowledge-base search (RAG), web search, tools and per-user permissions. [LiteLLM](https://www.litellm.ai/) is an open-source AI gateway that puts OpenAI, Anthropic, Gemini, OpenRouter, Bedrock, Azure and 100+ other providers behind one OpenAI-compatible API, with virtual keys, budgets and spend tracking. This template wires them together: every model you add in LiteLLM shows up in Open WebUI for your whole team.

## About Hosting Open WebUI + LiteLLM

The stack is three services: Open WebUI, LiteLLM and Postgres.

- **Upstream images, pinned.** Open WebUI runs from `ghcr.io/open-webui/open-webui:v0.12.0` (the plain image, no bundled Ollama) and LiteLLM from `ghcr.io/berriai/litellm:v1.105.0`, its current stable release. A redeploy never pulls an untested version; upgrading is bumping the tag.
- **Wired over the private network.** Open WebUI talks to LiteLLM at its private address using LiteLLM's master key, so no model traffic leaves Railway's network between the two.
- **Models are managed in LiteLLM.** `STORE_MODEL_IN_DB` is on, so you add models, provider keys, virtual keys and budgets in the LiteLLM admin UI; they are stored in Postgres, and provider keys are encrypted with a generated salt key.
- **One Postgres, two databases.** Open WebUI keeps users, chats and settings in the `openwebui` database; LiteLLM keeps models, keys and spend logs in its own `litellm` database, created on Postgres's first boot. Both run their migrations on start.
- **Uploads on a volume.** Uploaded files, the document-search index and the small local embedding model live on Open WebUI's volume at `/app/backend/data`.
- **Every secret generated.** The LiteLLM master key and salt key, Open WebUI's session secret and the database password are generated per deployment.

## Common Use Cases

- A private ChatGPT for a team or company, with access to GPT, Claude, Gemini and open models in one place
- Chat with your documents: upload files or build knowledge bases and ask questions about them
- An internal AI gateway: give other apps and developers LiteLLM virtual keys with budgets and rate limits instead of raw provider keys
- Track AI spend per user, team and key across providers

## Dependencies for Open WebUI + LiteLLM Hosting

- Postgres 17 (included, private network only)
- At least one model provider API key (OpenAI, Anthropic, OpenRouter, Google AI Studio or any provider LiteLLM supports)

### Deployment Dependencies

- [Open WebUI documentation](https://docs.openwebui.com/)
- [LiteLLM proxy documentation](https://docs.litellm.ai/docs/simple_proxy)
- [Open WebUI on GitHub](https://github.com/open-webui/open-webui)
- [LiteLLM on GitHub](https://github.com/BerriAI/litellm)
- [Template source on GitHub](https://github.com/nomideusz/open-webui-litellm-railway)

### Implementation Details

**Create the admin account first.** Open the Open WebUI service's Railway domain as soon as the deploy is green and sign up: the first account becomes the admin. Until then anyone who finds the URL could claim it. Open WebUI then switches public sign-up off: add teammates in Admin Panel > Users, or turn sign-up back on in Admin Settings > General (new accounts wait as "pending" until you approve them).

**Add your first model.** Open the LiteLLM service's domain at `/ui` and log in with username `admin` and the `LITELLM_MASTER_KEY` value (LiteLLM service > Variables). Under Models, add a model: pick a provider and model, paste the provider key (or leave the key empty if you set that provider's key as a variable on the LiteLLM service, e.g. `OPENAI_API_KEY`), and save. Choosing a wildcard such as "All OpenAI models" adds a whole provider at once. Reload Open WebUI and the model is in the model picker.

**Provider keys as variables (optional).** `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `OPENROUTER_API_KEY` and `GEMINI_API_KEY` on the LiteLLM service are used by any model you add without its own key. Any other provider's standard variable works the same way.

**LiteLLM for other apps.** The LiteLLM domain is a full OpenAI-compatible API (`https://<litellm-domain>/v1`). Create virtual keys with budgets and model limits under Virtual Keys rather than handing out the master key.

**Keep the keys.** `LITELLM_SALT_KEY` encrypts the provider keys stored in the database: back it up and never change it. Changing `LITELLM_MASTER_KEY` also requires updating `OPENAI_API_KEY` on Open WebUI (it references the master key, so a redeploy of Open WebUI picks it up) and in Open WebUI's Admin Settings > Connections, which keeps its own copy after the first boot.

**Custom domain.** After adding one to Open WebUI, set `WEBUI_URL` to `https://your-domain`.

**Single instance.** Open WebUI runs as one replica. Running several replicas needs Redis for websockets (`REDIS_URL`, `WEBSOCKET_MANAGER=redis`) and shared storage, which this template leaves out to stay lean.

**Resources.** On Railway the stack idled at about 1.8 GB of RAM: Open WebUI about 890 MB (it keeps the local embedding model for document search loaded), LiteLLM about 820 MB and Postgres about 60 MB. Both apps need more than the Trial plan's 512 MB per service, so use the Hobby plan or above.

## Why Deploy Open WebUI + LiteLLM on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying Open WebUI + LiteLLM on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
