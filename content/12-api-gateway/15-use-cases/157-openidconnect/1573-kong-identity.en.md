---
title : "Kong Identity"
weight : 1573
---

### Kong Identity

Kong Identity enables you to use Konnect to generate, authenticate, and authorize API access. You can use Kong Identity to:

* Create authorization servers per region
* Issue and validate access tokens
* Integrate secure authentication into your Kong Gateway APIs


### OAuth Grants

The two next topics describe Authorization Code OAuth and Client Credentials grants implemented by Kong Konnect and [Kong Identity](https://developer.konghq.com/kong-identity/) as the Identity Provider. Let's start instantiating an Authentication Service in Kong Identity.


### Authentication Server in Kong Identity

Before you can configure any authentication plugin, you must first create an auth server in Kong Identity. The auth server name is unique per each organization and each Konnect region.

Create an auth server using the /v1/auth-servers endpoint:

```
curl -sX POST "https://us.api.konghq.com/v1/auth-servers" \
  -H "Authorization: Bearer $PAT"\
  -H "Content-Type: application/json" \
  --json '{
    "name": "AuthN_Server_1",
    "audience": "http://myhttpbin.dev",
    "description": "AuthN Server 1"
  }' | jq
```

You should get a response like this:

```
{
  "audience": "http://myhttpbin.dev",
  "created_at": "2025-09-19T20:59:43.302149Z",
  "description": "AuthN Server 1",
  "id": "6ddf6bee-6cc0-417f-89ad-8375eead428b",
  "issuer": "https://dhafxmmgz56rlw6b.us.identity.konghq.com/auth",
  "labels": {},
  "metadata_uri": "https://dhafxmmgz56rlw6b.us.identity.konghq.com/auth/.well-known/openid-configuration",
  "name": "AuthN_Server_1",
  "signing_algorithm": "RS256",
  "updated_at": "2025-09-19T20:59:43.302149Z"
}
```


##### Check your AuthN Server

```
curl -sX GET "https://us.api.konghq.com/v1/auth-servers" \
  -H "Authorization: Bearer $PAT" | jq
```

Get the AuthN Server Id:

```
export AUTHN_SERVER_ID=$(curl -sX GET "https://us.api.konghq.com/v1/auth-servers" -H "Authorization: Bearer $PAT" | jq -r '.data[0].id')
```

Get the Issuer URL:

```
export ISSUER_URL=$(curl -sX GET "https://us.api.konghq.com/v1/auth-servers" -H "Authorization: Bearer $PAT" | jq -r '.data[0].issuer')
```




##### Delete the AuthN Server
If you want to delete it run:

```
curl -sX DELETE "https://us.api.konghq.com/v1/auth-servers/$AUTHN_SERVER_ID" \
  -H "Authorization: Bearer $PAT" | jq
```


### Configure the auth server with scopes

Configure a scope in your auth server using the /v1/auth-servers/$AUTHN_SERVER_ID/scopes endpoint:

```
curl -sX POST "https://us.api.konghq.com/v1/auth-servers/$AUTHN_SERVER_ID/scopes" \
  -H "Authorization: Bearer $PAT"\
  -H "Content-Type: application/json" \
  --json '{
    "name": "scope1",
    "description": "scope1",
    "default": false,
    "include_in_metadata": false,
    "enabled": true
  }' | jq
```

Expected response

```
{
  "created_at": "2025-09-19T21:05:34.604098Z",
  "default": false,
  "description": "scope1",
  "enabled": true,
  "id": "897f7a59-b6e0-4dca-b0d7-b896c681d50b",
  "include_in_metadata": false,
  "name": "scope1",
  "updated_at": "2025-09-19T21:05:34.604098Z"
}
```

Export your scope ID:

```
export SCOPE_ID=$(curl -sX GET "https://us.api.konghq.com/v1/auth-servers/$AUTHN_SERVER_ID/scopes" -H "Authorization: Bearer $PAT" | jq -r '.data[0].id')
```



### Configure the auth server with custom claims

Configure a custom claim using the ```/v1/auth-servers/$AUTHN_SERVER_ID/claims``` endpoint:

```
curl -sX POST "https://us.api.konghq.com/v1/auth-servers/$AUTHN_SERVER_ID/claims" \
  -H "Authorization: Bearer $PAT" \
  -H "Content-Type: application/json" \
  --json '{
    "name": "claim1",
    "value": "claim1",
    "include_in_token": true,
    "include_in_all_scopes": false,
    "include_in_scopes": [
      "'$SCOPE_ID'"
    ],
    "enabled": true
  }' | jq
```

Expected output:

```
{
  "created_at": "2025-09-19T21:08:28.258487Z",
  "enabled": true,
  "id": "9be4da8d-6a0e-404c-86bd-df11b6c74e32",
  "include_in_all_scopes": false,
  "include_in_scopes": [
    "897f7a59-b6e0-4dca-b0d7-b896c681d50b"
  ],
  "include_in_token": true,
  "name": "claim1",
  "updated_at": "2025-09-19T21:08:28.258487Z",
  "value": "claim1"
}
```

### Create a client in the AuthN Server

The client is the machine-to-machine credential. In this tutorial, Konnect will autogenerate the client ID and secret, but you can alternatively specify one yourself.

Configure the client using the ```/v1/auth-servers/$AUTHN_SERVER_ID/clients``` endpoint:

```
curl -sX POST "https://us.api.konghq.com/v1/auth-servers/$AUTHN_SERVER_ID/clients" \
  -H "Authorization: Bearer $PAT"\
  -H "Content-Type: application/json" \
  --json '{
    "name": "client1",
    "grant_types": [
      "client_credentials"
    ],
    "allow_all_scopes": false,
    "allow_scopes": [
      "'$SCOPE_ID'"
    ],
    "access_token_duration": 3600,
    "id_token_duration": 3600,
    "response_types": [
      "id_token",
      "token"
    ]
  }' | jq
```

Expected output:

```
{
  "access_token_duration": 3600,
  "allow_all_scopes": false,
  "allow_scopes": [
    "897f7a59-b6e0-4dca-b0d7-b896c681d50b"
  ],
  "client_secret": "50e6onbo97p3v8oxq1dlz0me",
  "created_at": "2025-09-19T21:10:46.583789Z",
  "grant_types": [
    "client_credentials"
  ],
  "id": "324939us3jgatrdc",
  "id_token_duration": 3600,
  "labels": {},
  "login_uri": null,
  "name": "client1",
  "redirect_uris": [],
  "response_types": [
    "id_token",
    "token"
  ],
  "token_endpoint_auth_method": "client_secret_post",
  "updated_at": "2025-09-19T21:10:46.583789Z"
}
```

The Client Secret will not be shown again, so copy both ID and Secret:

```
export CLIENT_ID=<YOUR_CLIENT_ID>
export CLIENT_SECRET=<YOUR_CLIENT_SECRET>
```

### Configure the OIDC plugin

You can configure the OIDC plugin to use Kong Identity as the identity provider for your Gateway Services. In this example, you’ll apply the plugin to the control plane globally, but you can alternatively apply it to the Gateway Service.

First, get the ID of the ```serverless-cp1``` control plane you configured in the prerequisites:

```
export CONTROL_PLANE_ID=$(curl -sX GET "https://us.api.konghq.com/v2/control-planes?filter%5Bname%5D%5Bcontains%5D=serverless-cp1" \
    -H "Authorization: Bearer $PAT" | jq -r '.data[0].id')
```


Enable the OIDC plugin globally:

```
curl -sX POST "https://us.api.konghq.com/v2/control-planes/$CONTROL_PLANE_ID/core-entities/plugins/" \
  -H "Authorization: Bearer $PAT"\
  -H "Content-Type: application/json" \
  --json '{
    "name": "openid-connect",
    "config": {
      "issuer": "'$ISSUER_URL'",
      "auth_methods": [
        "bearer"
      ],
      "audience": [
        "http://myhttpbin.dev"
      ]
    }
  }' | jq
```


In this example:

* issuer: Setting that connects the plugin to your IdP (in this case, Kong Identity).
* auth_methods: Specifies that the plugin should use bearer for authentication.


### Generate a token for the client
The Gateway Service requires an access token from the client to access the Service. Generate a token for the client by making a call to the issuer URL:

```
curl -sX POST "$ISSUER_URL/oauth/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=$CLIENT_ID" \
  -d "client_secret=$CLIENT_SECRET" \
  -d "scope=scope1" | jq
```

Export your access token:

```
export ACCESS_TOKEN=$(curl -sX POST "$ISSUER_URL/oauth/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=$CLIENT_ID" \
  -d "client_secret=$CLIENT_SECRET" \
  -d "scope=scope1" | jq -r '.access_token')
```

Check the token

```
echo $ACCESS_TOKEN | jwt decode --json -
```

Expected result
```
{
  "header": {
    "typ": "JWT",
    "alg": "RS256",
    "kid": "b79c88f4-564d-4aea-9be7-3377e95ff758"
  },
  "payload": {
    "aud": [
      "http://myhttpbin.dev"
    ],
    "claim1": "claim1",
    "client_id": "324939us3jgatrdc",
    "exp": 1758321135,
    "iat": 1758317535,
    "iss": "https://dhafxmmgz56rlw6b.us.identity.konghq.com/auth",
    "jti": "a449f513-4c2d-41d6-85d7-685feb758e64",
    "nbf": 1758317535,
    "scope": "scope1",
    "sub": "324939us3jgatrdc"
  }
}
```


### Access the Gateway service using the token

Access the ```httpbin``` Gateway Service using the short-lived token generated by the authorization server from Kong Identity:

```
curl -i -X GET "$DATA_PLANE_URL/oidc-route/get" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

Check the token with:
```
curl -sX GET "$DATA_PLANE_URL/oidc-route/get" \
  -H "Authorization: Bearer $ACCESS_TOKEN" | jq -r '.headers.Authorization' | cut -d ' ' -f 2 | jwt decode -j -
{
  "header": {
    "typ": "JWT",
    "alg": "RS256",
    "kid": "b79c88f4-564d-4aea-9be7-3377e95ff758"
  },
  "payload": {
    "aud": [
      "http://myhttpbin.dev"
    ],
    "claim1": "claim1",
    "client_id": "324939us3jgatrdc",
    "exp": 1758321135,
    "iat": 1758317535,
    "iss": "https://dhafxmmgz56rlw6b.us.identity.konghq.com/auth",
    "jti": "a449f513-4c2d-41d6-85d7-685feb758e64",
    "nbf": 1758317535,
    "scope": "scope1",
    "sub": "324939us3jgatrdc"
  }
}
```

