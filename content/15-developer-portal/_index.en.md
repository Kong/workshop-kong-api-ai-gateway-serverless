---
title : "Konnect Developer Portal"
weight : 150
---


# Module 06 Labs: Developer Portal & API Products

[Back to Module 06 README](./README.md)

---

## Interactive Activities

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

[Back to Module 06 README](./README.md)

---

**References:**  
- [Konnect Dev Portal Overview](https://developer.konghq.com/dev-portal/#dev-portal)  
- [Publish your API to Dev Portal](https://developer.konghq.com/dev-portal/apis/#publish-your-api-to-dev-portal)  
- [API Products and Classic Dev Portal](https://developer.konghq.com/api-products/)  
- [Developer Self-Service and App Registration](https://developer.konghq.com/dev-portal/self-service/)  

*All steps above are achievable via the Konnect UI as described in the official documentation.*
