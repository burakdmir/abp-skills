---
name: abp-ai
description: "ABP Framework v10.x (10.4–10.6) AI integration: Microsoft.Extensions.AI (IChatClient), Semantic Kernel, Microsoft Agent Framework, AI Workspaces, AI Management module (Pro) with RAG, document indexing/chunking and MCP servers. Use when you need chat clients, AI agents, RAG or AI workspace management in ABP."
---

# ABP Framework — AI Integration

Guide to ABP Framework v10.x (10.4–10.6) AI integration. Microsoft.Extensions.AI, Semantic Kernel, Microsoft Agent Framework, AI Workspaces, AI Management module (Pro), RAG with document indexing and chunking.

## Trigger

- "ABP AI"
- "ABP chat client"
- "ABP IChatClient"
- "ABP AI workspace"
- "ABP Semantic Kernel"
- "ABP Agent Framework"
- "ABP AI Management"
- "ABP RAG"
- "ABP document indexing"
- "ABP MCP server"
- "ABP OpenAI"
- "ABP Ollama"

---

## Overview

ABP integrates AI via Microsoft's AI libraries around one core concept: the **AI Workspace** — an isolated, named AI configuration scope. You configure workspaces once, then resolve AI services per workspace.

```bash
abp add-package Volo.Abp.AI
```

The `Volo.Abp.AI` package integrates with three libraries:

| Library | When to use |
|---|---|
| **Microsoft.Extensions.AI** | Library/module developers — minimal, simple abstractions (`IChatClient`) |
| **Microsoft Agent Framework** (`Microsoft.Agents.AI`) | **Recommended for applications** — successor of AutoGen + Semantic Kernel; single/multi-agent patterns, thread-based state, filters, telemetry |
| **Microsoft.SemanticKernel** | Only if you need its specific AI integration features |

---

## Microsoft.Extensions.AI

### Resolving IChatClient

```csharp
public class MyService
{
    private readonly IChatClient _chatClient;
    public MyService(IChatClient chatClient) => _chatClient = chatClient;

    public async Task<string> GetResponseAsync(string prompt)
    {
        return await _chatClient.GetResponseAsync(prompt);
    }
}
```

### IChatClientAccessor (optional AI)

Use the accessor when AI is **optional** (e.g., a reusable module that may or may not have AI configured):

```csharp
public class MyService
{
    private readonly IChatClientAccessor _chatClientAccessor;
    public MyService(IChatClientAccessor chatClientAccessor) => _chatClientAccessor = chatClientAccessor;

    public async Task<string> GetResponseAsync(string prompt)
    {
        var chatClient = _chatClientAccessor.ChatClient;
        if (chatClient is null)
        {
            return "No chat client configured";
        }
        return await chatClient.GetResponseAsync(prompt);
    }
}
```

### Workspaces

A workspace is a class decorated with `WorkspaceNameAttribute`:

```csharp
using Volo.Abp.AI;

[WorkspaceName("CommentSummarization")]
public class CommentSummarization
{
}
```

Rules:
- Names must be **unique**, **case-sensitive**, and **cannot contain spaces** (use underscores or camelCase).
- Without the attribute, the class full name is used as the workspace name.

Resolve a typed (workspace-scoped) chat client:

```csharp
public class MyService
{
    private readonly IChatClient<CommentSummarization> _chatClient;
    public MyService(IChatClient<CommentSummarization> chatClient) => _chatClient = chatClient;

    public async Task<string> GetResponseAsync(string prompt)
    {
        return await _chatClient.GetResponseAsync(prompt);
    }
}
```

**Fallback:** if a chat client is not configured for the specified workspace, the **default workspace's** chat client is returned. `IChatClientAccessor<TWorkSpace>.ChatClient` is `null` only when neither the workspace nor the default is configured — always null-check the accessor.

### Configuration (AbpAIWorkspaceOptions)

Workspaces are configured with `AbpAIWorkspaceOptions` in **`PreConfigureServices`** — it must run before services are registered. You need a provider package such as `Microsoft.Extensions.AI.OpenAI` or `OllamaSharp`.

```csharp
[DependsOn(typeof(AbpAIModule))]
public class MyProjectModule : AbpModule
{
    public override void PreConfigureServices(ServiceConfigurationContext context)
    {
        PreConfigure<AbpAIWorkspaceOptions>(options =>
        {
            // Default workspace
            options.Workspaces.ConfigureDefault(configuration =>
            {
                configuration.ConfigureChatClient(chatClientConfiguration =>
                {
                    chatClientConfiguration.Builder = new ChatClientBuilder(
                        sp => new OllamaApiClient("http://localhost:11434", "mistral")
                    );
                });
            });

            // Isolated workspace
            options.Workspaces.Configure<CommentSummarization>(configuration =>
            {
                configuration.ConfigureChatClient(chatClientConfiguration =>
                {
                    chatClientConfiguration.Builder = new ChatClientBuilder(
                        sp => new OllamaApiClient("http://localhost:11434", "mistral")
                    );
                });
            });
        });
    }
}
```

- `Builder` is set once and builds the `IChatClient` instance.
- `BuilderConfigurers` is a list of actions applied to the builder for incremental changes, executed in order.

---

## Microsoft Agent Framework (Microsoft.Agents.AI)

Works **on top of `IChatClient`** — the workspace configuration is identical to Microsoft.Extensions.AI. Create agents with the `CreateAIAgent` extension method:

```csharp
using Microsoft.Agents.AI;
using Microsoft.Extensions.AI;

public class MyService
{
    private readonly IChatClient _chatClient;
    public MyService(IChatClient chatClient) => _chatClient = chatClient;

    public async Task<string> GetResponseAsync(string userMessage)
    {
        AIAgent agent = _chatClient.CreateAIAgent(
            instructions: "You are a helpful assistant that provides concise answers."
        );

        AgentRunResponse response = await agent.RunAsync(userMessage);
        return response.Text;
    }
}
```

Workspace-scoped agents work the same way — inject `IChatClient<CustomerSupport>` and call `CreateAIAgent` on it. For optional AI, use `IChatClientAccessor` / `IChatClientAccessor<TWorkSpace>` and null-check before `CreateAIAgent`.

---

## Semantic Kernel

Resolve `IKernelAccessor` (or `IKernelAccessor<TWorkSpace>` for a workspace). The `Kernel` may be `null` if not configured — always check:

```csharp
public class MyService
{
    private readonly IKernelAccessor _kernelAccessor;
    public MyService(IKernelAccessor kernelAccessor) => _kernelAccessor = kernelAccessor;

    public async Task<string> GetResponseAsync(string prompt)
    {
        var kernel = _kernelAccessor.Kernel;
        if (kernel is null)
        {
            return "No kernel configured";
        }
        return await kernel.InvokeAsync(prompt);
    }
}
```

Configuration mirrors the chat client, but uses `ConfigureKernel` (requires a connector package, e.g. `Microsoft.SemanticKernel.Connectors.AzureOpenAI`):

```csharp
PreConfigure<AbpAIWorkspaceOptions>(options =>
{
    options.Workspaces.ConfigureDefault(configuration =>
    {
        configuration.ConfigureKernel(kernelConfiguration =>
        {
            kernelConfiguration.Builder = Kernel.CreateBuilder()
                .AddAzureOpenAIChatClient("...", "...");
        });
    });
});
```

Note: unlike the typed chat client, a workspace-scoped kernel does **not** fall back to the default — `IKernelAccessor<TWorkSpace>.Kernel` is `null` if that workspace has no kernel configured.

---

## AI Management Module (Pro — Team license or higher)

The **AI Management** module (commercial) manages workspaces **dynamically** — database-persisted, with admin UI (MVC, Angular, Blazorise, MudBlazor, React Admin Console) and API endpoints — on top of the framework's AI Workspaces.

```bash
abp add-module Volo.AIManagement
```

**At least one provider package is required:**

```bash
abp add-package Volo.AIManagement.OpenAI    # OpenAI / Azure OpenAI-compatible
abp add-package Volo.AIManagement.Ollama    # Local models
```

### System vs Dynamic Workspaces

| | System | Dynamic |
|---|---|---|
| Defined | In code via `PreConfigure<AbpAIWorkspaceOptions>` | UI, or `ApplicationWorkspaceManager` + `IWorkspaceRepository` |
| Deletable in UI | No | Yes |
| Config source | Code-defined client by default; DB settings only when `OverrideSystemConfiguration = true` | Database |
| Synchronized | Automatically at startup | — |

Programmatic creation (data seeding):

```csharp
public class WorkspaceDataSeederContributor : IDataSeedContributor, ITransientDependency
{
    // Inject: IConfiguration, IWorkspaceRepository, ApplicationWorkspaceManager

    public async Task SeedAsync(DataSeedContext context)
    {
        var workspace = await _applicationWorkspaceManager.CreateAsync(
            name: "CustomerSupportWorkspace",
            provider: "OpenAI",
            modelName: "gpt-4");

        workspace.ApiKey = _configuration["AI:OpenAI:ApiKey"];
        workspace.SystemPrompt = "You are a helpful customer support assistant.";

        await _workspaceRepository.InsertAsync(workspace);
    }
}
```

**Security:** API keys, embedding keys, vector-store connection settings and MCP credentials are sensitive persisted configuration — never hard-code them in source control; supply from a secure configuration provider and restrict admin permissions. Duplicating a workspace also duplicates its credentials.

**Resolution precedence:** no persisted config / inactive / system workspace without override → code-defined keyed `IChatClient`, then default `IChatClient`. Active dynamic (or overridden system) workspace → the factory registered for its persisted `Provider` value.

### Remote Client (Microservice) Usage

Use `Volo.AIManagement.Client.HttpApi.Client` to consume a central AI Management service:

```csharp
// Inject IChatCompletionClientAppService
var request = new ChatClientCompletionRequestDto
{
    Messages = [new ChatMessageDto { Role = ChatRole.User, Content = prompt }]
};
var response = await _chatService.ChatCompletionsAsync(workspaceName, request);
return response.Text;
```

Streaming: `await foreach (var update in _chatService.StreamChatCompletionsAsync(workspaceName, request))`.

### OpenAI-Compatible API (Pro)

The module exposes an OpenAI-compatible REST API at `/v1` (`/v1/chat/completions`, `/v1/completions`, `/v1/models`, `/v1/embeddings`), so tools like AnythingLLM, Open WebUI or the OpenAI SDK can connect directly. The `model` value is a **workspace name**; all endpoints require an authenticated Bearer token; `GET /v1/models` returns only workspaces the user may consume.

### MCP Servers (Pro)

Manage external MCP servers as tools for workspaces. Transports: **Stdio**, **SSE**, **StreamableHttp**; HTTP auth: None / API Key / Bearer / Custom header. Tune timeouts:

```csharp
Configure<McpClientFactoryOptions>(options =>
{
    options.DefaultTimeoutMs = 180_000;              // default 120s
    options.StdioInitializationTimeoutMs = 240_000;
    options.StdioConnectionTestTimeoutSeconds = 300; // default 180s, clamped 30–600
});
```

### Custom Provider Factory (Pro)

One registered `IChatClientFactory` per provider name:

```csharp
public class AzureOpenAIChatClientFactory : IChatClientFactory, ITransientDependency
{
    public string Provider => "AzureOpenAI";

    public Task<IChatClient> CreateAsync(ChatClientCreationConfiguration configuration)
    {
        var client = new AzureOpenAIClient(
            new Uri(configuration.ApiBaseUrl ?? throw new ArgumentNullException(nameof(configuration.ApiBaseUrl))),
            new AzureKeyCredential(configuration.ApiKey ?? throw new ArgumentNullException(nameof(configuration.ApiKey)))
        );

        var modelName = configuration.ModelName ??
            throw new ArgumentNullException(nameof(configuration.ModelName));
        var builder = new ChatClientBuilder(sp =>
            client.GetChatClient(modelName).AsIChatClient());

        builder.UseFunctionInvocation(); // required for RAG/MCP tools
        return Task.FromResult<IChatClient>(builder.Build());
    }
}

// Registration
Configure<ChatClientFactoryOptions>(options =>
{
    options.AddFactory<AzureOpenAIChatClientFactory>("AzureOpenAI");
});
```

---

## RAG, Document Indexing & Chunking (Pro)

RAG requires an **embedder** (e.g., OpenAI `text-embedding-3-small`, Ollama `nomic-embed-text`) and a **vector store** per workspace:

```bash
abp add-package Volo.AIManagement.VectorStores.Pgvector   # or .MongoDB / .Qdrant
```

Built-in provider names: embedders `OpenAI`, `Ollama`; vector stores `MongoDb`, `Pgvector`, `Qdrant`. The `MongoDb` provider requires **MongoDB Atlas** (`$vectorSearch`) with a `vector_index` on the `AIVectorEmbeddings` collection. Qdrant settings discard the URL scheme — do not rely on `https://` for TLS.

### Document Processing Pipeline

When a file (`.txt`, `.md`, `.pdf` by default, max 10 MB — configurable via `WorkspaceDataSourceOptions`) is uploaded as a workspace data source:

1. File stored in blob storage.
2. `IndexDocumentJob` queued.
3. `DocumentProcessingManager` extracts text via content-type-specific `IDocumentTextExtractor`s.
4. Text chunked at a **1000-character target**; paragraph path carries up to **200 characters** into the next chunk; oversized paragraphs split by line with a **50-character carry**. Chunks are persisted via `IDocumentChunkRepository`.
5. Embeddings generated in **ordered batches** and stored in the configured vector store.
6. Data source marked `IsProcessed = true` after the final batch succeeds.

Each indexing run has an identifier and a per-data-source **distributed lock**; stale/duplicate batches are ignored, out-of-order batches retried by the background job system. A failed run can leave partial vector data until retry or re-index. Deleting a data source removes its vector embeddings, document chunks and blob.

**v10.6+:** indexing gained **bounded-batch resilience** (batch work is bounded so a failing or oversized batch cannot wedge the whole indexing run), and `IDocumentChunkRepository` gained a **new paged datasource query overload** — page through a data source's chunks instead of loading them all when building custom re-indexing or inspection logic.

### Indexing Options

```csharp
Configure<AIManagementIndexingOptions>(options =>
{
    options.EmbeddingBatchSize = 100;             // default 50
    options.MaxConcurrentIndexingJobs = 2;        // default 1
    options.DistributedLockTimeoutSeconds = 15;   // default 0 = immediate attempt
});
```

Both the concurrency limiter and per-data-source lock use the ABP distributed lock — cross-instance only with a real distributed lock provider.

### Chat Integration

A resolved dynamic (or overridden system) workspace with embedder config gets a `search_documents` tool function wrapped around its chat client (delegates to `IDocumentSearchService`, `TopK = 5`). The client **must** be a `FunctionInvokingChatClient` — built-in factories call `UseFunctionInvocation()`; custom factories must too. If RAG retrieval fails, chat continues without injected context.

### Data Source HTTP API

Under `/api/ai-management/workspace-data-sources`: `POST /workspace/{workspaceId}` (upload), `GET /by-workspace/{workspaceId}`, `GET /{id}`, `PUT /{id}`, `DELETE /{id}`, `GET /{id}/download`, `POST /{id}/reindex`, `POST /workspace/{workspaceId}/reindex-all`.

---

## Best Practices

1. **Pick the right library:** Microsoft.Extensions.AI for reusable modules; Agent Framework for applications; Semantic Kernel only for its specific features.
2. **Configure in `PreConfigureServices`:** `AbpAIWorkspaceOptions` must be set before service registration.
3. **Use accessors for optional AI:** `IChatClientAccessor` / `IKernelAccessor` return `null` when unconfigured — always null-check.
4. **One workspace per AI use case:** isolate configurations (model, prompt, provider) per named scope.
5. **Never persist secrets in code:** feed workspace API keys from `IConfiguration`/secret stores; restrict AI Management admin permissions.
6. **Call `UseFunctionInvocation()`** in custom chat client factories, or RAG/MCP tools will not execute.
7. **Tune indexing conservatively:** raise `MaxConcurrentIndexingJobs` only after checking embedder rate limits and vector-store capacity; use a real distributed lock provider in clusters.
8. **Authorize consumption:** use `RequiredPermissionName` or the `Volo.AIManagement.Workspaces.Workspace.Consume` resource permission — consumption grants do not allow editing workspaces or uploading documents.
9. **Multi-tenancy:** workspaces, MCP configs and data sources are **not** tenant-scoped — control access via resource permissions, not tenant isolation.

---

## Related

- [ABP Framework](../abp-framework/SKILL.md) — module system, `PreConfigureServices`, DI
- [Infrastructure](../abp-infrastructure/SKILL.md) — background jobs (indexing), distributed lock, BLOB storing
- ABP Docs: https://abp.io/docs/latest/framework/infrastructure/artificial-intelligence
