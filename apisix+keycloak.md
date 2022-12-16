# APISIX and Keycloak

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
1. Create realm
2. Create client
3. Create group
4. Create user

### APISIX
1. Create route
2. Create upstream
