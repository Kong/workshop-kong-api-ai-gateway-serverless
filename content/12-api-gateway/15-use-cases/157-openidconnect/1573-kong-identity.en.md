---
title : "Kong Identity"
weight : 1573
---

### Kong Identity

Kong Identity enables you to use Konnect to generate, authenticate, and authorize API access. You can use Kong Identity to:

* Create authorization servers per region
* Issue and validate access tokens
* Integrate secure authentication into your Kong Gateway APIs


### Create a Kong Service and Route

```
cat > httpbin.yaml << 'EOF'
_format_version: "3.0"
_konnect:
  control_plane_name: serverless-default
_info:
  select_tags:
  - httpbin-service-route
services:
- name: httpbin-service
  host: httpbin.konghq.com
  port: 80
  routes:
  - name: httpbin-route
    paths:
    - /httpbin-route
EOF
```

Submit the declaration

```
deck gateway sync --konnect-token $PAT httpbin.yaml
```


### Client Credentials Grant

This next section describe the OAuth Client Credentials grants implemented by Kong Konnect and [Kong Identity](https://developer.konghq.com/kong-identity/) as the Identity Provider. Let's start instantiating an Authentication Service in Kong Identity.


### Create the Authentication Server in Kong Identity

Before you can configure any authentication plugin, you must first create an auth server in Kong Identity. The auth server name is unique per each organization and each Konnect region.

Create an auth server using the ``/v1/auth-servers`` endpoint:

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
  "created_at": "2025-09-23T13:04:47.789958Z",
  "description": "AuthN Server 1",
  "id": "836fda4d-612c-4faf-9c45-284a0ecd637a",
  "issuer": "https://wt3fgfqb8r7fktwe.us.identity.konghq.com/auth",
  "labels": {},
  "metadata_uri": "https://wt3fgfqb8r7fktwe.us.identity.konghq.com/auth/.well-known/openid-configuration",
  "name": "AuthN_Server_1",
  "signing_algorithm": "RS256",
  "updated_at": "2025-09-23T13:04:47.789958Z"
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



### Configure the auth server with scopes

Configure a scope in your auth server using the ``/v1/auth-servers/$AUTHN_SERVER_ID/scopes`` endpoint:

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
  "created_at": "2025-09-23T13:06:24.252827Z",
  "default": false,
  "description": "scope1",
  "enabled": true,
  "id": "b71c1192-6416-4933-b913-5358904dd234",
  "include_in_metadata": false,
  "name": "scope1",
  "updated_at": "2025-09-23T13:06:24.252827Z"
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
  "created_at": "2025-09-23T13:06:56.096243Z",
  "enabled": true,
  "id": "9b149436-ce85-4fb9-9105-b887546e7b21",
  "include_in_all_scopes": false,
  "include_in_scopes": [
    "b71c1192-6416-4933-b913-5358904dd234"
  ],
  "include_in_token": true,
  "name": "claim1",
  "updated_at": "2025-09-23T13:06:56.096243Z",
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
    "b71c1192-6416-4933-b913-5358904dd234"
  ],
  "client_secret": "8vbywkjyj1zxcgsujljnuge1",
  "created_at": "2025-09-23T13:07:23.691662Z",
  "grant_types": [
    "client_credentials"
  ],
  "id": "fxsn74prsnrcyskm",
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
  "updated_at": "2025-09-23T13:07:23.691662Z"
}
```

The Client Secret will not be shown again, so copy both ID and Secret:

```
export CLIENT_ID=<YOUR_CLIENT_ID>
export CLIENT_SECRET=<YOUR_CLIENT_SECRET>
```

### Configure the OIDC plugin

You can configure the OIDC plugin to use Kong Identity as the identity provider for your Gateway Services. In this example, you’ll apply the plugin to the control plane globally, but you can alternatively apply it to the Gateway Service.

First, get the ID of the ```serverless-default``` control plane you configured in the prerequisites:

```
export CONTROL_PLANE_ID=$(curl -sX GET "https://us.api.konghq.com/v2/control-planes?filter%5Bname%5D%5Bcontains%5D=serverless-default" \
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
    "kid": "a01feebd-5bed-45d7-9244-582010807705"
  },
  "payload": {
    "aud": [
      "http://myhttpbin.dev"
    ],
    "claim1": "claim1",
    "client_id": "fxsn74prsnrcyskm",
    "exp": 1758636665,
    "iat": 1758633065,
    "iss": "https://wt3fgfqb8r7fktwe.us.identity.konghq.com/auth",
    "jti": "370cab31-31f4-44da-b0dc-74577b8a5a81",
    "nbf": 1758633065,
    "scope": "scope1",
    "sub": "fxsn74prsnrcyskm"
  }
}
```


### Access the Gateway service using the token

Access the ```httpbin``` Gateway Service using the short-lived token generated by the authorization server from Kong Identity:

```
curl -i -X GET "$DATA_PLANE_URL/httpbin-route/get" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

Check the token with:
```
curl -sX GET "$DATA_PLANE_URL/httpbin-route/get" \
  -H "Authorization: Bearer $ACCESS_TOKEN" | jq -r '.headers.Authorization' | cut -d ' ' -f 2 | jwt decode -j -
```

Expected output
```
{
  "header": {
    "typ": "JWT",
    "alg": "RS256",
    "kid": "a01feebd-5bed-45d7-9244-582010807705"
  },
  "payload": {
    "aud": [
      "http://myhttpbin.dev"
    ],
    "claim1": "claim1",
    "client_id": "fxsn74prsnrcyskm",
    "exp": 1758636665,
    "iat": 1758633065,
    "iss": "https://wt3fgfqb8r7fktwe.us.identity.konghq.com/auth",
    "jti": "370cab31-31f4-44da-b0dc-74577b8a5a81",
    "nbf": 1758633065,
    "scope": "scope1",
    "sub": "fxsn74prsnrcyskm"
  }
}
```


### Cleaning Up

After testing the configuration, reset the Control Plane:

```
deck gateway reset --konnect-control-plane-name serverless-default --konnect-token $PAT -f
```


Delete the AuthN Server:

```
curl -sX DELETE "https://us.api.konghq.com/v1/auth-servers/$AUTHN_SERVER_ID" \
  -H "Authorization: Bearer $PAT"
```
