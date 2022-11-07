# Overview

This is an example on how to setup AAD for authentication and authorization of API's in API Management.

It's using Roles to drive the authorization of the API call's to allow more granular control of an API estate.

It will be creating 3x AAD Application Registrations to cover:
- 1x API  Application
- 2x Client Applications

There's a script that can be run to setup the application registrations in ADD 


## Prerequisites  

- Azure Tenant with Global Admin access
- Azure Subscription with owner access
- VS Code / Git
- Instance of API Management for use


## Setup AAD Application Registrations

Run the following script: ``` ./API-App-Registration.azcli ```

This creates 3x Application Registrations for securing the API's with AAD



## Add API Application policy update

In the API Management inboard policy. Addt  he following:

``` 
<validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized" require-expiration-time="true" require-scheme="Bearer" require-signed-tokens="true">
            <openid-config url="https://login.microsoftonline.com/<TenantId>/.well-known/openid-configuration" />
            <required-claims>
                <claim name="aud" match="all">
                    <value><api://<ClientId>></value>
                </claim>
                <claim name="roles" match="all">
                    <value>returns.write</value>
                </claim>
            </required-claims>
        </validate-jwt>
```
Key to enter
 - The Tenant Id where the application registrations are. 
 - The audience (api://\<clientId\>)
 - The Roles that will be accepted for calling that API.



## Get Bearer Token

The generate a Bearer token for calling the API's you need to following the below:

###  Request Type     

POST

### URL

    https://login.microsoftonline.com/\<SubscriptionId\>/oauth2/v2.0/token

Replace the SubscriptionId with a subscription from the tenant that 

### Body (form-data) 
| Key  | Value  | Note|
|---|---|---|
| grant_type | client_credentials  |   |  
|  client_id | \<Application Registration client Id\>   |   |  
| client_secret  | \<Application Registration client Secret\>   |   
| scope  |  api://\<Application Registration client Id\>/.default |   
| redirect_uri  |   |   

Example: 

![Postman Get Bearer Token](./images/get%20bearer%20token.png)

Test the token by pasting the access_token into https://jwt.io  
Should be able to see the Roles assigned in the token (e.g. sales.read, sales.write)



## Call API

When calling the API the following needs to be included in the headers:

| Header | Value | Note |
| --- | --- | --- |
| Authorization | Bearer \<access_token\> |  |
| Ocp-Apim-Subscription-Key | \<API Key\> |  |






