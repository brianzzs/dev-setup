---
name: api-endpoint-conventions
description: >
  HTTP API endpoint conventions. Use when adding or changing any endpoint:
  matching repository conventions, required test paths, pagination,
  idempotency, documentation, and the mandatory copy/paste-ready curl example
  in the final response.
---

# APIs and endpoint changes

- Match the repository's routing, authentication/authorization, validation, serialization, error-contract, versioning, and OpenAPI conventions. Do not add cross-cutting infrastructure solely because it is fashionable.
- For new or changed behavior, cover the success path and at least the most relevant validation, authorization, not-found, conflict, or failure path when practical.
- Require pagination for potentially unbounded collections, idempotency for operations whose retry semantics need it, and caching only when access patterns justify it.
- Add or update endpoint documentation using the repository's mechanism (XML comments, OpenAPI metadata, decorators, docstrings), including a useful summary and response behavior.
- Every final response for an added or changed endpoint must include at least one concrete, copy/paste-ready `curl` for local testing and Postman import, in a fenced `bash` block. Derive the real method, route, local base URL, required headers, and valid body from the repository; never provide a generic template. Never invent secrets — if authentication or a runtime identifier is unknowable, use one clearly named placeholder and state how to obtain it.
