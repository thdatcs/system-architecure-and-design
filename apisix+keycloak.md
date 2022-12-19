# Setup APISIX and Keycloak for Authentication and Authorization

## Overview
The goal is to setup the gateway layer by using APISIX as an API Gateway and Keycloak as IAM. The result will look like the below image:

![image](https://user-images.githubusercontent.com/6086297/208076514-cae0ca41-59da-4190-9829-8b16017e9d30.png)

### Generate token flow
First, Resource Owner must generate the access token via Keyloack by using own credentail  

### Access resource flow
![image](https://user-images.githubusercontent.com/6086297/208073850-6011c5dd-4f0b-4361-93e7-1d5c65db3473.png)

Second, Resource Owner can access to Resource Server behind APISIX by filling the access token into **Authorization** HTTP Header. APISIX authenticates and authorizes the access token via Keycloak and proceeds to Resource Server if the authentication and authorization is successful.

## Settings
### Keycloak
#### Create a realm
Create a realm named **apisix_authz**
![image](https://user-images.githubusercontent.com/6086297/208343371-0d107df1-628f-48a4-b181-be910e09765c.png)

#### Create a client
Create a *OpenID Connect* client named **rule-engine-system** and enable **authorization services**
![image](https://user-images.githubusercontent.com/6086297/208343540-c2faaf3c-a531-4ae7-aba5-6ea46b1fb5ae.png)
![image](https://user-images.githubusercontent.com/6086297/208343569-8044a2e7-629e-44d6-ab07-4be39bd03a99.png)

##### Create client roles
Access to **rule-engine-system** client and select **Roles** tab to create client roles 
- Create a client role named **role_only**
![image](https://user-images.githubusercontent.com/6086297/208346733-1c774ed7-fad6-42b8-afe3-81938737c99b.png)

- Create a client role named **workflow_only**
![image](https://user-images.githubusercontent.com/6086297/208346774-2da32519-bb4d-4d23-bc9b-c08e7b1bab19.png)

##### Setup authorization services
Access to **rule-engine-system** client and select **Authorization** to setup authorization services
###### Create authorization scopes
Create a authorization scope named **GET**
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

#### Create users

### APISIX
#### Create a route
#### Create an upstream
