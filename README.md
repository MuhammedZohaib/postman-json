## Postman JSON Codex Skill

This repo provides a **Codex/Cursor agent skill** that generates or refreshes **Postman collection JSON** from an Express/Node codebase. The skill is not meant to be used directly via CLI; instead, it is **invoked automatically by AI agents** when they need to understand or sync API endpoints with Postman.

### What the skill does

- **Discovers routers and routes**
  - Finds `app.use("/prefix", routerAlias)` in the server entry file.
  - Resolves `routerAlias` from both ESM imports and CommonJS `require(...)` aliases.
  - Parses `router.<method>(path, ...middleware, handler)` calls (e.g. `get`, `post`, `put`, `delete`).
- **Infers request requirements**
  - Adds bearer auth when auth middleware is detected.
  - Enables multipart and file fields when upload middleware or `req.file` / `req.files` are used.
  - Infers JSON body fields from `req.body` access and middleware-derived values (for example `turnstileToken`).
- **Syncs a deterministic subtree in a collection**
  - Preserves non-generated top-level items in the Postman collection.
  - Replaces only the generated root subtree owned by this skill.
  - Keeps folder hierarchy aligned with route file paths and prefixes.

### How agents use it

- **Triggering**: Codex/Cursor agents choose this skill when the user asks to generate or update a Postman collection for an Express/Node project (for example: “sync a Postman collection for this API”).
- **Inputs**: The agent passes in information such as:
  - Project root
  - Server entry file (if known)
  - Desired output location and collection name
  - Base URL (or leaves it to defaults)
- **Behavior**: The skill analyzes the code, updates the relevant subtree of the Postman collection JSON, and returns the updated collection to the calling agent, which may then write it into the user’s repo.

As a user, you **do not need to call this skill directly**. You just describe your intent in natural language; the agent decides when and how to invoke the skill.

### Implementation details (for contributors)

- Core logic lives in **`scripts/sync_postman_collection.py`**.
- Additional inference rules (if present) may live under **`references/request-inference-rules.md`**.
- The skill is designed to be:
  - **Deterministic**: the same codebase and parameters produce the same collection subtree.
  - **Route-scoped**: inference is local to each endpoint; no unintended leakage between routes.
  - **Non-destructive**: only the generated subtree is replaced; custom, non-generated parts of the collection are preserved.

When extending this skill, prefer improving the parser and inference rules over encouraging users to hand-edit generated Postman requests.
