---
name: abp-ai
description: "ABP Framework v10.x (10.4–10.6) AI integration: Microsoft.Extensions.AI (IChatClient), Semantic Kernel, Microsoft Agent Framework, AI Workspaces, AI Management module (Pro) with RAG, document indexing/chunking and MCP servers. Use when you need chat clients, AI agents, RAG or AI workspace management in ABP."
---

# ABP Framework — AI Integration

ABP Framework v10.x (10.4–10.6) AI integration. Core concept: **AI Workspace** — isolated named AI configuration scope. Install: `abp add-package Volo.Abp.AI`.

## Trigger

- "ABP AI" / "ABP chat client" / "ABP AI workspace"
- "ABP Semantic Kernel" / "ABP Agent Framework"
- "ABP AI Management" / "ABP RAG" / "ABP MCP server"

## Library Choice

| Library | Use for |
|---|---|
| Microsoft.Extensions.AI | Reusable modules/libraries (minimal `IChatClient` abstraction) |
| Microsoft Agent Framework | **Recommended for apps** (AutoGen + SK successor) |
| Semantic Kernel | Only for its specific features |

## Microsoft.Extensions.AI

```csharp
// Direct — throws if unconfigured (workspace falls back to default)
public MyService(IChatClient chatClient) { ... }
var text = await _chatClient.GetResponseAsync(prompt);

// Optional AI — always null-check
var client = _chatClientAccessor.ChatClient;   // IChatClientAccessor
if (client is null) return "No chat client configured";

// Workspace-scoped: IChatClient<CommentSummarization>, IChatClientAccessor<CommentSummarization>
[WorkspaceName("CommentSummarization")]        // unique, case-sensitive, no spaces
public class CommentSummarization { }
```

**Configuration — in `PreConfigureServices`, before service registration:**

```csharp
PreConfigure<AbpAIWorkspaceOptions>(options =>
{
    options.Workspaces.ConfigureDefault(cfg =>                     // or .Configure<CommentSummarization>(...)
        cfg.ConfigureChatClient(c =>
            c.Builder = new ChatClientBuilder(sp => new OllamaApiClient("http://localhost:11434", "mistral"))));
});
```

## Agent Framework

Works on top of `IChatClient` — same workspace configuration:

```csharp
AIAgent agent = _chatClient.CreateAIAgent(instructions: "You are a helpful assistant.");
AgentRunResponse response = await agent.RunAsync(userMessage);
return response.Text;
```

## Semantic Kernel

```csharp
var kernel = _kernelAccessor.Kernel;           // IKernelAccessor / IKernelAccessor<TWorkSpace>
if (kernel is null) return "No kernel configured";   // no default fallback for typed kernels
return await kernel.InvokeAsync(prompt);

// Config: cfg.ConfigureKernel(k => k.Builder = Kernel.CreateBuilder().AddAzureOpenAIChatClient("...", "..."));
```

## AI Management Module (Pro — Team license+)

`abp add-module Volo.AIManagement` + at least one provider: `Volo.AIManagement.OpenAI` or `Volo.AIManagement.Ollama`. Admin UI + API for **dynamic** (DB-persisted) workspaces; **system** workspaces stay code-defined unless `OverrideSystemConfiguration = true`.

```csharp
// Programmatic workspace (seeding)
var ws = await _applicationWorkspaceManager.CreateAsync(name: "Support", provider: "OpenAI", modelName: "gpt-4");
ws.ApiKey = _configuration["AI:OpenAI:ApiKey"];   // never hard-code secrets
await _workspaceRepository.InsertAsync(ws);

// Remote client (Volo.AIManagement.Client.HttpApi.Client)
var response = await _chatService.ChatCompletionsAsync(workspaceName, request);  // IChatCompletionClientAppService
```

- **OpenAI-compatible API** at `/v1` (`model` = workspace name, Bearer auth).
- **MCP servers** as workspace tools: Stdio/SSE/StreamableHttp; timeouts via `McpClientFactoryOptions`.
- **Custom provider:** implement `IChatClientFactory`, register with `ChatClientFactoryOptions.AddFactory<T>("Name")`; call `builder.UseFunctionInvocation()` or RAG/MCP tools won't run.

## RAG & Document Indexing (Pro)

Needs embedder (OpenAI/Ollama) + vector store (`Volo.AIManagement.VectorStores.Pgvector` / `.MongoDB` (Atlas only) / `.Qdrant`).

Pipeline: upload (`.txt`/`.md`/`.pdf`, 10 MB default — `WorkspaceDataSourceOptions`) → blob → `IndexDocumentJob` → `DocumentProcessingManager` extracts + chunks (1000-char target, 200-char paragraph carry, 50-char line carry; chunks via `IDocumentChunkRepository`) → ordered embedding batches → vector store → `IsProcessed = true`.

```csharp
Configure<AIManagementIndexingOptions>(options =>
{
    options.EmbeddingBatchSize = 100;            // default 50
    options.MaxConcurrentIndexingJobs = 2;       // default 1 — mind provider rate limits
    options.DistributedLockTimeoutSeconds = 15;  // default 0
});
```

Chat gets a `search_documents` tool (`TopK = 5`) when embedder is configured; requires a `FunctionInvokingChatClient`. Data source API: `/api/ai-management/workspace-data-sources` (upload, list, download, delete, reindex, reindex-all).

**v10.6+:** bounded-batch indexing resilience (a failing/oversized batch can't wedge the run) and a new paged datasource query overload on `IDocumentChunkRepository`.

## Best Practices

1. Agent Framework for apps; Microsoft.Extensions.AI for reusable modules
2. Configure `AbpAIWorkspaceOptions` in `PreConfigureServices` only
3. Null-check `IChatClientAccessor` / `IKernelAccessor` for optional AI
4. One workspace per AI use case; secrets from configuration, never code
5. `UseFunctionInvocation()` in every custom chat client factory
6. Workspaces/MCP/data sources are **not** tenant-scoped — use resource permissions (`Volo.AIManagement.Workspaces.Workspace.Consume`)

## Related

[ABP Framework](../abp-framework/SKILL.md) · [Infrastructure](../abp-infrastructure/SKILL.md) · Docs: https://abp.io/docs/latest/framework/infrastructure/artificial-intelligence
