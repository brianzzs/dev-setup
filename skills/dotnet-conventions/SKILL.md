---
name: dotnet-conventions
description: >
  .NET and C# conventions. Use for any C# or .NET work: reading solution and
  project configuration, choosing language features and APIs, EF Core usage,
  async and cancellation patterns, test stack selection, and build/test
  validation commands.
---

# .NET and C#

- Read `global.json`, `Directory.Build.*`, solution/project files, package configuration, and nearby code before selecting APIs or language features. Existing targets win; do not upgrade a repository unless asked.
- For a new unconstrained .NET project, prefer .NET 10 and the corresponding supported C# version. Otherwise use the language version the project supports.
- Respect nullable reference types and existing analyzer rules. Prefer idiomatic C#, descriptive names, dependency injection, options, and structured `ILogger` logging where the project uses them.
- Use `async`/`await` for genuine I/O and propagate `CancellationToken` through cancellable boundaries when consistent with the codebase. Avoid `.Result`, `.Wait()`, and unnecessary async wrappers.
- With EF Core, preserve query semantics and transaction boundaries; project only needed data, prevent N+1 access, and use no-tracking reads when appropriate. In existing repositories, do not introduce a repository layer unless their architecture calls for it. Net-new repositories/projects run a `grill-me` session to discuss the architecture first.
- Keep public API contracts, nullable annotations, serialization names, and error responses deliberate. Update XML/OpenAPI documentation through the mechanism already used by the repository.
- Use the existing test stack (for example xUnit, NUnit, Moq, or NSubstitute); do not introduce another framework without need.
- Always use the `tdd` skill when developing new features or hotfixes.
- Prefer repository validation scripts. When none exist, select the relevant solution/project and run suitable `dotnet build` and `dotnet test` commands, plus configured formatting/analyzer checks.
