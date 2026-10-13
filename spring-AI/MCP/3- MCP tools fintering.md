# Spring AI — MCP Tools Filtering

Since you're learning MCP in Spring AI, let's understand MCP tools filtering with a practical example and Java code.

## 1. What is MCP tools filtering?

An MCP server can expose multiple tools to an MCP client. However, your Spring AI application may not need every tool.

MCP tools filtering means controlling which tools are made available to your AI application from the tools exposed by an MCP server.

For example, imagine you connect your Spring AI application to a filesystem MCP server.

Filesystem MCP Server

Exposes several tools

MCP Tools Filtering

Allowed: `read_file`

AI can read files

Allowed: `list_directory`

AI can list files

Excluded: `write_file`

AI cannot invoke this tool through the filtered tool set

Spring AI ChatClient / Model

Receives only the selected tools

The goal is to give the model only the capabilities it needs, which helps reduce accidental tool usage and makes tool selection more predictable.

Important distinction: Filtering tools is not the same as securing the MCP server. If a tool is excluded from the AI's tool list, that alone does not prevent another client from calling it directly. Sensitive operations still need authorization and server-side validation.

## 2. Why do we need MCP tools filtering?

Suppose your application connects to two MCP servers:

| MCP server | Tools exposed                               |
| ---------- | ------------------------------------------- |
| Filesystem | `read_file`, `write_file`, `list_directory` |
| Fetch      | `fetch_url`                                 |

Your application only needs to answer questions about local files and retrieve website content.

You might want to:

- Allow `read_file` and `list_directory`.
- Allow `fetch_url`.
- Exclude `write_file` because the application should not modify files.

Without filtering, you might expose more tools than the AI needs. With filtering, you can define a smaller, more intentional tool set.

## 3. Implement filtering using `McpToolFilter`

Spring AI provides the `McpToolFilter` interface for filtering discovered MCP tools. It lets you decide whether a tool should be included based on:

- The MCP server connection information.
- The tool's name and description.

The interface is essentially a predicate: return `true` to include a tool and `false` to exclude it.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.spring.io\&sz=32)

Home

+1



### Example: Filter tools from your Fetch MCP server

Let's say your Fetch MCP server exposes these tools:

| Tool name    | Include? |
| ------------ | -------- |
| `fetch_url`  | Yes      |
| `read_file`  | No       |
| `write_file` | No       |

We'll create a filter that permits only `fetch_url`.

FetchToolFilter.java

```

import org.springframework.ai.mcp.McpConnectionInfo;
import org.springframework.ai.mcp.McpToolFilter;
import io.modelcontextprotocol.spec.McpSchema;

import org.springframework.stereotype.Component;

@Component
public class FetchToolFilter implements McpToolFilter {

    @Override
    public boolean test(
            McpConnectionInfo connectionInfo,
            McpSchema.Tool tool) {

        return "fetch_url".equals(tool.name());
    }
}

```

Let's break down the important part:

```
return "fetch_url".equals(tool.name());
```

- If the tool name is `fetch_url`, the result is `true`, so Spring AI includes it.
- If the tool name is `read_file`, the result is `false`, so Spring AI excludes it.
- The same applies to every other tool whose name isn't `fetch_url`.

Notice that this example filters by tool name, not by MCP server. If you have multiple servers, you can also use `connectionInfo` to restrict filtering to a particular server.

One important detail: The tool name must match the actual name returned by your MCP server. For your Fetch server, inspect the discovered tool names first; don't assume every Fetch implementation names its tool `fetch_url`.

## 5. Connect the filtered tools to `ChatClient`

With the MCP client starter and the filter bean configured, you can inject the tool provider and pass it to your `ChatClient`.

```
@Configuration
public class ChatClientConfig {

    @Bean
    ChatClient chatClient(
            ChatClient.Builder builder,
            SyncMcpToolCallbackProvider mcpTools) {

        return builder
                .defaultTools(mcpTools)
                .build();
    }
}
```

Then your application can call:

```
String answer = chatClient.prompt()
        .user("Fetch the contents of https://spring.io")
        .call()
        .content();
```

Assuming the MCP server exposes a compatible fetch tool and the model chooses to invoke it, Spring AI can execute that tool. The filter limits the available MCP tool callbacks; it doesn't guarantee that the model will call a tool on every prompt.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.spring.io\&sz=32)

Home

+1



This configuration uses the current Spring AI 2.x API style. If your project uses an older Spring AI release, check its version-specific API and property names.

## 6. A slightly more advanced filter

What if you want to allow multiple tools, but only from your Fetch MCP server?

You can combine the server identity and tool name checks:

```
@Component
public class FetchToolFilter implements McpToolFilter {

    @Override
    public boolean test(
            McpConnectionInfo connectionInfo,
            McpSchema.Tool tool) {

        boolean allowedTool =
                "fetch_url".equals(tool.name())
                || "fetch_document".equals(tool.name());

        // Add a server-identity check here if required.
        return allowedTool;
    }
}
```

This example filters by tool name only. To make the server restriction real, inspect the `McpConnectionInfo` API for your Spring AI version and check the connection's actual identifying field.

Remember: Spring AI's MCP client starter supports one `McpToolFilter` bean in the application context. If you need multiple filtering rules, combine them into one filter rather than defining several independent filter beans.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.spring.io\&sz=32)

Home

+1



## 7. MCP tools filtering vs. tool search

These concepts are related but solve different problems.

| Feature           | MCP tools filtering                                      | Tool search                                      |
| ----------------- | -------------------------------------------------------- | ------------------------------------------------ |
| Purpose           | Exclude tools you don't want available                   | Discover relevant tools from a larger collection |
| Tool availability | Permanently restricted by the filter's rules             | Tools can be discovered when needed              |
| Best suited for   | Capability restriction and reducing unnecessary exposure | Applications with dozens or hundreds of tools    |
| Example           | Never expose `write_file`                                | Find a suitable tool for searching documentation |

Spring AI 2.x also provides a `ToolSearchToolCallingAdvisor` for progressive tool disclosure, which is useful when an application has many tools.


