---
title : "Round Robin"
weight : 2
---

Let's get started with a simple round-robin policy:

```
cat > ai-proxy-advanced.yaml << 'EOF'
_format_version: "3.0"
_info:
  select_tags:
  - llm
_konnect:
  control_plane_name: serverless-default
services:
- name: ai-proxy-advanced-service
  host: localhost
  port: 32000
  routes:
  - name: route1
    paths:
    - /route1
    plugins:
    - name: ai-proxy-advanced
      instance_name: ai-proxy-advanced1
      config:
        balancer:
          algorithm: round-robin
        targets:
        - model:
            provider: openai
            name: gpt-4.1
            options:
              temperature: 1.0
          route_type: "llm/v1/chat"
          auth:
            header_name: Authorization
            header_value: Bearer ${{ env "DECK_OPENAI_API_KEY" }}
        - model:
            provider: openai
            name: gpt-5
            options:
              temperature: 1.0
          route_type: "llm/v1/chat"
          auth:
            header_name: Authorization
            header_value: Bearer ${{ env "DECK_OPENAI_API_KEY" }}
EOF
```



Apply the declaration with decK:
```
deck gateway reset --konnect-control-plane-name serveless-default --konnect-token $PAT -f
deck gateway sync --konnect-token $PAT ai-proxy-advanced.yaml
```

Send the request few times. Note we are going to receive responses from both LLMs, in a round-robin way.

```
curl -s -X POST \
  --url $DATA_PLANE_URL/route1 \
  --header 'Content-Type: application/json' \
  --data '{
     "messages": [
       {
         "role": "user",
         "content": "Who is considered the greatest Polish writer?"
       }
     ]
   }' | jq
```


Here's the gpt-5 response:
```
{
  "id": "chatcmpl-CJQJOSWpos9jzGMiYiXpeXcrMyGEN",
  "object": "chat.completion",
  "created": 1758745410,
  "model": "gpt-5-2025-08-07",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Adam Mickiewicz is most often considered the greatest Polish writer (and Poland’s national bard). Depending on criteria, others frequently cited include Henryk Sienkiewicz, Stanisław Lem, Czesław Miłosz, and Wisława Szymborska.",
        "refusal": null,
        "annotations": []
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 14,
    "completion_tokens": 385,
    "total_tokens": 399,
    "prompt_tokens_details": {
      "cached_tokens": 0,
      "audio_tokens": 0
    },
    "completion_tokens_details": {
      "reasoning_tokens": 320,
      "audio_tokens": 0,
      "accepted_prediction_tokens": 0,
      "rejected_prediction_tokens": 0
    }
  },
  "service_tier": "default",
  "system_fingerprint": null
}
```

And here is gpt-4's:
```
{
  "id": "chatcmpl-CJQJYClEQQXwtNpnm6BiAuhLufLex",
  "object": "chat.completion",
  "created": 1758745420,
  "model": "gpt-4.1-2025-04-14",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "The title of **greatest Polish writer** is most often attributed to **Adam Mickiewicz** (1798–1855). Mickiewicz, a Romantic-era poet, is widely regarded as Poland’s national poet and a key figure in Polish literature. His epic poem **\"Pan Tadeusz\"** is considered the national epic of Poland, and his influence on Polish identity, language, and literature is profound.\n\nOther notable Polish writers who are sometimes mentioned in discussions of the greatest include:\n\n- **Henryk Sienkiewicz** (Nobel Prize in Literature 1905), known for the historical novel **\"Quo Vadis\"** and the \"Trilogy\" (“Trylogia”).\n- **Wisława Szymborska** (Nobel Prize in Literature 1996), celebrated for her poetry.\n- **Czesław Miłosz** (Nobel Prize in Literature 1980), a poet and essayist.\n- **Bruno Schulz**, known for his unique prose.\n- **Stanisław Lem**, a world-renowned science fiction author.\n\nHowever, in terms of cultural impact and universal recognition as the quintessential Polish writer, **Adam Mickiewicz** is most frequently given that distinction.",
        "refusal": null,
        "annotations": []
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 15,
    "completion_tokens": 252,
    "total_tokens": 267,
    "prompt_tokens_details": {
      "cached_tokens": 0,
      "audio_tokens": 0
    },
    "completion_tokens_details": {
      "reasoning_tokens": 0,
      "audio_tokens": 0,
      "accepted_prediction_tokens": 0,
      "rejected_prediction_tokens": 0
    }
  },
  "service_tier": "default",
  "system_fingerprint": "fp_62bb2e3c55"
}
```