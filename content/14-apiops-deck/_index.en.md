---
title : "APIOps and decK"
weight : 140
---

## Concept

APIOps applies DevOps principles (automation, version control, CI/CD) to API lifecycle management.  
With Kong Konnect and **decK**, APIs can be treated as code — versioned, tested, and deployed through pipelines.

### Download the OpenAPI specification

Download the [kong-air-apis.yaml](/code/kong-air-apis.yaml) OpenAPI spec. The spec defines routes to consume APIs exposed by the [**KongAir API Server**](https://api.kong-air.com/routes)



### Convert the OpenAPI spec into Kong configuration

```
deck file openapi2kong --spec ./kong-air-apis.yaml -o kong.yaml
```



### Add tags for traceability

```
deck file add-tags kong-konnect-workshop -s kong.yaml -o kong.yaml
```



### Merge plugins configuration

Create a plugin configuration file

```
cat > plugins.yaml << 'EOF'
_format_version: "3.0"
_konnect:
   control_plane_name: serverless-default
_info:
   select_tags:
   - kong-konnect-workshop
plugins:
- name: proxy-cache
  enabled: true
  config:
   strategy: memory
   cache_ttl: 30
EOF
```


Merge the files

```
deck file merge ./kong.yaml ./plugins.yaml -o kong.yaml
```



### Sync with Konnect

```
deck gateway sync kong.yaml \
    --konnect-addr "https://us.api.konghq.com" \
    --konnect-control-plane-name serverless-default \
    --konnect-token $PAT \
    --select-tag kong-konnect-workshop
```

Expected output:

```
creating service routes-service
creating route routes-service_get-route
creating route routes-service_health-check
creating route routes-service_get-routes
creating plugin proxy-cache (global)
Summary:
  Created: 5
  Updated: 0
  Deleted: 0
```


### Consume a Kong Route

```
curl -s $DATA_PLANE_URL/routes | jq
```



## CI tool integration

As a best practice, for a CI tool integration, these steps should be included in a script and integrated into a CI tool of your choice (e.g., Jenkins, GitHub Actions, GitLab CI).

The script should be triggered the script on every code push or pull request.

Lastly, you should store your Konnect token as secrets.
   

### Key Takeaways

- Treat APIs as code: store OpenAPI + Kong configs in version control.
- Automate API deployment with decK for repeatability and reliability.
- CI/CD tools can run Bash scripts to enable APIOps in real-world projects.

### Next Steps

- Extend the script with tests (linting, validation, diff before sync).
- Explore GitHub Actions or Jenkins examples for pipeline automation.
- Move towards full GitOps by integrating declarative API configs with source control workflows.
