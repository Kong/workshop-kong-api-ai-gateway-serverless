---
title : "Weight"
weight : 3
---

Now, let's redirect 80% of the request to OpenAI's gpt-4.1 with a weight based policy:

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
          weight: 80
        - model:
            provider: openai
            name: gpt-5
            options:
              temperature: 1.0
          route_type: "llm/v1/chat"
          auth:
            header_name: Authorization
            header_value: Bearer ${{ env "DECK_OPENAI_API_KEY" }}
          weight: 20
EOF
```


Apply the declaration with decK:
```
deck gateway reset --konnect-control-plane-name serverless-default --konnect-token $PAT -f
deck gateway sync --konnect-token $PAT ai-proxy-advanced.yaml
```
