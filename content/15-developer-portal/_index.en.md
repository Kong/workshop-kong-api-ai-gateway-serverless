---
title : "Konnect Developer Portal"
weight : 150
---

The Konnect Developer Portal is a customizable website for developers to locate, access, and consume API services. It enables developers to browse and search API documentation, try API operations, and manage their own credentials. The Portal supports both internal and external APIs through flexible deployment options.

Dev Portal APIs allow you to publish APIs using OpenAPI or AsyncAPI specifications and Markdown documentation. You can provide highly customizable content at both the site and API level to offer additional context to developers.

Konnect Developer Portal provides an extensive list of benefits to **Developers** as well as **Organizations**:

**Developers**
* Accelerates Onboarding:
  * Self-service and instant access to documentation of APIs and testing tools
* Empowers Innovation through discovery of APIs and usage instructions
  * Build applications faster
  * Foster culture of innovation
* Improve Developer Experience
  * Discovery of APIs
  * In browser testing and troubleshooting tools


**Organizations**
* Drives API Adoption
  * Public or Partner facing portal markets APIs, fueling innovation and new revenue streams
* Reduces Support Overhead
  * Comprehensive and searchable documentation and self-service tools shifts the burden of support
* Ensures Governance and Security
  * Portal acts as single source of truth
  * Ensures developers are using the correct, approved versions of APIs and adhering to policies
* Enhances Collaboration



## Basic Developer Portal implementation

In this section we are going to cover the basic steps to get a Developer Portal deployed

### Kong Service and Route

1. Create a Kong Service based on ``http://httpbin.konghq.com``
2. Create a Kong Route with path ``/``
3. Globally enable the CORS plugin to your Control Plane. That's needed to solve the relationship between the Dev Portal and the Upstream Service.


### Dev Portal creation

1. Choose the **Dev Portal** menu option and click **+ Create a new portal**.
2. Create a **Private Portal** named ``portal1``. In the **New Portal** page accept the default values and click **Save**
![portal1](/static/images/portal1.png)
3. Click **Go to overview**. You should see the **Overview** page of your portal
![portal1_overview](/static/images/portal1_overview.png)
4. Check the Portal configuration:
* **User authentication**: ``Konnect built-in`` - That defines the default mechanism the Dev Portal uses for the user authentication. Besides the **build-in** option, it can be configured as **OIDC** or **SAML**.
* **Default authentication strategy**: ``key-auth`` - That defines the mechanism to control the API consumption inside the Dev Portal.

### Test your Dev Portal

1. Sign Up

If you click on the Dev Portal URL, you will play the Developer role and see your new portal home page. Since there's no developer created, click **Sing up** and register to the Portal. Type your name and use a real email since the Dev Portal will send a confirmation request to it.

![portal1_signup](/static/images/portal1_signup.png)

Check your email and click the **Confirm your email address**. Still playing the developer role, you should get redirected to the Dev Portal to define your password.

After creating the password, if you try to login, you'll receive an error message saying "Your account is disabled or pending approval". That's because, by default, the Dev Portal was created with the **Auto approved developers** option disabled, meaning the administrator has to manually approve the new developers registrations.

2. Approve the Developer registration

Getting back to the Dev Portal Administrator role, return to the **Dev Portal** menu option and choose **Access and approvals**. You can approve the new developer registration in the page:

![portal1_developer_approval](/static/images/portal1_developer_approval.png)

3. Login to the Dev Portal

Playing the Developer role again, try to login to the Dev Portal one more time. You should get redirected to the Dev Portal home page. Click the **API** tab. You are supposed to get an empty page since we don't have any API published.

![portal1_no_apis](/static/images/portal1_no_apis.png)




### API creation

1. Prepare your OpenAPI specification

Download the [httpbin_spec.yaml](/code/httpbin_spec.yaml) OpenAPI specification. From the Konnect Dev Portal perspective, the spec has two main configurations:
* The ``servers`` section. Make sure the ``url`` parameters has your Proxy URL:
```  
  - url: <YOUR_PROXY_URL>
```

* Note the spec has added specific DevPortal elements in the ``security`` & ``securitySchemes`` sections. That means the DevPortal will use the Key Auth plugin to control the API Consumption inside the Portal.
```
#################################
# Kong DevPortal Security mechanism
#################################
security:
  - ApiKeyAuth: []
#################################
```

```
components:
  securitySchemes:
#################################
# Kong Gateway Key-Auth
#################################
    ApiKeyAuth:
      type: apiKey
      in: header
      name: apikey
#################################
```


2. Create your API

Choose the **APIs** menu option inside **Dev Portal** and click **+ New API**. Upload your ``httpbin_orig.yaml`` and click **Create**. You should see your ``httpbin`` API page:

![httpbin_api](/static/images/httpbin_api.png)



3. Test your API

Click the **API Specification** tab. Click **try it!** in the **Returns Origin IP**

![tryit](/static/images/tryit.png)


4. Add a documentation

Click the **Documentation** tab. Create a new and empty document page with both name and slug as ``doc1``. Click **edit** and type some documentation. Click **save** and switch the **Published** toggle on.


![api_documentation](/static/images/api_documentation.png)






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
