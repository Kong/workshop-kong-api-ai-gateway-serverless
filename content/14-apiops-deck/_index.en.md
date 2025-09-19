---
title : "APIOps and decK"
weight : 140
---

### Concept

APIOps applies DevOps principles (automation, version control, CI/CD) to API lifecycle management.  
With Kong Konnect and **decK**, APIs can be treated as code — versioned, tested, and deployed through pipelines.

### Exercise

1. **Clone/Open your lab repo** where specs and configs are stored.

2. **Review the Bash script** below. It demonstrates:
   - Converting an OpenAPI spec into Kong configuration (`openapi2kong`)
   - Tagging routes for traceability
   - Merging platform defaults, plugins, and upstreams
   - Syncing the configuration with **Konnect**

    ```bash
    #!/bin/sh
    set -e

    INPUT_SPEC="./spec/kong-air-apis.yaml"
    OUTPUT_SPEC="./decks/kong.yaml"

    echo "Step 1: Convert OpenAPI → Kong decK format"
    deck file openapi2kong --spec "$INPUT_SPEC" -o "$OUTPUT_SPEC"

    echo "Step 2: Add tags"
    deck file add-tags kong-konnect-bootcamp -s "$OUTPUT_SPEC" -o "$OUTPUT_SPEC"

    echo "Step 3: Merge platform configs"
    deck file merge "$OUTPUT_SPEC" ./platform/*.yaml -o "$OUTPUT_SPEC"

    echo "Step 4: Sync with Konnect"
    deck gateway sync "$OUTPUT_SPEC" \
    --konnect-addr "https://eu.api.konghq.com" \
    --konnect-control-plane-name "bootcamp-2025" \
    --konnect-token "$DECK_KONNECT_TOKEN" \
    --select-tag kong-konnect-bootcamp
    ```

    - Refer to `./scripts/apiops.sh` for the full Bash script.

3. Run the script locally to simulate a pipeline execution.
   - Replace spec/config paths with your files.
   - Ensure decK is installed and your Konnect token is set.

4. (Optional) Integrate this script into a CI tool of your choice (e.g., Jenkins, GitHub Actions, GitLab CI).
   - Store Konnect tokens as secrets.
   - Trigger the script on every code push or pull request.

## Key Takeaways

- **Treat APIs as code**: store OpenAPI + Kong configs in version control.
- **Automate API deployment** with decK for repeatability and reliability.
- **CI/CD tools** can run the same Bash script to enable APIOps in real-world projects.

## Next Steps

- Extend the script with tests (linting, validation, diff before sync).
- Explore GitHub Actions or Jenkins examples for pipeline automation.
- Move towards full GitOps by integrating declarative API configs with source control workflows.

---
