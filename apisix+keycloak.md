# Setup APISIX and Keycloak for Authentication and Authorization

## Table of Contents
- [Overview](apisix+keycloak.md#overview)
- [Settings](apisix+keycloak.md#settings)
  - [Keycloak](apisix+keycloak.md#keycloak)
    - [Components](apisix+keycloak.md#components)
    - [Realm](apisix+keycloak.md#create-a-realm)  
    - [Client](apisix+keycloak.md#create-a-client) 
      - [Client Roles](apisix+keycloak.md#create-client-roles) 
      - [Authorization Service](apisix+keycloak.md#setup-authorization-services) 
    - [Groups](apisix+keycloak.md#create-groups)  
    - [Users](apisix+keycloak.md#create-users) 
  - [APISIX](apisix+keycloak.md#apisix)
    - [Upstream](apisix+keycloak.md#create-an-upstream)  
    - [Route](apisix+keycloak.md#create-a-route) 
- [Testing](apisix+keycloak.md#testing)

## Overview
The goal is to setup the gateway layer by using APISIX as an API Gateway and Keycloak as IAM. The result will look like the below image:

![image](https://user-images.githubusercontent.com/6086297/208076514-cae0ca41-59da-4190-9829-8b16017e9d30.png)

### Generate access token flow
First, Resource Owner must generate the access token via Keyloack by using own credentail  

### Access resource flow
![image](https://user-images.githubusercontent.com/6086297/208073850-6011c5dd-4f0b-4361-93e7-1d5c65db3473.png)

Second, Resource Owner can access to Resource Server behind APISIX by filling the access token into **Authorization** HTTP Header. APISIX authenticates and authorizes the access token via Keycloak and proceeds to Resource Server if the authentication and authorization is successful.

## Settings
### Keycloak
#### Components
![image](https://user-images.githubusercontent.com/6086297/208571389-1da4beca-8e2b-4aa1-86f8-44ab03e9e7fc.png)

#### Create a realm
- Create a realm named **apisix_authz**
![image](https://user-images.githubusercontent.com/6086297/208343371-0d107df1-628f-48a4-b181-be910e09765c.png)

#### Create a client
- Create a *OpenID Connect* client named **rule-engine-system** and enable **authorization services**
![image](https://user-images.githubusercontent.com/6086297/208343540-c2faaf3c-a531-4ae7-aba5-6ea46b1fb5ae.png)
![image](https://user-images.githubusercontent.com/6086297/208343569-8044a2e7-629e-44d6-ab07-4be39bd03a99.png)

##### Create client roles
Access to **rule-engine-system** client and select **Roles** tab to create client roles 
- Create a client role named **role_only**
![image](https://user-images.githubusercontent.com/6086297/208346733-1c774ed7-fad6-42b8-afe3-81938737c99b.png)

- Create a client role named **workflow_only**
![image](https://user-images.githubusercontent.com/6086297/208346774-2da32519-bb4d-4d23-bc9b-c08e7b1bab19.png)

##### Setup authorization services
Access to **rule-engine-system** client and select **Authorization** tab to setup authorization services
###### Create authorization scopes
- Create a authorization scope named **GET**
![image](https://user-images.githubusercontent.com/6086297/208348904-1f7c4b0e-8d54-4795-acc3-454b62d5db59.png)

###### Create resources
- Create a resource named **rule_resource**
![image](https://user-images.githubusercontent.com/6086297/208347862-33003653-cdc7-414d-9bb3-39485bb51fb6.png)

- Create a resource named **workflow_resource**
![image](https://user-images.githubusercontent.com/6086297/208347965-158e05fe-471f-40a7-a717-fdc8002b05a5.png)

###### Create policies
- Create a *Role* policy named **rule_policy**
![image](https://user-images.githubusercontent.com/6086297/208348370-b0d96818-d85d-4b8b-9d6d-e23b9dde4990.png)

- Create a *Role* policy named **workflow_policy**
![image](https://user-images.githubusercontent.com/6086297/208348643-b7ba2271-4126-40a9-abf4-23b66e1eae06.png)

###### Create permissions
- Create a *Resource-Based* permission named **only_access_rule**
![image](https://user-images.githubusercontent.com/6086297/208349234-64bc0ac2-e7f9-4efd-9593-a59d4ac0d33e.png)

- Create a *Resource-Based* permission named **only_access_workflow**
![image](https://user-images.githubusercontent.com/6086297/208349305-0e30e3ba-1a78-4e1f-9422-b3b1b772a094.png)

#### Create groups
- Create a group named **admin**
![image](https://user-images.githubusercontent.com/6086297/208361774-f6863460-8515-412c-b951-f5e00c4a6600.png)
  - Assign roles: select **Role mapping** tab to assign **rule_only** and **workflow_only** roles
![image](https://user-images.githubusercontent.com/6086297/208362133-d22f2950-791c-4970-bdbe-f29a22878ede.png)

#### Create users
- Create a user named **admin** and assign this user into **admin** group. This user can access to **rule_resource** and **workflow_resource**
![image](https://user-images.githubusercontent.com/6086297/208362690-24412e98-a811-410b-b491-009e2b8f1436.png)

- Create a user named **only_rule** and assign **rule_only** role. This user only access to **rule_resource**
![image](https://user-images.githubusercontent.com/6086297/208362979-48c02625-b6d4-4a90-ba02-0098ff000e60.png)
  - Assign roles: select **Role mapping** tab to assign **rule_only** role
![image](https://user-images.githubusercontent.com/6086297/208365639-d7b8be8f-143b-4660-a69f-d4ff3ffb9111.png)

- Create a user named **only_workflow** and assign **workflow_only** role. This user only access to **workflow_resource**
![image](https://user-images.githubusercontent.com/6086297/208363022-396382ee-854f-446a-a36d-65b6e2d81b3f.png)
  - Assign roles: select **Role mapping** tab to assign **workflow_only** role
![image](https://user-images.githubusercontent.com/6086297/208366289-17ea37b1-b258-447a-9bda-30ed5d9a5d1e.png)

### APISIX
#### Create an upstream
```json
{
  "name": "ups-to-res",
  "nodes": [
    {
      "host": "{resource server host}",
      "port": {resource server port},
      "weight": 1
    }
  ],
  "timeout": {
    "connect": 6,
    "send": 6,
    "read": 6
  },
  "type": "roundrobin",
  "scheme": "http",
  "pass_host": "pass",
  "name": "ups-to-res",
  "keepalive_pool": {
    "idle_timeout": 60,
    "requests": 1000,
    "size": 320
  }
}
```
#### Create a route
```json
{
  "uri": "/v1/res*",
  "name": "res-api",
  "methods": [
    "GET",
    "POST",
    "PUT",
    "DELETE",
    "PATCH",
    "HEAD",
    "OPTIONS",
    "CONNECT",
    "TRACE"
  ],
  "plugins": {
    "authz-keycloak": {
      "access_token_expires_in": 300,
      "access_token_expires_leeway": 0,
      "cache_ttl_seconds": 86400,
      "client_id": "rule-engine-system",
      "client_secret": "{client secret of rule-engine-system}",
      "disable": false,
      "discovery": "http://{keycloak host}:{keycloak port}/realms/apisix-authz/.well-known/uma2-configuration",
      "grant_type": "urn:ietf:params:oauth:grant-type:uma-ticket",
      "http_method_as_scope": true,
      "keepalive": true,
      "keepalive_pool": 5,
      "keepalive_timeout": 60000,
      "lazy_load_paths": true,
      "policy_enforcement_mode": "ENFORCING",
      "refresh_token_expires_in": 3600,
      "refresh_token_expires_leeway": 0,
      "resource_registration_endpoint": "http://{keycloak host}:{keycloak port}/realms/apisix-authz/authz/protection/resource_set",
      "ssl_verify": true,
      "timeout": 3000
    },
    "openid-connect": {
      "access_token_in_authorization_header": true,
      "bearer_only": true,
      "client_id": "rule-engine-system",
      "client_secret": "{client secret of rule-engine-system}",
      "disable": false,
      "discovery": "http://{keycloak host}:{keycloak port}/realms/apisix-authz/.well-known/openid-configuration",
      "introspection_endpoint_auth_method": "client_secret_basic",
      "logout_path": "/logout",
      "realm": "apisix-authz",
      "redirect_uri": "/v1/res/",
      "scope": "openid",
      "set_access_token_header": true,
      "set_id_token_header": true,
      "set_refresh_token_header": false,
      "set_userinfo_header": true,
      "ssl_verify": false,
      "timeout": 3,
      "use_pkce": false
    }
  },
  "upstream_id": "{upstream id}",
  "status": 1
}
```
## Testing
### Generate access token

#### For only_rule
```
POST /realms/apisix-authz/protocol/openid-connect/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded
User-Agent: PostmanRuntime/7.29.2
Accept: */*
Postman-Token: 3af18a14-026d-4221-b0cc-4f827eaa7922
Host: 10.251.0.127:32219
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Content-Length: 127
 
client_secret={client secret of rule-engine-system}&grant_type=password&username=only_rule&password=123&client_id=rule-engine-system
 
HTTP/1.1 200 OK
Referrer-Policy: no-referrer
X-Frame-Options: SAMEORIGIN
Strict-Transport-Security: max-age=31536000; includeSubDomains
Cache-Control: no-store
X-Content-Type-Options: nosniff
Set-Cookie: KEYCLOAK_LOCALE=; Version=1; Comment=Expiring cookie; Expires=Thu, 01-Jan-1970 00:00:10 GMT; Max-Age=0; Path=/realms/apisix-authz/; HttpOnly
Set-Cookie: KC_RESTART=; Version=1; Expires=Thu, 01-Jan-1970 00:00:10 GMT; Max-Age=0; Path=/realms/apisix-authz/; HttpOnly
Pragma: no-cache
X-XSS-Protection: 1; mode=block
Content-Type: application/json
content-length: 2019
 
{"access_token":"eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJ4YUgwdFhjRnNOQVFfc2dCUjFldGtVRkM0WTRLZS1DTGR3UGhWdEV5eGRnIn0.eyJleHAiOjE2NzE0NDI4NzgsImlhdCI6MTY3MTQ0MjU3OCwianRpIjoiNzIzNGU1Y2UtMTZkMS00MDhjLTkwNDQtODI4NjlmNTU4Y2Q1IiwiaXNzIjoiaHR0cDovLzEwLjI1MS4wLjEyNzozMjIxOS9yZWFsbXMvYXBpc2l4LWF1dGh6Iiwic3ViIjoiODQ3YmY2NjAtNzg5Ny00ZDAxLWEyOGUtYmZkNTA2ZTkwM2JiIiwidHlwIjoiQmVhcmVyIiwiYXpwIjoicnVsZS1lbmdpbmUtc3lzdGVtIiwic2Vzc2lvbl9zdGF0ZSI6IjdkNDMxM2Q2LTk1MmItNDAyMC1iNTUxLThhZDRjNTU5ZGE4MCIsImFjciI6IjEiLCJyZXNvdXJjZV9hY2Nlc3MiOnsicnVsZS1lbmdpbmUtc3lzdGVtIjp7InJvbGVzIjpbInJ1bGVfb25seSJdfX0sInNjb3BlIjoiZW1haWwgcHJvZmlsZSIsInNpZCI6IjdkNDMxM2Q2LTk1MmItNDAyMC1iNTUxLThhZDRjNTU5ZGE4MCIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwicHJlZmVycmVkX3VzZXJuYW1lIjoib25seV9ydWxlIiwiZ2l2ZW5fbmFtZSI6IiIsImZhbWlseV9uYW1lIjoiIn0.XtnRM3AvcNnDlUzx5tRGktK03VRNwvIelptNU6ZOEmGNaPgYhUe7n5PA5Hm5cAusBJtjUrWoi_4xKIZknOaMdU7I-9mxwvx-wWA4o_TnGDgJQK4kvqK_jRsiBcFsqydnojXmsRSdLSlon4X02CGCYsIpUiIRPc_So4JnPbKqPijovY2A49U3hzfyBBeO0CZRGoCWE3XxBbNHOo8Pe19pvvgDozvy9vepVqwyMa0QlthFk04zBLvmdjI99t0Xs8yz2MlczFJGYtVvNEMRG1PvX7c_BB88bR6PEWBMNTxqPkMjnMraN9iSAxCTIIO8X9lRIMjAWWyY0VGfnO5Ftt2Qlg","expires_in":300,"refresh_expires_in":1800,"refresh_token":"eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJjOTU1ODFiZS0zOWU3LTQxMmItYmVhZC0yNjZkZmU5YmJiNDMifQ.eyJleHAiOjE2NzE0NDQzNzgsImlhdCI6MTY3MTQ0MjU3OCwianRpIjoiY2EwZDJlZmItMzg1MS00YjIwLWFkMGQtMGNiMzMwNWM1ZWU0IiwiaXNzIjoiaHR0cDovLzEwLjI1MS4wLjEyNzozMjIxOS9yZWFsbXMvYXBpc2l4LWF1dGh6IiwiYXVkIjoiaHR0cDovLzEwLjI1MS4wLjEyNzozMjIxOS9yZWFsbXMvYXBpc2l4LWF1dGh6Iiwic3ViIjoiODQ3YmY2NjAtNzg5Ny00ZDAxLWEyOGUtYmZkNTA2ZTkwM2JiIiwidHlwIjoiUmVmcmVzaCIsImF6cCI6InJ1bGUtZW5naW5lLXN5c3RlbSIsInNlc3Npb25fc3RhdGUiOiI3ZDQzMTNkNi05NTJiLTQwMjAtYjU1MS04YWQ0YzU1OWRhODAiLCJzY29wZSI6ImVtYWlsIHByb2ZpbGUiLCJzaWQiOiI3ZDQzMTNkNi05NTJiLTQwMjAtYjU1MS04YWQ0YzU1OWRhODAifQ.BAyL56w8YVUsiLFYHuCnp-aLL6mcsmmDEV6u_kDZp_c","token_type":"Bearer","not-before-policy":0,"session_state":"7d4313d6-952b-4020-b551-8ad4c559da80","scope":"email profile"}
```

#### For only_workflow
```
POST /realms/apisix-authz/protocol/openid-connect/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded
User-Agent: PostmanRuntime/7.29.2
Accept: */*
Postman-Token: 1697dafe-c518-4e3e-9c64-d9c9a83b882b
Host: 10.251.0.127:32219
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Content-Length: 131
 
client_secret={client secret of rule-engine-system}&grant_type=password&username=only_workflow&password=123&client_id=rule-engine-system
 
HTTP/1.1 200 OK
Referrer-Policy: no-referrer
X-Frame-Options: SAMEORIGIN
Strict-Transport-Security: max-age=31536000; includeSubDomains
Cache-Control: no-store
X-Content-Type-Options: nosniff
Set-Cookie: KEYCLOAK_LOCALE=; Version=1; Comment=Expiring cookie; Expires=Thu, 01-Jan-1970 00:00:10 GMT; Max-Age=0; Path=/realms/apisix-authz/; HttpOnly
Set-Cookie: KC_RESTART=; Version=1; Expires=Thu, 01-Jan-1970 00:00:10 GMT; Max-Age=0; Path=/realms/apisix-authz/; HttpOnly
Pragma: no-cache
X-XSS-Protection: 1; mode=block
Content-Type: application/json
content-length: 2030
 
{"access_token":"eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJ4YUgwdFhjRnNOQVFfc2dCUjFldGtVRkM0WTRLZS1DTGR3UGhWdEV5eGRnIn0.eyJleHAiOjE2NzE0NDMwMzQsImlhdCI6MTY3MTQ0MjczNCwianRpIjoiODg0MzcxN2YtY2NjNy00NjY4LTgxM2UtN2NhMzU5YmFiNjM4IiwiaXNzIjoiaHR0cDovLzEwLjI1MS4wLjEyNzozMjIxOS9yZWFsbXMvYXBpc2l4LWF1dGh6Iiwic3ViIjoiN2UwN2I1Y2MtNzkyOS00YmQ4LThkMTYtMmIzMmE2ZWIzMmExIiwidHlwIjoiQmVhcmVyIiwiYXpwIjoicnVsZS1lbmdpbmUtc3lzdGVtIiwic2Vzc2lvbl9zdGF0ZSI6ImFkODAyMjBkLTI1ZjAtNGIyNy1iZDI5LTA1ZTgxNDU4NzE2NCIsImFjciI6IjEiLCJyZXNvdXJjZV9hY2Nlc3MiOnsicnVsZS1lbmdpbmUtc3lzdGVtIjp7InJvbGVzIjpbIndvcmtmbG93X29ubHkiXX19LCJzY29wZSI6ImVtYWlsIHByb2ZpbGUiLCJzaWQiOiJhZDgwMjIwZC0yNWYwLTRiMjctYmQyOS0wNWU4MTQ1ODcxNjQiLCJlbWFpbF92ZXJpZmllZCI6ZmFsc2UsInByZWZlcnJlZF91c2VybmFtZSI6Im9ubHlfd29ya2Zsb3ciLCJnaXZlbl9uYW1lIjoiIiwiZmFtaWx5X25hbWUiOiIifQ.EEDQnjifYcCcD2KsVNDJVwUKy-oW3Okho8C2GN6-FPnsl5_XHalzv0DoJHknG-1esgRliCMv67lAZYXZD_9u4IW-A8_8AXUP95x7fY_VxhScmX1MCI0up3Cw1Q7Pq-e8ybbsRK-9QS-i_WWKUTaA0GPtNwCM_NixkXoZYBQLaJMEwylO1ZlZUWkWIIWzuL51Z3zPAdKRNmY9xnwljsNWM1ghx-3etdBHOCQhtUpCJIUlMW1eK_IJlX6r8on2czdyAVo0xS8ZYcwtlmmClFiExAezZvrt8Z6i-t2OT2ifns-tg_aYNPNMjmGI0nBg0vZoTlI0Dw01y65NvatitINgkw","expires_in":300,"refresh_expires_in":1800,"refresh_token":"eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJjOTU1ODFiZS0zOWU3LTQxMmItYmVhZC0yNjZkZmU5YmJiNDMifQ.eyJleHAiOjE2NzE0NDQ1MzQsImlhdCI6MTY3MTQ0MjczNCwianRpIjoiNmQzMDk0ZTktZmMwYi00ZjIyLTkyOTktZmYyOTZkZDA5N2I5IiwiaXNzIjoiaHR0cDovLzEwLjI1MS4wLjEyNzozMjIxOS9yZWFsbXMvYXBpc2l4LWF1dGh6IiwiYXVkIjoiaHR0cDovLzEwLjI1MS4wLjEyNzozMjIxOS9yZWFsbXMvYXBpc2l4LWF1dGh6Iiwic3ViIjoiN2UwN2I1Y2MtNzkyOS00YmQ4LThkMTYtMmIzMmE2ZWIzMmExIiwidHlwIjoiUmVmcmVzaCIsImF6cCI6InJ1bGUtZW5naW5lLXN5c3RlbSIsInNlc3Npb25fc3RhdGUiOiJhZDgwMjIwZC0yNWYwLTRiMjctYmQyOS0wNWU4MTQ1ODcxNjQiLCJzY29wZSI6ImVtYWlsIHByb2ZpbGUiLCJzaWQiOiJhZDgwMjIwZC0yNWYwLTRiMjctYmQyOS0wNWU4MTQ1ODcxNjQifQ.VIi_iwJLZGmIgtuRWB_Kyvvha9374RGthP48piM2SuI","token_type":"Bearer","not-before-policy":0,"session_state":"ad80220d-25f0-4b27-bd29-05e814587164","scope":"email profile"}
```

### Access resource

#### For only_rule
```
GET /v1/res/management/rules HTTP/1.1
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJ4YUgwdFhjRnNOQVFfc2dCUjFldGtVRkM0WTRLZS1DTGR3UGhWdEV5eGRnIn0.eyJleHAiOjE2NzE0NDI4NzgsImlhdCI6MTY3MTQ0MjU3OCwianRpIjoiNzIzNGU1Y2UtMTZkMS00MDhjLTkwNDQtODI4NjlmNTU4Y2Q1IiwiaXNzIjoiaHR0cDovLzEwLjI1MS4wLjEyNzozMjIxOS9yZWFsbXMvYXBpc2l4LWF1dGh6Iiwic3ViIjoiODQ3YmY2NjAtNzg5Ny00ZDAxLWEyOGUtYmZkNTA2ZTkwM2JiIiwidHlwIjoiQmVhcmVyIiwiYXpwIjoicnVsZS1lbmdpbmUtc3lzdGVtIiwic2Vzc2lvbl9zdGF0ZSI6IjdkNDMxM2Q2LTk1MmItNDAyMC1iNTUxLThhZDRjNTU5ZGE4MCIsImFjciI6IjEiLCJyZXNvdXJjZV9hY2Nlc3MiOnsicnVsZS1lbmdpbmUtc3lzdGVtIjp7InJvbGVzIjpbInJ1bGVfb25seSJdfX0sInNjb3BlIjoiZW1haWwgcHJvZmlsZSIsInNpZCI6IjdkNDMxM2Q2LTk1MmItNDAyMC1iNTUxLThhZDRjNTU5ZGE4MCIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwicHJlZmVycmVkX3VzZXJuYW1lIjoib25seV9ydWxlIiwiZ2l2ZW5fbmFtZSI6IiIsImZhbWlseV9uYW1lIjoiIn0.XtnRM3AvcNnDlUzx5tRGktK03VRNwvIelptNU6ZOEmGNaPgYhUe7n5PA5Hm5cAusBJtjUrWoi_4xKIZknOaMdU7I-9mxwvx-wWA4o_TnGDgJQK4kvqK_jRsiBcFsqydnojXmsRSdLSlon4X02CGCYsIpUiIRPc_So4JnPbKqPijovY2A49U3hzfyBBeO0CZRGoCWE3XxBbNHOo8Pe19pvvgDozvy9vepVqwyMa0QlthFk04zBLvmdjI99t0Xs8yz2MlczFJGYtVvNEMRG1PvX7c_BB88bR6PEWBMNTxqPkMjnMraN9iSAxCTIIO8X9lRIMjAWWyY0VGfnO5Ftt2Qlg
User-Agent: PostmanRuntime/7.29.2
Accept: */*
Postman-Token: fd00acce-3d32-4457-a1d6-d85c2e016120
Host: 10.251.0.127:32538
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
 
HTTP/1.1 200 OK
Content-Type: application/json
Transfer-Encoding: chunked
Connection: keep-alive
Grpc-Metadata-Content-Type: application/grpc
Date: Mon, 19 Dec 2022 09:36:41 GMT
Server: APISIX/2.15.0
 
{...}
```

#### For only_workflow
```
GET /v1/res/management/workflows HTTP/1.1
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJ4YUgwdFhjRnNOQVFfc2dCUjFldGtVRkM0WTRLZS1DTGR3UGhWdEV5eGRnIn0.eyJleHAiOjE2NzE0NDMwMzQsImlhdCI6MTY3MTQ0MjczNCwianRpIjoiODg0MzcxN2YtY2NjNy00NjY4LTgxM2UtN2NhMzU5YmFiNjM4IiwiaXNzIjoiaHR0cDovLzEwLjI1MS4wLjEyNzozMjIxOS9yZWFsbXMvYXBpc2l4LWF1dGh6Iiwic3ViIjoiN2UwN2I1Y2MtNzkyOS00YmQ4LThkMTYtMmIzMmE2ZWIzMmExIiwidHlwIjoiQmVhcmVyIiwiYXpwIjoicnVsZS1lbmdpbmUtc3lzdGVtIiwic2Vzc2lvbl9zdGF0ZSI6ImFkODAyMjBkLTI1ZjAtNGIyNy1iZDI5LTA1ZTgxNDU4NzE2NCIsImFjciI6IjEiLCJyZXNvdXJjZV9hY2Nlc3MiOnsicnVsZS1lbmdpbmUtc3lzdGVtIjp7InJvbGVzIjpbIndvcmtmbG93X29ubHkiXX19LCJzY29wZSI6ImVtYWlsIHByb2ZpbGUiLCJzaWQiOiJhZDgwMjIwZC0yNWYwLTRiMjctYmQyOS0wNWU4MTQ1ODcxNjQiLCJlbWFpbF92ZXJpZmllZCI6ZmFsc2UsInByZWZlcnJlZF91c2VybmFtZSI6Im9ubHlfd29ya2Zsb3ciLCJnaXZlbl9uYW1lIjoiIiwiZmFtaWx5X25hbWUiOiIifQ.EEDQnjifYcCcD2KsVNDJVwUKy-oW3Okho8C2GN6-FPnsl5_XHalzv0DoJHknG-1esgRliCMv67lAZYXZD_9u4IW-A8_8AXUP95x7fY_VxhScmX1MCI0up3Cw1Q7Pq-e8ybbsRK-9QS-i_WWKUTaA0GPtNwCM_NixkXoZYBQLaJMEwylO1ZlZUWkWIIWzuL51Z3zPAdKRNmY9xnwljsNWM1ghx-3etdBHOCQhtUpCJIUlMW1eK_IJlX6r8on2czdyAVo0xS8ZYcwtlmmClFiExAezZvrt8Z6i-t2OT2ifns-tg_aYNPNMjmGI0nBg0vZoTlI0Dw01y65NvatitINgkw
User-Agent: PostmanRuntime/7.29.2
Accept: */*
Postman-Token: c1eb5366-0cde-45f9-b0bb-2820445796ec
Host: 10.251.0.127:32538
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
 
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 348
Connection: keep-alive
Grpc-Metadata-Content-Type: application/grpc
Date: Mon, 19 Dec 2022 09:39:14 GMT
Server: APISIX/2.15.0
 
{...}
```
