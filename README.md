# Building 'eShop' from Zero to Hero: Provisioning Chatbot AI Service in AppHost project

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

## 2. 

