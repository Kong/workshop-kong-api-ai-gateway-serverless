---
title : "AI Rate Limiting Advanced"
weight : 158
---

With the existing API Key policy, we can control the incoming requests. However, the policies implemented by the other plugins are the same regardless the consumer.

In this section, we are going to define specific Rate Limiting policies for each Consumer represented by its API Key.

### Kong Consumer Policies

It's important then to be able to define specific policies for each one of these consumers. For example, it would be great to define Rate Limiting policies for different consumers like this:

* consumer1:
    * apikey = 123456
    * rate limiting policy = 500 tokens per minute
* consumer2:
    * apikey = 987654
    * rate limiting policy = 10000 tokens per minute

Doing that, the Data Plane is capable to not just protect the Route but to identify the consumer based on the key injected to enforce specific policies to the consumer. Keep in mind that a Consumer might have other plugins also enabled such as [TCP Log](https://docs.konghq.com/hub/kong-inc/tcp-log/), etc.


#### New Consumer and AI Rate Limiting Advanced plugin Policies

Then, create the second ``consumer2``, just like you did with the first one, with the ``987654`` key. Both Kong Consumers have the **AI Rate Limiting Advanced** plugin enabled with specific configurations.


```
cat > ai-key-auth-rate-limiting-advanced.yaml << 'EOF'
_format_version: "3.0"
_konnect:
  control_plane_name: serverless-default
_info:
  select_tags:
  - llm
services:
- name: service1
  host: localhost
  port: 32000
  routes:
  - name: openai-route
    paths:
    - /openai-route
    plugins:
    - name: ai-proxy
      instance_name: ai-proxy-openai
      config:
        route_type: llm/v1/chat
        auth:
          header_name: Authorization
          header_value: Bearer ${{ env "DECK_OPENAI_API_KEY" }}
        model:
          provider: openai
          name: gpt-4.1
          options:
            temperature: 1.0
    - name: key-auth
      instance_name: key-auth1
      enabled: true
consumers:
- keyauth_credentials:
  - key: "123456"
  username: user1
  plugins:
  - name: ai-rate-limiting-advanced
    instance_name: ai-rate-limiting-advanced-consumer1
    config:
      llm_providers:
      - name: openai
        window_size:
        - 60
        limit:
        - 500
- keyauth_credentials:
  - key: "987654"
  username: user2  
  plugins:
  - name: ai-rate-limiting-advanced
    instance_name: ai-rate-limiting-advanced-consumer2
    config:
      llm_providers:
      - name: openai
        window_size:
        - 60
        limit:
        - 10000
EOF
```


Apply the declaration with decK:
```
deck gateway reset --konnect-control-plane-name serverless-default --konnect-token $PAT -f
deck gateway sync --konnect-token $PAT ai-key-auth-rate-limiting-advanced.yaml
```


#### Use both Kong Consumers

If you will, you can inject both keys to your requests.

```
curl -i -X POST \
  --url $DATA_PLANE_URL/openai-route \
  --header 'Content-Type: application/json' \
  --header 'apikey: 123456' \
  --data '{
   "messages": [
     {
       "role": "user",
       "content": "Who is Jimi Hendrix?"
     }
   ]
  }'
```

* Expected output

```
HTTP/2 200 
content-type: application/json
x-kong-request-id: e3526809fc3549492e2463168ecae109
x-ai-ratelimit-limit-minute-openai: 500
x-ai-ratelimit-remaining-minute-openai: 500
x-ratelimit-remaining-tokens: 29993
x-ratelimit-reset-requests: 120ms
x-ratelimit-reset-tokens: 14ms
x-request-id: req_1713c9930dba475a83c14e825e8c4c03
x-openai-proxy-wasm: v0.1
strict-transport-security: max-age=31536000; includeSubDomains; preload
server: cloudflare
x-content-type-options: nosniff
access-control-expose-headers: X-Request-ID
alt-svc: h3=":443"; ma=86400
openai-organization: user-4qzstwunaw6d1dhwnga5bc5q
date: Wed, 24 Sep 2025 20:17:41 GMT
openai-project: proj_r4KYFyenuGWthS5te4zaurNN
cf-ray: 9844f7583ca8447a-EWR
openai-version: 2020-10-01
cf-cache-status: DYNAMIC
openai-processing-ms: 4861
x-ratelimit-limit-requests: 500
x-ratelimit-limit-tokens: 30000
x-envoy-upstream-service-time: 5001
x-ratelimit-remaining-requests: 499
x-kong-llm-model: openai/gpt-4.1
content-length: 2087
x-kong-upstream-latency: 5155
x-kong-proxy-latency: 1
via: 1.1 kong/3.11.0.0-enterprise-edition

{
  "id": "chatcmpl-CJQDgFMHEtVk3h2VotEk31bNLi0am",
  "object": "chat.completion",
  "created": 1758745056,
  "model": "gpt-4.1-2025-04-14",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "**Jimi Hendrix** (born Johnny Allen Hendrix, later renamed James Marshall Hendrix; November 27, 1942 – September 18, 1970) was an American guitarist, singer, and songwriter who is widely regarded as one of the most influential electric guitarists in the history of popular music. \n\nHendrix gained fame in the mid-1960s with his band, **The Jimi Hendrix Experience**, and is celebrated for his innovative and experimental use of the electric guitar, pioneering the use of distortion, overdrive, and feedback. Songs like **\"Purple Haze,\" \"Hey Joe,\" \"The Wind Cries Mary,\" and \"All Along the Watchtower\"** are among his most famous recordings.\n\nHis stage presence, technical ability, and creative use of guitar effects revolutionized rock music. Hendrix's performance at the 1969 Woodstock Festival—particularly his iconic rendition of \"The Star-Spangled Banner\"—has become legendary.\n\nHe released only three studio albums during his lifetime:  \n- **Are You Experienced** (1967)  \n- **Axis: Bold as Love** (1967)  \n- **Electric Ladyland** (1968)  \n\nDespite his brief career—he died at the age of 27—Hendrix's music and style have had a lasting impact, inspiring generations of musicians. He is frequently ranked among the greatest guitarists of all time.",
        "refusal": null,
        "annotations": []
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 13,
    "completion_tokens": 284,
    "total_tokens": 297,
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
  "system_fingerprint": "fp_3502f4eb73"
}
```

or

```
curl -i -X POST \
  --url $DATA_PLANE_URL/openai-route \
  --header 'Content-Type: application/json' \
  --header 'apikey: 987654' \
  --data '{
   "messages": [
     {
       "role": "user",
       "content": "Who is Jimi Hendrix?"
     }
   ]
  }'
```

* Expected output

```
HTTP/2 200 
content-type: application/json
x-kong-request-id: 52a6a066cc4e210a67ab850612ced730
x-ai-ratelimit-limit-minute-openai: 10000
x-ai-ratelimit-remaining-minute-openai: 10000
x-ratelimit-remaining-tokens: 29993
x-ratelimit-reset-requests: 120ms
x-ratelimit-reset-tokens: 14ms
x-request-id: req_99bf604880844d7c8ca5c47ffd8210ea
x-openai-proxy-wasm: v0.1
strict-transport-security: max-age=31536000; includeSubDomains; preload
server: cloudflare
x-content-type-options: nosniff
access-control-expose-headers: X-Request-ID
alt-svc: h3=":443"; ma=86400
openai-organization: user-4qzstwunaw6d1dhwnga5bc5q
date: Wed, 24 Sep 2025 20:18:24 GMT
openai-project: proj_r4KYFyenuGWthS5te4zaurNN
cf-ray: 9844f860ff07447a-EWR
openai-version: 2020-10-01
cf-cache-status: DYNAMIC
openai-processing-ms: 5501
x-ratelimit-limit-requests: 500
x-ratelimit-limit-tokens: 30000
x-envoy-upstream-service-time: 5532
x-ratelimit-remaining-requests: 499
x-kong-llm-model: openai/gpt-4.1
content-length: 2003
x-kong-upstream-latency: 5641
x-kong-proxy-latency: 1
via: 1.1 kong/3.11.0.0-enterprise-edition

{
  "id": "chatcmpl-CJQEMCDo5VIyVDRMI9Q7B951HlWxt",
  "object": "chat.completion",
  "created": 1758745098,
  "model": "gpt-4.1-2025-04-14",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "**Jimi Hendrix** (full name: **James Marshall Hendrix**; born November 27, 1942 – died September 18, 1970) was an American guitarist, singer, and songwriter. He is widely regarded as one of the most influential electric guitarists in the history of popular music and one of the most celebrated musicians of the 20th century.\n\nHendrix combined blues, rock, soul, and psychedelia in his innovative guitar style, known for his creative use of feedback, distortion, and other effects. He rose to fame in the 1960s with his band **The Jimi Hendrix Experience**, releasing iconic albums such as **\"Are You Experienced\"** (1967), **\"Axis: Bold as Love\"** (1967), and **\"Electric Ladyland\"** (1968).\n\nSome of his most famous songs include:\n- \"Purple Haze\"\n- \"All Along the Watchtower\" (a Bob Dylan cover)\n- \"Foxy Lady\"\n- \"Voodoo Child (Slight Return)\"\n- \"The Wind Cries Mary\"\n- \"Hey Joe\"\n\nHendrix's legendary performances, including his rendition of \"The Star-Spangled Banner\" at **Woodstock** in 1969, helped solidify his status as a rock icon. Despite his short career—he died at age 27—Hendrix's influence on rock, blues, and popular music continues to be profound.",
        "refusal": null,
        "annotations": []
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 13,
    "completion_tokens": 296,
    "total_tokens": 309,
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
  "system_fingerprint": "fp_3502f4eb73"
}
```


Again, test the rate-limiting policy by executing the following command multiple times and observe the rate-limit headers in the response, specially, ``X-AI-RateLimit-Limit-minute-openai`` and ``X-AI-RateLimit-Remaining-minute-openai``:



Now, let's consume it with the Consumer1's API Key. As you can see the Data Plane is processing the Rate Limiting processes independently.

```
curl -i -X POST \
  --url $DATA_PLANE_URL/openai-route \
  --header 'Content-Type: application/json' \
  --header 'apikey: 123456' \
  --data '{
   "messages": [
     {
       "role": "user",
       "content": "Who is Jimi Hendrix?"
     }
   ]
  }'
```


```
HTTP/2 200 
content-type: application/json
x-kong-request-id: 90665c528c6845101ac2eb3b85937540
x-ai-ratelimit-limit-minute-openai: 500
x-ai-ratelimit-remaining-minute-openai: 478
x-ratelimit-remaining-tokens: 29993
x-ratelimit-reset-requests: 120ms
x-ratelimit-reset-tokens: 14ms
x-request-id: req_2333577fbb0a4041aa6535ceba2e6102
x-openai-proxy-wasm: v0.1
strict-transport-security: max-age=31536000; includeSubDomains; preload
server: cloudflare
x-content-type-options: nosniff
access-control-expose-headers: X-Request-ID
alt-svc: h3=":443"; ma=86400
openai-organization: user-4qzstwunaw6d1dhwnga5bc5q
date: Wed, 24 Sep 2025 20:19:04 GMT
openai-project: proj_r4KYFyenuGWthS5te4zaurNN
cf-ray: 9844f94a4821447a-EWR
openai-version: 2020-10-01
cf-cache-status: DYNAMIC
openai-processing-ms: 8492
x-ratelimit-limit-requests: 500
x-ratelimit-limit-tokens: 30000
x-envoy-upstream-service-time: 8543
x-ratelimit-remaining-requests: 499
x-kong-llm-model: openai/gpt-4.1
content-length: 2309
x-kong-upstream-latency: 8627
x-kong-proxy-latency: 1
via: 1.1 kong/3.11.0.0-enterprise-edition
```


If we keep sending requests using the first API Key, eventually, as expected, we'll get an error code:

```
curl -i -X POST \
  --url $DATA_PLANE_URL/openai-route \
  --header 'Content-Type: application/json' \
  --header 'apikey: 123456' \
  --data '{
   "messages": [
     {
       "role": "user",
       "content": "Who is Jimi Hendrix?"
     }
   ]
  }'
```

```
HTTP/2 429 
date: Wed, 24 Sep 2025 20:19:45 GMT
content-type: application/json; charset=utf-8
x-kong-request-id: 16d3d8047c19485e57837e36e9f2191f
x-ai-ratelimit-reset-minute-openai: 29
x-ai-ratelimit-retry-after: 29
x-ai-ratelimit-reset: 29
x-ai-ratelimit-limit-minute-openai: 500
x-ai-ratelimit-remaining-minute-openai: 0
x-ai-ratelimit-retry-after-minute-openai: 29
content-length: 66
x-kong-response-latency: 1
server: kong/3.11.0.0-enterprise-edition

{"message":"AI token rate limit exceeded for provider(s): openai"}
```

However, the second API Key is still allowed to consume the Kong Route:


```
curl -i -X POST \
  --url $DATA_PLANE_URL/openai-route \
  --header 'Content-Type: application/json' \
  --header 'apikey: 987654' \
  --data '{
   "messages": [
     {
       "role": "user",
       "content": "Who is Jimi Hendrix?"
     }
   ]
  }'
```

```
HTTP/2 200 
content-type: application/json
x-kong-request-id: e6ee574fbc371d4c54b745507ba08b12
x-ai-ratelimit-limit-minute-openai: 10000
x-ai-ratelimit-remaining-minute-openai: 10000
x-ratelimit-remaining-tokens: 29993
x-ratelimit-reset-requests: 120ms
x-ratelimit-reset-tokens: 14ms
x-request-id: req_ee7712a38e284ab1a06ddf85f37742da
x-openai-proxy-wasm: v0.1
strict-transport-security: max-age=31536000; includeSubDomains; preload
server: cloudflare
x-content-type-options: nosniff
access-control-expose-headers: X-Request-ID
alt-svc: h3=":443"; ma=86400
openai-organization: user-4qzstwunaw6d1dhwnga5bc5q
date: Wed, 24 Sep 2025 20:20:14 GMT
openai-project: proj_r4KYFyenuGWthS5te4zaurNN
cf-ray: 9844fb1a48a2eef5-EWR
openai-version: 2020-10-01
cf-cache-status: DYNAMIC
openai-processing-ms: 4642
x-ratelimit-limit-requests: 500
x-ratelimit-limit-tokens: 30000
x-envoy-upstream-service-time: 4669
x-ratelimit-remaining-requests: 499
x-kong-llm-model: openai/gpt-4.1
content-length: 1870
x-kong-upstream-latency: 4784
x-kong-proxy-latency: 2
via: 1.1 kong/3.11.0.0-enterprise-edition

{
  "id": "chatcmpl-CJQGAUQK485uaDMZvu5fqtFWGG6KL",
  "object": "chat.completion",
  "created": 1758745210,
  "model": "gpt-4.1-2025-04-14",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "**Jimi Hendrix** (born **James Marshall Hendrix** on November 27, 1942 – died September 18, 1970) was an American guitarist, singer, and songwriter. Widely regarded as **one of the greatest and most influential electric guitarists in the history of popular music**, Hendrix revolutionized the way the instrument was played, using innovative techniques such as feedback, distortion, and controlled noise.\n\nHe first gained fame in the UK after forming the **Jimi Hendrix Experience** in 1966, releasing hit songs like “Hey Joe,” “Purple Haze,” and “The Wind Cries Mary.” His groundbreaking performances at the **Monterey Pop Festival** in 1967 and **Woodstock** in 1969 are legendary, with his rendition of “The Star-Spangled Banner” at Woodstock being especially iconic.\n\nHendrix’s music blended rock, blues, psychedelia, and funk, influencing countless musicians. His career was tragically short; he died at age 27. Today, Jimi Hendrix is remembered as a cultural icon and is frequently cited on lists of the greatest guitarists of all time.",
        "refusal": null,
        "annotations": []
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 13,
    "completion_tokens": 232,
    "total_tokens": 245,
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
  "system_fingerprint": "fp_3502f4eb73"
}
```

Kong-gratulations! have now reached the end of this module by authenticating the API requests with a key and associating different consumers with policy plans. You can now click **Next** to proceed with the next module.

