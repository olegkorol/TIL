# Cognito - OAuth2.0 Client Credentials

Use cognito as a JWT issuer that supports the Client Credentials OAuth flow for Server-to-Server auth.

## 1. Create a user pool

```bash
aws cognito-idp create-user-pool --pool-name <API_NAME>
```

Save the user-pool id and its ARN for later.

## 2. Create a resource server

```bash
aws cognito-idp create-resource-server --name <SOURCE_NAME> --identifier <SOURCE_ID> --user-pool-id <POOL_ID_FROM_PREV_STEP> --scopes ScopeName=<NAME_eg_"get">,ScopeDescription=<DESCRIPTION> ScopeName=<NAME_eg_"post">,ScopeDescription=<DESCRIPTION>
```

## 3. Create a client app
  
Create the client that will be authorising against Cognito and choose what scopes (defined in the previous step) and OAuth flows (in this case, Client Credentials) it is able to use.
  
```bash
aws cognito-idp create-user-pool-client --user-pool-id <POOL_ID> --allowed-o-auth-flows client_credentials --client-name <NAME> --generate-secret --allowed-o-auth-scopes <RESOURCE_NAME/SCOPE_NAME_eg_profiles/get> --allowed-o-auth-flows-user-pool-client
```

The *client_id* and *client_secret* will be given after the client has been created. Save them in a safe place.

## 4. Add a domain

...to authenticate against and get JWTs from.

```bash
aws cognito-idp create-user-pool-domain --domain <*UNIQUE*_DOMAIN_NAME> --user-pool-id <POOL_ID>
```

This is how the domain will be look like:

```text
https://<DOMAIN_NAME>.auth.<AWS_REGION>.amazoncognito.com
```

This will be referred to as *<AUTH_DOMAIN>* in the next step.

## This is how the client would get a token thereafter

1. Generate a Basic Auth token:

```bash
echo 'client_id:client_secret' | base64
```

This will be referred to as *<BASIC_AUTH_TOKEN>* in the next step.

2. Get the JWT:

```bash
curl -X POST \
  <AUTH_DOMAIN>/oauth2/token \
  -H 'authorization: Basic <BASIC_AUTH_TOKEN>' \
  -H 'content-type: application/x-www-form-urlencoded' \
  -d 'grant_type=client_credentials&scope=<ANY_OF_THE_CREATED_SCOPES_eg_profiles/get>'
```

If later you want to verify the signature of the token, to make sure it was issued by your Incognito instance, the JWKS endpoint would look like this:

```text
https://cognito-idp.<AWS_REGION>.amazonaws.com/<COGNITO_POOL_ID>/.well-known/jwks.json
```

If you are using AWS API Gateway to manage your API, you can add Cognito as an Authorizer by choosing *JWT Auhtorizer* and adding the URL of your user pool.
