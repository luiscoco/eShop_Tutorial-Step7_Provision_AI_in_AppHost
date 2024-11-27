# Building 'eShop' from Zero to Hero: Provisioning Chatbot AI Service in AppHost project

We can register and configure the **AI Chatbot Service** in the **AppHost** project or in the **WebApp** project

In the previous sample **Step6**:

https://github.com/luiscoco/eShop_Tutorial-Step6_Adding_Chatbot, 

we configured the AI Service in the **WebApp**, and now in this **Step7**:

https://github.com/luiscoco/eShop_Tutorial-Step7_Provision_AI_in_AppHost

we are going to configure the AI Service in the**AppHost**

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

We add the following code to register the AI Service in the **Program.cs**:

```csharp
const string openAIName = "openai";
const string chatModelName = "gpt-4o";

IResourceBuilder<IResourceWithConnectionString> openAI;
openAI = builder.AddConnectionString(openAIName);

var AzureOpenAI_key = builder.Configuration["ConnectionStrings:openai:Key"];
var AzureOpenAI_endpoint = builder.Configuration["ConnectionStrings:openai:Endpoint"];

webApp
    .WithReference(openAI)
    .WithEnvironment("AI__OPENAI__CHATMODEL", chatModelName)
    .WithEnvironment("ConnectionStrings:openai:Endpoint", AzureOpenAI_endpoint)
    .WithEnvironment("ConnectionStrings:openai:Key", AzureOpenAI_key);
```

This code configures a web application to use **Azure OpenAI** services

It Sets up a resource connection (openAI)

It Retrieves the necessary connection details (API key and endpoint)

It Configures the application with environment variables, specifying the AI model to use and connection information

This setup allows the application to authenticate with Azure OpenAI services and interact with the specified chat model (gpt-4o)

Here's a breakdown of its components:

**Constants Declaration**

```csharp
const string openAIName = "openai";
const string chatModelName = "gpt-4o";
```

openAIName: Specifies the name identifier for the OpenAI resource being configured

chatModelName: Specifies the name of the AI model to use, such as "gpt-4o"

**OpenAI Resource Configuration**

```csharp
IResourceBuilder<IResourceWithConnectionString> openAI;
openAI = builder.AddConnectionString(openAIName);
```

builder.AddConnectionString(openAIName): Associates the OpenAI resource with a connection string using the specified name ("openai")

This sets up the resource to connect to Azure OpenAI services

The type IResourceBuilder<IResourceWithConnectionString> indicates that this builder is for resources that require a connection string

**Retrieving Azure OpenAI Configuration**

```csharp
var AzureOpenAI_key = builder.Configuration["ConnectionStrings:openai:Key"];
var AzureOpenAI_endpoint = builder.Configuration["ConnectionStrings:openai:Endpoint"];
```

These lines retrieve the OpenAI connection details (API key and endpoint) from the application's configuration file (e.g., appsettings.json or environment variables)

AzureOpenAI_key: Stores the API key for authenticating with Azure OpenAI

AzureOpenAI_endpoint: Stores the endpoint URL for accessing Azure OpenAI services

**Adding Environment Variables to the Web Application**

```csharp
webApp
    .WithReference(openAI)
    .WithEnvironment("AI__OPENAI__CHATMODEL", chatModelName)
    .WithEnvironment("ConnectionStrings:openai:Endpoint", AzureOpenAI_endpoint)
    .WithEnvironment("ConnectionStrings:openai:Key", AzureOpenAI_key);
```

WithReference(openAI): Links the openAI resource configuration to the web application

WithEnvironment: Sets environment variables for the application:

AI__OPENAI__CHATMODEL: Specifies the chat model to use (e.g., "gpt-4o")

ConnectionStrings:openai:Endpoint: Configures the endpoint URL for Azure OpenAI services

ConnectionStrings:openai:Key: Configures the API key for Azure OpenAI services

We can review the whole Program.cs code: 

```csharp
using Microsoft.Extensions.Configuration;

var builder = DistributedApplication.CreateBuilder(args);

var postgres = builder.AddPostgres("postgres")
    .WithImage("ankane/pgvector")
    .WithImageTag("latest")
    .WithLifetime(ContainerLifetime.Persistent);

var catalogDb = postgres.AddDatabase("catalogdb");
var identityDb = postgres.AddDatabase("IdentityDB");

var launchProfileName = ShouldUseHttpForEndpoints() ? "http" : "https";

var identityApi = builder.AddProject<Projects.Idintity_API>("identity-api", launchProfileName)
    .WithExternalHttpEndpoints()
    .WithReference(identityDb);

var identityEndpoint = identityApi.GetEndpoint(launchProfileName);

var catalogApi = builder.AddProject<Projects.Catalog_API>("catalog-api")
    .WithReference(catalogDb);

var webApp = builder.AddProject<Projects.WebApp>("webapp", launchProfileName)
    .WithExternalHttpEndpoints()
    .WithReference(catalogApi)
    .WithEnvironment("IdentityUrl", identityEndpoint);

const string openAIName = "openai";
const string chatModelName = "gpt-4o";

IResourceBuilder<IResourceWithConnectionString> openAI;
openAI = builder.AddConnectionString(openAIName);

var AzureOpenAI_key = builder.Configuration["ConnectionStrings:openai:Key"];
var AzureOpenAI_endpoint = builder.Configuration["ConnectionStrings:openai:Endpoint"];

webApp
    .WithReference(openAI)
    .WithEnvironment("AI__OPENAI__CHATMODEL", chatModelName)
    .WithEnvironment("ConnectionStrings:openai:Endpoint", AzureOpenAI_endpoint)
    .WithEnvironment("ConnectionStrings:openai:Key", AzureOpenAI_key);


webApp.WithEnvironment("CallBackUrl", webApp.GetEndpoint(launchProfileName));

// Identity has a reference to all of the apps for callback urls, this is a cyclic reference
identityApi.WithEnvironment("WebAppClient", webApp.GetEndpoint(launchProfileName));

builder.Build().Run();
static bool ShouldUseHttpForEndpoints()
{
    const string EnvVarName = "ESHOP_USE_HTTP_ENDPOINTS";
    var envValue = Environment.GetEnvironmentVariable(EnvVarName);

    // Attempt to parse the environment variable value; return true if it's exactly "1".
    return int.TryParse(envValue, out int result) && result == 1;
}
```

## 4. We add the AI ConnectionString in the appsettings.json (AppHost project)

We also have to add the AI Service Connection String in the **appsettings.json** file

and also we define the AI Model

**appsettings.json**

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Aspire.Hosting.Dcp": "Warning"
    }
  },
  "ConnectionStrings": {
    "openai": "Endpoint=https://myaiserviceluiscoco.openai.azure.com/;Key=XXXXXXXXXXXXXXXXXXXXXXXXXXX"
  },
  "AI": {
    "OpenAI": {
      "ChatModel": "gpt-4o"
    }
  }
}
```

## 5. We modify the WebApp middleware

To register the AI Service in the WebApp project we have to include the following code:

```csharp
builder.Services.AddSingleton<IChatClient>(serviceProvider =>
{
    var configuration = serviceProvider.GetRequiredService<IConfiguration>();

    // Read the OpenAI connection string
    var openAIConnectionString = configuration.GetConnectionString("openai");
    if (string.IsNullOrWhiteSpace(openAIConnectionString))
        throw new InvalidOperationException("Connection string 'openai' is missing.");

    // Parse the connection string
    var endpointValue = openAIConnectionString.Split(';').FirstOrDefault(s => s.StartsWith("Endpoint="))?.Replace("Endpoint=", "");
    var apiKeyValue = openAIConnectionString.Split(';').FirstOrDefault(s => s.StartsWith("Key="))?.Replace("Key=", "");

    // Read environment variable for OpenAI Chat Model
    var deploymentName = configuration["AI:OpenAI:ChatModel"];
    if (string.IsNullOrWhiteSpace(deploymentName))
        throw new InvalidOperationException("Configuration key 'AI:OpenAI:ChatModel' is missing.");

    // Validate connection string values
    if (string.IsNullOrWhiteSpace(endpointValue))
        throw new InvalidOperationException("The 'Endpoint' value in the OpenAI connection string is missing.");
    if (string.IsNullOrWhiteSpace(apiKeyValue))
        throw new InvalidOperationException("The 'Key' value in the OpenAI connection string is missing.");

    Console.WriteLine($"AI Chat Model: {deploymentName}");
    Console.WriteLine($"OpenAI Endpoint: {endpointValue}");
    Console.WriteLine($"OpenAI API Key: {apiKeyValue}");

    // Create AzureKeyCredential and OpenAI client
    if (!Uri.TryCreate(endpointValue, UriKind.Absolute, out var endpoint))
        throw new InvalidOperationException($"Azure OpenAI Endpoint '{endpointValue}' is not a valid URI.");

    var credentials = new AzureKeyCredential(apiKeyValue);
    var client = new AzureOpenAIClient(endpoint, credentials).AsChatClient(deploymentName);

    return new ChatClientBuilder(client)
        .UseFunctionInvocation()
        .Build();
});
```

We can also review the whole Program.cs file:

```csharp
using WebApp.Components;
using eShop.WebApp.Services;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Components.Authorization;
using Microsoft.AspNetCore.Components.Server;
using Microsoft.IdentityModel.JsonWebTokens;
using Azure.AI.OpenAI;
using Azure;
using Microsoft.Extensions.AI;

var builder = WebApplication.CreateBuilder(args);

builder.AddServiceDefaults();

// Add services to the container.
builder.Services.AddRazorComponents()
.AddInteractiveServerComponents();

// Register the chat client for Azure OpenAI
builder.Services.AddSingleton<IChatClient>(serviceProvider =>
{
    var configuration = serviceProvider.GetRequiredService<IConfiguration>();

    // Read the OpenAI connection string
    var openAIConnectionString = configuration.GetConnectionString("openai");
    if (string.IsNullOrWhiteSpace(openAIConnectionString))
        throw new InvalidOperationException("Connection string 'openai' is missing.");

    // Parse the connection string
    var endpointValue = openAIConnectionString.Split(';').FirstOrDefault(s => s.StartsWith("Endpoint="))?.Replace("Endpoint=", "");
    var apiKeyValue = openAIConnectionString.Split(';').FirstOrDefault(s => s.StartsWith("Key="))?.Replace("Key=", "");

    // Read environment variable for OpenAI Chat Model
    var deploymentName = configuration["AI:OpenAI:ChatModel"];
    if (string.IsNullOrWhiteSpace(deploymentName))
        throw new InvalidOperationException("Configuration key 'AI:OpenAI:ChatModel' is missing.");

    // Validate connection string values
    if (string.IsNullOrWhiteSpace(endpointValue))
        throw new InvalidOperationException("The 'Endpoint' value in the OpenAI connection string is missing.");
    if (string.IsNullOrWhiteSpace(apiKeyValue))
        throw new InvalidOperationException("The 'Key' value in the OpenAI connection string is missing.");

    Console.WriteLine($"AI Chat Model: {deploymentName}");
    Console.WriteLine($"OpenAI Endpoint: {endpointValue}");
    Console.WriteLine($"OpenAI API Key: {apiKeyValue}");

    // Create AzureKeyCredential and OpenAI client
    if (!Uri.TryCreate(endpointValue, UriKind.Absolute, out var endpoint))
        throw new InvalidOperationException($"Azure OpenAI Endpoint '{endpointValue}' is not a valid URI.");

    var credentials = new AzureKeyCredential(apiKeyValue);
    var client = new AzureOpenAIClient(endpoint, credentials).AsChatClient(deploymentName);

    return new ChatClientBuilder(client)
        .UseFunctionInvocation()
        .Build();
});


builder.AddApplicationServices();

//builder.AddAuthenticationServices();

//builder.Services.AddHttpForwarderWithServiceDiscovery();

//builder.Services.AddSingleton<IProductImageUrlProvider, ProductImageUrlProvider>();
//builder.Services.AddHttpClient<CatalogService>(o => o.BaseAddress = new("http://localhost:5301"))
//    .AddApiVersion(1.0);

var app = builder.Build();

app.MapDefaultEndpoints();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error", createScopeForErrors: true);
    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
}

app.UseHttpsRedirection();

app.UseAntiforgery();

app.MapStaticAssets();
app.MapRazorComponents<App>()
    .AddInteractiveServerRenderMode();

app.MapForwarder("/product-images/{id}", "http://localhost:5301", "/api/catalog/items/{id}/pic");

app.Run();
```

## 6. We remove the AI connection string from appsettings.json (WebApp project)

We remove the AI ConnectionString from the **appsettings.json** file

**appsettings.json**

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

## 7. We run the Application and verify the results


