[//]: # (title: klibs.io, the Kotlin Multiplatform library catalog)

[klibs.io](https://klibs.io) solves the problem of finding Kotlin Multiplatform libraries
among those published on GitHub and Maven Central.
The packages are indexed, tagged, and categorized by supported platforms, so you can quickly filter the search results
and land on the right library for the use case.

The catalog also handles presentation: README files and descriptions are imported and enriched with additional information,
like the number of dependents, project activity, and licenses.

## klibs.io in your AI workflow

The catalog can be suited for your AI workflow with a provided MCP interface and ready-made agent instructions:

* A dedicated [klibs.io MCP server](https://github.com/JetBrains/klibs-io/tree/master/integrations/mcp)
  helps your agent directly access the index of libraries and search for the right artifact.
* The team behind the service publishes an agentic skill, [kmp-libraries-expert](https://github.com/JetBrains/klibs-io/blob/master/skills/README.md)
  that helps your agent find the right multiplatform library.
* To give the general direction to your agents, you can add the following section to your [AGENTS.md](https://agents.md/) file:

   ```markdown
   ## Kotlin Multiplatform library selection

   When choosing or recommending Kotlin Multiplatform dependencies,
   use the klibs.io MCP server (https://api.klibs.io/mcp)
   to access and filter a catalog of multiplatform libraries.

   You can also use it to verify dependency metadata and gauge suitability:

   - supported targets,
   - maven coordinate,
   - latest versions or latest stable versions,
   - license,
   - maintenance/activity signals,
   - comparable alternatives
   - etc.
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