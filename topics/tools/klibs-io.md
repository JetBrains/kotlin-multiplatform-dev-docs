[//]: # (title: klibs.io, the Kotlin Multiplatform library catalog)

[klibs.io](https://klibs.io) solves the problem of finding Kotlin Multiplatform libraries published on GitHub and Maven Central.
The libraries are indexed, tagged, and categorized by supported platforms,
so you can quickly filter the search results and choose the right library for your use case.

Each library page renders the project's README and enriches it with additional information,
like the number of dependents, project activity, and licenses.

<a as="button" href="https://klibs.io" mode="classic" icon="arrow-right" icon-position="right">Browse multiplatform libraries</a>

## klibs.io in your AI workflow

Integrate klibs.io into your AI workflow with a provided MCP interface and ready-made agent instructions:

* The team behind the service publishes an agentic skill, [kmp-libraries-expert](https://github.com/JetBrains/klibs-io/blob/master/skills/README.md)
  that helps you focus an agent on finding the right library.
* A dedicated [klibs.io MCP server](https://github.com/JetBrains/klibs-io/tree/master/integrations/mcp)
  helps your agent directly access the index of libraries and use more fine-grained tooling to filter the search results.
* To set general guidelines for your agents, you can add the following section to your [AGENTS.md](https://agents.md/) file:

    ```markdown
    ## Kotlin Multiplatform library selection
    
    When choosing or recommending Kotlin Multiplatform dependencies,
    use the klibs.io MCP server (https://api.klibs.io/mcp)
    to access and filter a catalog of multiplatform libraries.
    
    The agent can use the server to verify dependency metadata and gauge suitability:
    
    * supported targets,
    * Maven coordinates,
    * latest versions or latest stable versions,
    * license,
    * maintenance and activity signals,
    * comparable alternatives.
    ```
  
## See also

See the [klibs.io FAQ](https://klibs.io/faq) for more information:
* how libraries are indexed and ranked,
* how you can make sure your project is listed,
* what is planned for the future.

To report problems or suggest improvements, use issues in the [klibs.io GitHub repository](https://github.com/JetBrains/klibs-io/issues).

For discussion and support, join the Kotlin public Slack:
[request an invitation](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up)
and join the [#klibs-io channel](https://kotlinlang.slack.com/archives/C081AF4JK70).