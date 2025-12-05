---
date: '2025-12-01T03:00:00+05:30'
title: '📨 Pub Sub'
summary: 'Publish / Subscribe'
tags: ['.NET Aspire', 'DAPR', 'Pub-Sub', 'Redis', 'RabbitMQ']
ShowToc: true
weight: 3
---

This article shows how to use Pub-Sub (Publish / Subscribe) with `DAPR`'s building block, combined with `.NET Aspire` for orchestration.

You'll learn how services can communicate asynchronously through events in a distributed environment, with `Aspire` managing the setup and `DAPR` providing the abstraction for message brokers.

```mermaid
flowchart LR
    ServiceA["Service A (Publisher)"]
    subgraph DAPR["DAPR Sidecars"]
        SidecarA["Sidecar A"]
        SidecarB["Sidecar B"]
    end
    Broker["Message Broker<br/>(Redis / RabbitMQ)"]
    ServiceB["Service B (Subscriber)"]
    
    ServiceA -->|"Publish event"| SidecarA
    SidecarA -->|"Store event"| Broker
    Broker -->|"Deliver event"| SidecarB
    SidecarB -->|"Invoke subscriber"| ServiceB
```

![Pub / Sub](https://i.ibb.co/70ksSGH/pub-sub-menu.jpg)

---

## Resources

### 👩‍💻 Source Code

If you are stuck, you can refer the final source code, available at [GitHub Repository](https://github.com/NetRecipes/pub-sub)

## What is Pub-Sub, and Why It Matters?

Pub-Sub (Publish / Subscribe) is a messaging pattern where services communicate through events without direct coupling between publishers and subscribers.

In distributed systems, services often need to broadcast information to multiple interested parties or react to events from other services. Without proper pub-sub patterns, services become tightly coupled, harder to scale, and difficult to extend with new functionality.

## Synchronous vs Asynchronous Communication

In distributed systems, services can communicate in two fundamental ways:

| Aspect | Synchronous Communication | Asynchronous Communication |
|--------|--------------------------|----------------------------|
| **Pattern** | Request-Response | Fire-and-Forget / Event-Driven |
| **Waiting** | Caller waits for response | Caller continues immediately |
| **Coupling** | Tight coupling — caller knows callee | Loose coupling — via message broker |
| **Use Case** | Direct queries, immediate results needed | Background tasks, event notifications |
| **Example** | Get user profile, calculate price | Send email, process order, update analytics |
| **DAPR Building Block** | **Service Invocation** ([see article](/courses/dapr-aspire/service-invocation/)) | **Pub/Sub** ([this article](#)) |
| **Failure Handling** | Immediate error response | Retry via message queue |

**This article focuses on asynchronous `pub / sub` communication** using `DAPR`'s `Pub/Sub` building block.  
For example, think of an e-commerce system where ServiceA creates an order. With pub-sub:

- ServiceA publishes an "order-created" event and continues
- ServiceB (inventory) receives the event and reserves stock
- ServiceC (email) receives the same event and sends confirmation
- ServiceD (analytics) receives the event and updates dashboards

> 💡 **Note:** For synchronous communication patterns where services directly invoke and awaits a synchronous response, refer to the [📞 Service Invocation article](/courses/dapr-aspire/service-invocation/).

## Hands-On Setup

We'll scaffold a new .NET Aspire solution and add two Web API services that communicate through events.

Each command below is shown individually with its purpose explained.

### 1. Create the Aspire host project

This sets up the orchestration project named `PubSub`.

```bash
dotnet new aspire --name PubSub --no-https --output .
```

### 2. Create the 2 Web API services

Generates 2 Web API projects called `ServiceA` (publisher) & `ServiceB` (subscriber) using controllers.

And add both services to the solution:

```bash
dotnet new webapi --name ServiceA --no-https --use-controllers
dotnet new webapi --name ServiceB --no-https --use-controllers
dotnet sln add .\ServiceA\ .\ServiceB\
```

### 3. Migrate to the new `.slnx` format (Optional)

Converts the solution to the modern format used by Aspire.

And cleans up the legacy solution file, leaving only `PubSub.slnx`.

```bash
dotnet sln migrate
rm PubSub.sln
```

> 💡 **Note:** The newer `.slnx` solution format is a general .NET enhancement. It's cleaner and more minimal than the traditional `.sln`, reducing boilerplate and making solutions easier to manage in modern .NET projects.

Now, open `PubSub.slnx` with Visual Studio or Rider, or simply open the directory with VS Code.

Alternatively, you can skip these steps and clone the final companion repository: [NetRecipes/pub-sub](https://github.com/NetRecipes/pub-sub).

## NuGet Packages

Depending on your IDE, install the following NuGet packages in the specified projects:

### ServiceA and ServiceB

In both `ServiceA` and `ServiceB` projects, install the following NuGet packages to enable DAPR integration, API documentation, and UI enhancements.

(Note: `Microsoft.AspNetCore.OpenApi` is usually included by default in the Web API template, so you may not need to install it separately.)

| Package ID | Purpose |
|------------|---------|
| [Dapr.AspNetCore](https://www.nuget.org/packages/Dapr.AspNetCore) | Integrates DAPR, including pub-sub, state management, and bindings for ASP.NET Core services. |
| [Swashbuckle.AspNetCore.SwaggerUI](https://www.nuget.org/packages/Swashbuckle.AspNetCore.SwaggerUI) | Provides Swagger UI for API documentation and testing. |
| [AspNetCore.SwaggerUI.Themes](https://www.nuget.org/packages/AspNetCore.SwaggerUI.Themes) | Adds modern themes to Swagger UI for better visual experience. |

```bash
dotnet add ServiceA package Dapr.AspNetCore
dotnet add ServiceA package Swashbuckle.AspNetCore
dotnet add ServiceA package Swashbuckle.AspNetCore.SwaggerUI
dotnet add ServiceA package AspNetCore.SwaggerUI.Themes

dotnet add ServiceB package Dapr.AspNetCore
dotnet add ServiceB package Swashbuckle.AspNetCore
dotnet add ServiceB package Swashbuckle.AspNetCore.SwaggerUI
dotnet add ServiceB package AspNetCore.SwaggerUI.Themes
```

### PubSub.AppHost

In the `PubSub.AppHost` project, install the following NuGet packages to enable integration with DAPR sidecars and Redis for pub-sub messaging.

| Package ID | Purpose |
|------------|---------|
| [CommunityToolkit.Aspire.Hosting.Dapr](https://www.nuget.org/packages/CommunityToolkit.Aspire.Hosting.Dapr) | Integrates DAPR sidecars into an Aspire application, enabling service orchestration with DAPR building blocks. |
| [Aspire.Hosting.Redis](https://www.nuget.org/packages/Aspire.Hosting.Redis) | Adds Redis container support to Aspire hosting, allowing you to use Redis Streams as a pub-sub broker. |
| [Aspire.Hosting.RabbitMQ](https://www.nuget.org/packages/Aspire.Hosting.RabbitMQ) | Adds RabbitMQ container support to Aspire hosting (alternative message broker). |

```bash
dotnet add PubSub.AppHost package CommunityToolkit.Aspire.Hosting.Dapr
dotnet add PubSub.AppHost package Aspire.Hosting.Redis
dotnet add PubSub.AppHost package Aspire.Hosting.RabbitMQ
```

## Code Walkthrough

With the setup complete, let's implement pub-sub messaging between `ServiceA` and `ServiceB`.

### 1. Program.cs – Add DAPR support (Both Services)

In both `ServiceA/Program.cs` and `ServiceB/Program.cs`, register DAPR services and Swagger UI:

```csharp
using AspNetCore.Swagger.Themes;

var builder = WebApplication.CreateBuilder(args);

builder.AddServiceDefaults(); // Aspire hosting helpers
builder.Services.AddDaprClient(); // Enables DAPR integration
builder.Services.AddControllers().AddDapr(); // Adds DAPR support to controllers
builder.Services.AddOpenApi();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.UseSwaggerUI(
        Theme.Futuristic,
        options => options.SwaggerEndpoint("/openapi/v1.json", "ServiceA v1")); // Change to ServiceB for ServiceB
}

app.UseAuthorization();
app.UseCloudEvents(); // Required for pub-sub
app.MapControllers();
app.MapSubscribeHandler(); // Required for DAPR to discover subscriptions

app.Run();
```

Also create a Common class library project that can be referenced by both `ServiceA` and `ServiceB` to contain a common event model:

```csharp
public record OrderCreatedEvent(string OrderId, string Product, int Quantity, decimal TotalPrice);
```

> It's fine to skip the Common project and just have the record in both projects as well, for simplicity.

### 2. PublisherController.cs – Publish events from ServiceA

Create a controller `Controllers/PublisherController.cs` in `ServiceA` that will publish order events.

```csharp
[HttpPost("publish")]
public async Task<IActionResult> Publish([FromBody] Order order)
{
    logger.LogInformation("Publishing {Order}", order);
    await daprClient.PublishEventAsync(
        "pubsub",
        "order",
        order);
    
    // continue with what you want to do

    return Ok();
}
```

Key points about publishing:

- `"pubsub"` is the component name defined in DAPR configuration
- `"order-created"` is the topic — subscribers listen to specific topics
- The event is published and the method returns immediately
- No knowledge of subscribers is needed

### 3. SubscriberController.cs – Subscribe to events in ServiceB

Create a controller `Controllers/SubscriberController.cs` in `ServiceB` that will subscribe to order events.

```csharp
[HttpPost("subscribe")]
[Topic("pubsub", "order")] // This is the key to subscribe
public async Task<IActionResult> Subscribe([FromBody] Order order)
{
    logger.LogInformation("Received {Order}", order);
    return Ok();
}
```

Key points about subscribing:

- The `[Topic]` attribute tells DAPR which component and topic to subscribe to
- Multiple services can subscribe to the same topic
- Subscribers process events independently and asynchronously
- If processing fails, `DAPR` can retry based on configuration

### 4. Configure Pub-Sub Component (Redis)

Create a `components` folder at the solution root and add a `pubsub.yaml` file:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: pubsub
spec:
  type: pubsub.redis
  version: v1
  metadata:
    - name: redisHost
      value: localhost:6500
    - name: redisPassword
      value: "localDev"
```

This configures Redis Streams as the message broker.

> 💡 **Note:** Redis Streams is different from Redis Pub/Sub. Streams provide message persistence, consumer groups, and delivery guarantees — features needed for reliable distributed messaging.

### 5. Configure Services in AppHost

In `PubSub.AppHost/Program.cs`, configure both services with DAPR sidecars and Redis:

```csharp
using CommunityToolkit.Aspire.Hosting.Dapr;

var builder = DistributedApplication.CreateBuilder(args);

var redisPassword = builder.AddParameter("RedisPassword", true);
var redisPort = builder.AddParameter("RedisPort");
var redisPortValue = await redisPort.Resource.GetValueAsync(CancellationToken.None);

var redis = builder
    .AddRedis(
        "redis",
        int.Parse(redisPortValue!),
        redisPassword)
    .WithLifetime(ContainerLifetime.Persistent)
    .WithRedisInsight();

var servicea = builder
    .AddProject<Projects.ServiceA>("servicea")
    .WaitFor(redis)
    .WithDaprSidecar(new DaprSidecarOptions
    {
        ResourcesPaths = [Path.Combine("..", "components")]
    });

var serviceb = builder
    .AddProject<Projects.ServiceB>("serviceb")
    .WaitFor(redis)
    .WithDaprSidecar(new DaprSidecarOptions
    {
        ResourcesPaths = [Path.Combine("..", "components")]
    });

builder.Build().Run();

```

To pass the Redis credentials, update the `appsettings.Development.json` file in the `PubSub.AppHost` project:

```json
{
  "Logging": {
    ...
  },
  "Parameters": {
    "RedisPort": "6500",
    "RedisPassword": "localDev"
  }
}
```

## Running the Application

Now, when you run your setup, you should see the Aspire dashboard with both services and Redis.

![Aspire Dashboard](https://i.ibb.co/Ggy0mnL/pub-sub-aspire-dashboard.png)

### Testing the Pub-Sub Pattern

1. Visit `ServiceA`'s Swagger endpoint, with `/swagger` at the end of the URL
2. Call the `/api/order/create` endpoint with a request body:  

    **Request**

    ```json
    {
        "product": "Banana",
        "quantity": 12,
        "pricePerUnit": 2.99
    }
    ```

    ![Service Call](https://i.ibb.co/W48j8KLw/pub-sub-service-call.png)

3. You'll receive a `202 OK` response immediately  
4. Check the logs for `ServiceB` — you'll see it received and processed the event

    ![Redis Insight](https://i.ibb.co/VWPCRKTq/pub-sub-redis-insight.png)

#### *Structured Logging*

![Structured Logging](https://i.ibb.co/TBh4nxcN/pub-sub-structured-logs.png)

#### *Distributed Tracing*

![Distributed Tracing](https://i.ibb.co/wZT3J66p/pub-sub-distributed-tracing.png)

### What's Happening Behind the Scenes

When you publish an event:

1. `ServiceA` publishes the event to its DAPR sidecar
2. DAPR forwards the event to Redis Streams
3. Redis persists the event and notifies subscribers
4. DAPR sidecar for `ServiceB` receives the event
5. DAPR invokes the subscribed endpoint in `ServiceB`
6. `ServiceB` processes the event asynchronously

```mermaid
sequenceDiagram
    participant ServiceA
    participant SidecarA as Dapr Sidecar (ServiceA)
    participant Redis as Redis Streams
    participant SidecarB as Dapr Sidecar (ServiceB)
    participant ServiceB
    
    ServiceA->>SidecarA: Publish "order-created" event
    SidecarA->>Redis: Store event in stream
    Redis-->>SidecarA: Acknowledge
    SidecarA-->>ServiceA: Return 202 Accepted
    
    Redis->>SidecarB: Deliver event (push or pull)
    SidecarB->>ServiceB: POST /api/ordersubscriber/order-created
    ServiceB-->>SidecarB: Return 200 OK
    SidecarB-->>Redis: Acknowledge processing
```

All of this happens with guaranteed delivery, automatic retries, and message persistence.

## Pub-Sub Benefits

By using `DAPR` for pub-sub, you get several advantages:

| Benefits | Description |
| --- | --- |
| `Loose Coupling` | Publishers don't know about subscribers and vice versa. You can add new subscribers without modifying publishers. This makes the system easier to extend and maintain. |
| `Multiple Subscribers` | Many services can react to the same event. One order event can trigger inventory updates, email notifications, analytics, and more — all independently. |
| `Resilience` | Built-in retries, dead letter queues, and message persistence. If a subscriber fails, `DAPR` retries automatically based on configuration. |
| `Scalability` | Subscribers can scale independently. If email sending is slow, scale up the email service without affecting inventory updates. |
| `Simplified Testing` | No special emulators specific to Redis or RabbitMQ required for performing Integration testing. Since they're endpoints by end of the day, API integration testing is good enough. |
| `Flexibility` | Switch between any `DAPR` supported [Pub / Sub brokers](https://docs.dapr.io/reference/components-reference/supported-pubsub/) without changing service code. `DAPR` abstracts the messaging layer. |

## Switching to RabbitMQ

Want to see the power of abstraction? Stop the debugger and replace the contents of `components/pubsub.yaml`:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: pubsub
spec:
  type: pubsub.rabbitmq
  version: v1
  metadata:
    - name: host
      value: "amqp://guest:guest@localhost:5672"
```

Update `PubSub.AppHost/Program.cs` to use RabbitMQ instead of Redis:

```csharp
using CommunityToolkit.Aspire.Hosting.Dapr;

var builder = DistributedApplication.CreateBuilder(args);

var rabbitmqUsername = builder.AddParameter("RabbitMQUsername");
var rabbitmqPassword = builder.AddParameter("RabbitMQPassword", true);
var rabbitMQPort = builder.AddParameter("RabbitMQPort");
var rabbitMQPortValue = await rabbitMQPort.Resource.GetValueAsync(CancellationToken.None);

var rabbitmq = builder
    .AddRabbitMQ(
        "rabbitmq",
        rabbitmqUsername,
        rabbitmqPassword,
        int.Parse(rabbitMQPortValue!))
    .WithLifetime(ContainerLifetime.Persistent)
    .WithManagementPlugin();

var servicea = builder
    .AddProject<Projects.ServiceA>("servicea")
    WaitFor(rabbitmq)
    .WithDaprSidecar(new DaprSidecarOptions
    {
        ResourcesPaths = [Path.Combine("..", "components")]
    });

var serviceb = builder
    .AddProject<Projects.ServiceB>("serviceb")
    WaitFor(rabbitmq)
    .WithDaprSidecar(new DaprSidecarOptions
    {
        ResourcesPaths = [Path.Combine("..", "components")]
    });

builder.Build().Run();

```

Now run the application again. Everything works exactly as before, but with RabbitMQ as the message broker instead of Redis Streams.

![RabbitMQ Management Plugin](https://i.ibb.co/QFzkmcs6/pub-sub-management-plugin.png)

No service code changes. No NuGet package changes. Just configuration.

This demonstrates `DAPR`'s powerful abstraction layer — switch to any supported DAPR [Pub-Sub Broker](https://docs.dapr.io/reference/components-reference/supported-pubsub/), without changing implementation logic or boilerplate.

## Advanced Patterns

### Multiple Topics

You can publish and subscribe to multiple topics for different event types:

```csharp
// Publish different event types
await daprClient.PublishEventAsync("pubsub", "order-created", orderEvent);
await daprClient.PublishEventAsync("pubsub", "order-cancelled", cancelEvent);
await daprClient.PublishEventAsync("pubsub", "inventory-updated", inventoryEvent);
```

```csharp
// Subscribe to multiple topics in the same service
[Topic("pubsub", "order-created")]
[HttpPost("order-created")]
public IActionResult HandleOrderCreated([FromBody] OrderCreatedEvent order) { ... }

[Topic("pubsub", "order-cancelled")]
[HttpPost("order-cancelled")]
public IActionResult HandleOrderCancelled([FromBody] OrderCancelledEvent order) { ... }
```

### Multiple Subscribers to Same Topic

Multiple services can subscribe to the same topic and each will receive a copy of the event:

```csharp
// ServiceB - Inventory Management
[Topic("pubsub", "order-created")]
public IActionResult UpdateInventory(...) { ... }

// ServiceC - Email Notifications
[Topic("pubsub", "order-created")]
public IActionResult SendEmail(...) { ... }

// ServiceD - Analytics
[Topic("pubsub", "order-created")]
public IActionResult RecordAnalytics(...) { ... }
```

Each service processes the event independently. If one fails, others continue unaffected.

## Summary

With `DAPR` and `Aspire`, you can build resilient, event-driven systems with minimal boilerplate.

Services communicate through events without knowing about each other, making systems easier to extend and maintain. Whether you start with Redis Streams or switch to `RabbitMQ`, `Kafka`, or `Azure Service Bus`, the service logic stays the same — thanks to `DAPR`'s abstraction and `Aspire`'s orchestration.

Your services react to events, not direct calls, making them more scalable and cloud-ready from day one.
