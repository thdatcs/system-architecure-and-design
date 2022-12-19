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
Create a client named **rule-engine-system** and enable **authorization services**
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
###### Create scopes
Create a scope named **GET**
![image](https://user-images.githubusercontent.com/6086297/208347376-0c374965-762f-43ab-99f8-20d6b441ca3b.png)

###### Create resources
- Create a resource named **rule_resource**
![image](https://user-images.githubusercontent.com/6086297/208347862-33003653-cdc7-414d-9bb3-39485bb51fb6.png)

- Create a resource named **workflow_resource**
![image](https://user-images.githubusercontent.com/6086297/208347965-158e05fe-471f-40a7-a717-fdc8002b05a5.png)

###### Create policies
###### Create permissions

#### Create groups

#### Create users

### APISIX
#### Create a route
#### Create an upstream
