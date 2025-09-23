---
title : "Konnect Developer Portal"
weight : 150
---



Kong Service and Route
1. Create a Kong Service based on http://httpbin.org
2. Create a Kong Route with path “/“



CORS
1. Globally enable the CORS plugin to your Control Plane



API
1. Create an API
Use httpbin-orig.yaml. Show the two sections: security & securitySchemes

2. Check Spec and send a “try it!” the “Returns Origin IP”

3. Add a documentation



Portal
1. Create a portal

2. Check
User authn: Konnect built-in
RBAC: disabled
Default authentication strategy: key-auth

3. Check Settings -> Security tab
User authN: on with build-in
RBAC: off
AuthN strategy: key-auth

4. Play the developer role. Click on the portal link and sign up

5. Approve the developer

6. Play the developer role. Sign in and show there’s no API available

7. Go to APIs -> Portals tab and publish the API to the Portal with Private visibility

8. Play the developer role. Reload the Portal and see the new API available 

9. You should be able to consume the API, e.g. “Returns Origin IP”.



RBAC

1. Go to APIs -> Gateway Service. Link the API to the Kong Gateway Service. Show the Konnect App Auth plugin is enabled to the Service.
You can link to a Konnect Gateway Service to allow developers to create applications and generate credentials or API keys. This is available to data planes running Kong Gateway 3.6 or later.

When you link a service with an API, Konnect automatically adds the Konnect Application Auth (KAA) plugin on that Service. The KAA plugin is responsible for applying authentication and authorization on the Service. The authentication strategy that you select for the API defines how clients authenticate. While you can’t directly modify the KAA plugin as it’s managed by Konnect, you can modify the plugin’s behavior by adding JSON to the advanced configuration of your application auth strategy.

2. Play the developer role. If you try to consume the API will get a 401

3. Portal -> Settings -> Security -> turn RBAC on to your Portal

4. Portal -> Access and approvals -> create a Team

5. Add the developer to the team

6. Inside your team -> APIs -> Add a new role with the existing API and “API Consumer” role

7. Play the developer role. If you try to consume the API you still get a 401



App

1. Play the developer role. Click “Use this API” and create an application (the Auth strategy is the default - api key auth)

2. Copy the Credential (API Key - 3cI5F8xFj7DAkeAEfFA5vpHnQJjByYmx)

3. Approve the Application

4. Play the developer role. Add your API Key in the Authentication box. You should be able to consume the API (e.g. Returns Origin IP)

5. Choose your app and navigate to the Credentials tab.




### 1. Create a Dev Portal
- Navigate to **Konnect** and go to **Dev Portal**.
- Click **Create Dev Portal** and fill in the required details (name, authentication settings, visibility, etc.).
- Save and note your Dev Portal URL.

### 2. Publish an API to the Dev Portal
- Go to **Dev Portal > APIs**.
- Click **New API** and provide the API name, version, and upload an OpenAPI spec or Markdown documentation.
- (Optional) Link the API to a Gateway Service to enable developer self-service and authentication.
- Publish the API to your Dev Portal and select an authentication strategy if required.  
  > ℹ️ You must have the `Product Publisher` role to publish APIs to the portal.

### 3. Register as a Developer and Create an Application
- Open your Dev Portal URL in a new browser window.
- Sign up as a new developer (or sign in if you already have an account).
- Once approved, go to **My Apps** and create a new application.
- Register your application for one or more APIs and generate credentials (API key or OIDC, depending on the authentication strategy).

### 4. Explore API Products (Classic Dev Portal v2)
- In **Konnect**, navigate to **API Products**.
- Create a new API Product, add a version, and link it to a Gateway Service.
- Add documentation (OpenAPI spec and/or Markdown).
- Publish the API Product to a classic Dev Portal (v2) by selecting the portal and confirming publication.

---

## Key Takeaways

- The Konnect Dev Portal enables you to publish, document, and manage APIs for internal, partner, or public consumption.
- APIs can be published to the Dev Portal with OpenAPI/AsyncAPI specs and Markdown documentation, and linked to Gateway Services for authentication and self-service.
- Developers can self-register, create applications, and generate credentials directly from the Dev Portal.
- API Products (v2) allow you to bundle multiple services and versions, manage documentation, and publish to classic Dev Portals for broader consumption.
- All of these actions can be performed via the Konnect UI, providing a user-friendly, centralized management experience for your API ecosystem.

---

## Next Steps

- Explore advanced Dev Portal customization options (branding, custom pages, and layouts).
- Review analytics for your APIs and API Products in Konnect.
- Learn about automating API and Dev Portal management using Konnect APIs or Terraform.
---


**References:**  
- [Konnect Dev Portal Overview](https://developer.konghq.com/dev-portal/#dev-portal)  
- [Publish your API to Dev Portal](https://developer.konghq.com/dev-portal/apis/#publish-your-api-to-dev-portal)  
- [API Products and Classic Dev Portal](https://developer.konghq.com/api-products/)  
- [Developer Self-Service and App Registration](https://developer.konghq.com/dev-portal/self-service/)  

*All steps above are achievable via the Konnect UI as described in the official documentation.*
