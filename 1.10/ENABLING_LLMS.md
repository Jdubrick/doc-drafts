# Enabling LLMs for Lightspeed

For both the RHDH Operator and RHDH Helm installs, you will have have a Kubernetes Secret will the following keys:

```yaml
ENABLE_VLLM: ""
ENABLE_VERTEX_AI: ""
ENABLE_OPENAI: ""
ENABLE_OLLAMA: ""
ENABLE_VALIDATION: ""
VLLM_URL: ""
VLLM_API_KEY: ""
VLLM_MAX_TOKENS: ""
VLLM_TLS_VERIFY: ""
OPENAI_API_KEY: ""
VERTEX_AI_PROJECT: ""
VERTEX_AI_LOCATION: ""
GOOGLE_APPLICATION_CREDENTIALS: ""
OLLAMA_URL: ""
VALIDATION_PROVIDER: ""
VALIDATION_MODEL_NAME: ""
LLAMA_STACK_LOGGING: ""
```

The following sections will break down the values per-provider. As a general guideline however, if the `ENABLE_*` value is unset, (`""`), it is **disabled.**

## vLLM
To enable the use of LLMs via a vLLM provider (OpenAI API compatible) you will need to set the following in your Secret:

```yaml
ENABLE_VLLM=true
VLLM_URL=<your-url>/v1
VLLM_API_KEY<your-key>
```

You can also optionally set:
```yaml
VLLM_MAX_TOKENS=<defaults-to-4096>
VLLM_TLS_VERIFY=<defaults-to-true>
```

## OpenAI
To enable the use of LLMs from OpenAI:

```yaml
ENABLE_OPENAI=true
OPENAI_API_KEY=<your-key>
```

## Ollama
To enable the use of LLMs from Ollama:

```yaml
ENABLE_OLLAMA=true
OLLAMA_URL=<your-url>
```

## Vertex AI
To enable the use of LLMs from Vertex AI (Gemini)

```yaml
ENABLE_VERTEX_AI=true
VERTEX_AI_PROJECT=<your-project>
VERTEX_AI_LOCATION=<location>
GOOGLE_APPLICATION_CREDENTIALS=<path-to-creds-json-mounted-to-sidecar>
```

**Note:** Vertex is the least tested and one of the more complex setups.

## Validation

Developer Lightspeed supports query validation, which restricts the chatbot to RHDH-related questions. When enabled, off-topic queries (e.g., asking about the weather) will be rejected while development-related questions are allowed.

```env
# Enable query validation
ENABLE_VALIDATION=true

# REQUIRED if validation is enabled: Must be one of your enabled providers
# Example: if ENABLE_OPENAI=true, then set VALIDATION_PROVIDER=openai
VALIDATION_PROVIDER=openai

# REQUIRED if validation is enabled: Must be an available model for the chosen provider
# Example: VALIDATION_MODEL_NAME=gpt-4o-mini
VALIDATION_MODEL_NAME=gpt-4o-mini
```

> [!NOTE]
> The validation provider must be one of your enabled inference providers, and the model must be available on that provider.