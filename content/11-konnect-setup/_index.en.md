---
title : "Konnect Setup"
weight : 110
---

This chapter will walk you through

* Konnect Control Plane and Data Plane.
* Access Kong Data Plane.

Here's a Reference Architecture that will be implemented in this workshop:

![kong](/static/images/ref_arch.png)

* Kong Konnect Control Plane: responsible for managing your APIs
* Kong Konnect Data Plane: connected to the Control Plane, it is responsible for processing all the incoming requests sent by the consumers.
* Kong provides a plugin framework, where each one of them is responsible for a specific functionality. As a can see, there are two main collections of plugins:
    *  On the left, the historic and regular API Gateway plugins, implementing all sort of policies including, for example, OIDC based Authentication processes with Keycloak, Amazon Cognito and Okta or Observability with Prometheus/Grafana and Dynatrace.
    * On the right, another plugin collection for AI-based use cases. For example, the AI Rate Limiting plugin implements policies like this based on the number of tokens consumed be the requests. Or, as another example is the AI Semantic Cache plugin, which caches data based on the semantics related to the responses coming from the LLM models.
* Kong AI Gateway supports, out of the box, a variety of infrastructures, including not just OpenAI, but also Amazon Bedrock, Google Gemini, Mistral, Anthropic, etc. In order to deal with embeddings, the Gateway also supports also vector databases.
* Kong Gateway protects not just the LLM Models but also the upstream services, including your application micros surfaces or services.



### Serverless Proxy URL

Log in to the Kong Konnect UI. Click "API Gateway" and choose the ``serverless-default`` Control Plane. You should see the following landing page:

![Serverless Control Page](/static/images/serverless_control_plane.png)

Copy the value of your Control Plane ``Proxy URL`` and keep it handy. That's the URL you Data Plane is located.

### Send a request to your Serverless Data Plane

Save your URL in an enviroment variable:

```
export DATA_PLANE_URL=<YOUR_DATA_PLANE_URL>
```

You can use ``curl`` to send the first request to the Data Plane

```
curl $DATA_PLANE_URL
```

Expected result
```
{
  "message":"no Route matched with those values",
  "request_id":"84fac2649eb6ae01f4d920115a4df70d"
}
```






### Konnect Control Plane and Kong Objects
There are multiple ways to create new Kong Objects in your Control Plane:
* Konnect User Interface.
* [RESTful Admin API](https://developer.konghq.com/api/), a fundamental mechanism for administration purposes.
* [Kong Gateway Operator (KGO)](https://developer.konghq.com/gateway-operator/) and Kubernetes CRDs

To get an easier and faster deployment, this workshop uses Konnect User Interface.

This tutorial is intended to be used for labs and PoC only. There are many aspects and processes, typically implemented in production sites, not described here. For example: Digital Certificate issuing, Cluster monitoring, etc. For a production ready deployment, refer Kong on Terraform Constructs, available [here](https://developer.konghq.com/terraform/)

You can now click **Next** to begin the module.