# Building 'eShop' from Zero to Hero: Provisioning Chatbot AI Service in AppHost project

We can register and configure the **AI Chatbot Service** in the **AppHost** project or in the **WebApp** project

In the previous sample **Step6**, we configured the AI Service in the **WebApp**, and now in this **Step7** we are going to configure the AI Service in the**AppHost**

## 1. Purpose of Provisioning AI Service in AppHost

The **AppHost** project manages the infrastructure configuration and orchestration for your distributed application

By provisioning **OpenAI** in the **AppHost**, you achieve the following benefits:

**a) Centralized Resource Management**: All infrastructure components (e.g., OpenAI services, databases, message queues) are configured in a single location (AppHost), ensuring consistency across the application

**b) Simplified Environment Management**: The AppHost dynamically provisions OpenAI services, sets environment variables (e.g., AI__OPENAI__CHATMODEL), and passes them to dependent projects (like WebApp)

This ensures the WebApp is properly configured without duplicating resource setup logic

**c) Dynamic Provisioning**: If the OpenAI resource doesn’t exist, the AppHost can provision it dynamically in Azure (via AddAzureOpenAI)

This abstracts the infrastructure complexity from the WebApp

**d) Dependency Injection Setup**: The AppHost ensures the WebApp receives references to OpenAI resources, such as the connection string and model name, without hardcoding these values in WebApp

**e) Scalability**: If other projects (e.g., CatalogApi, OrderingApi) also need OpenAI, the AppHost avoids redundant configuration by sharing the same resource across projects

## 2. We download the solution from github repo

The start point for this sample is this github repo: 

https://github.com/luiscoco/eShop_Tutorial-Step6_Adding_Chatbot

We donwload the Step6 application code from the above github repo

## 3. We modify the AppHost middleware



## 4. We add the AI ConnectionString in the appsettings.json (AppHost project)

## 5. We modify the WebApp middleware

## 6. We remove the AI connection string from appsettings.json (WebApp project)

## 7. We run the Application and verify the results


