---
title: "Kong Konnect Architectural Overview"
weight: 105
---

The Kong Konnect platform provides a cloud control plane (CP), which manages all service configurations. It propagates those configurations to all Runtime control planes, which use in-memory storage. These nodes can be installed anywhere, on-premise or in AWS.

![Konnect Architecture](/static/images/konnect-introduction.png)

For today workshop, we will be focusing on Kong Gateway.
Kong Gateway data plane listen for traffic on the proxy port 443 by default. The data plane evaluates incoming client API requests and routes them to the appropriate backend APIs. While routing requests and providing responses, policies can be applied with plugins as necessary.

# Konnect modules

Kong Konnect Enterprise features are described in this section, including modules and plugins that extend and enhance the functionality of the Kong Konnect platform.

## Control Plane (Gateway Manager)
Control Plane empowers your teams to securely collaborate and manage their own set of runtimes and services without the risk of impacting other teams and projects. Control Plane instantly provisions hosted Kong Gateway control planes and supports securely attaching Kong Gateway data planes from your cloud or hybrid environments.

Through the Control Plane, increase the security of your APIs with out-of-the-box enterprise and community plugins, including OpenID Connect, Open Policy Agent, Mutual TLS, and more.

![gateway_manager](/static/images/gateway_cp.png)

## AI Manager
Manage all of your LLMs in a single dashboard providing a unified control plane to create, manage, and monitor LLMs using the Konnect platform. With AI Manager you can assign Gateway Services and define how traffic is distributed across models, enable streaming responses and manage authentication through the AI Gateway, monitor request and token volumes, track error rates, and measure average latency with historical comparisons, etc.

## Dev Portal
Streamline developer onboarding with the Dev Portal, which offers a self-service developer experience to discover, register, and consume published services from your Service Hub catalog. This customizable experience can be used to match your own unique branding and highlights the documentation and interactive API specifications of your services. Enable application registration to automatically secure your APIs with a variety of authorization providers.

## Analytics
Use Analytics to gain deep insights into service, route, and application usage and health monitoring data. Keep your finger on the pulse of the health of your API products with custom reports and contextual dashboards. In addition, you can enhance the native monitoring and analytics capabilities with Kong Gateway plugins that enable streaming monitoring metrics to third-party analytics providers.

## Teams
To help secure and govern your environment, Konnect provides the ability to manage authorization with teams. You can use Konnect’s predefined teams for a standard set of roles, or create custom teams with any roles you choose. Invite users and add them to these teams to manage user access. You can also map groups from your existing identity provider into Konnect teams.

# Further Reading

* [Gateway Manager](https://developer.konghq.com/gateway-manager/)
* [AI Manager](https://developer.konghq.com/ai-manager/)
* [Dev Portal](https://developer.konghq.com/dev-portal/)
* [Analytics](https://developer.konghq.com/analytics/)

