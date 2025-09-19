---
title : "Prerequisites"
weight : 100
---


## Knowledge

Attendees should have intermediate knowledge of **Kubernetes** and **Docker** for installations of Konnect Data Plane and other products like **Keycloak**, **Redis**, **Prometheus** etc.

Attendees should have basic knowledge of **GenAI models**, specially **LLMs** and **Embedding models** as well as providers like **OpenAI**, **Mistral**, **Anthropic**, **AWS**, **GCP**, **Azure**.


## Kong Academy

Complete two badges via Kong Academy: **Kong Gateway Foundations** and **Kong Gateway Operations**


## Kong Konnect Subscription
You will need a Kong Konnect Subscription to execute this workshop.


## Environment
Please, make sure you have [**Minikube 1.36.0**](https://minikube.sigs.k8s.io/) Kubernetes Cluster over [**Podman 5.6.0**](https://podman.io/) installed.

* [Podman installation](https://podman.io/docs/installation)
* [Minikube installation](https://minikube.sigs.k8s.io/docs/start)


## Command Line Utilities
In this workshop, we will use the following command line utilities

* [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
* [helm](https://helm.sh/docs/intro/install/)
* [k9s](https://k9scli.io/)
* [curl](https://curl.se/)
* [jq](https://jqlang.org/)
* [yq](https://github.com/mikefarah/yq)
* [jwt-cli](https://github.com/mike-engel/jwt-cli)
* [wget](https://www.gnu.org/software/wget/)


## Redis
[**Redis**](https://redis.io/docs/) is used in some use cases, including Rate Limiting, Caching, Semantic Routing and RAG.


## LLM
### Ollama
The Kong AI Gateway use cases consume and protect LLMs running on [**Ollama**](https://github.com/ollama)


### OpenAI
Some AI use cases also use OpenAI's Embeddings and LLMs. Please make sure you have an OpenAI API key.



## Recommended hardware not including Ollama

* CPU: 4-6 vCPUs
* Memory: 8-16GB
* Disk: 30–50GB

## Recommended hardware including Ollama

* CPU: 6-8 vCPUs
* Memory: 12-24GB
* Disk:	50-100GB
