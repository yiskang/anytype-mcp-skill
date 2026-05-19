---
name: anytype-mcp
description: Use this skill whenever you need to interact with the user's Anytype workspace through the anytype-mcp MCP server. Triggers whenever the user mentions Anytype, asks to create/find/update/delete notes, pages, tasks, or any objects in Anytype, wants to search their knowledge base, manage spaces or members, work with collections or lists, set up object types or properties, or use tags. Also triggers for GTD workflows (inbox, waiting, someday), task management in Anytype, or any question about what's in their Anytype space. If the user references their notes, their space, their tasks, or their wiki and you have the anytype-mcp tools available, use this skill.
---

# Anytype MCP Skill

This skill covers how to interact with Anytype using the `anytype-mcp` MCP server tools. The server exposes Anytype's REST API as MCP tools — each tool is a direct mapping to an API endpoint.

See `references/operations.md` for the full tool reference organized by domain.

## Core mental model

Everything in Anytype is an **object**. Objects have a **type** (e.g. "task", "page", "note"), belong to a **space**, and can carry **properties** (typed key-value metadata). Properties use **tags** for `select` and `multi_select` values. Objects can be grouped into **collections** (curated lists) or **sets** (filter-based views). The `anytype-mcp` server exposes all of this over ~30 MCP tools.

**You always need a `space_id`** for most operations. If the user hasn't told you which space, run the discovery flow in `references/operations.md` — it takes one or two tool calls and only needs to happen once per conversation. If IDs were established earlier in the conversation or are in memory, use them directly without rediscovering. Check the auto-memory file `reference_anytype_space.md` for a cached space ID before calling `API-list-spaces`.

## Workflow patterns

### Querying tasks or notes

1. If you don't have the space ID, run the discovery flow (see `references/operations.md`). Otherwise use the ID already in context.
2. Use `API-search-objects-in-space` with a query for fast lookup by content, or `API-get-objects` with a `type` filter to list all objects of a given type.
3. If the user references a specific collection or list view (e.g. "my GTD Inbox"), use `API-get-list-views` to find the view ID, then `API-get-list-view-objects` to get the filtered results.
4. Call `API-get-object` for full object details including body content.

### Creating objects

1. Know the type key (e.g. `"task"`, `"page"`). Use `API-list-types` if you need to discover available types.
2. **Always ask the user if they want to use a template** before creating. Call `API-list-templates` with the type ID to show available options — never skip this step, even if the user didn't mention a template. Templates can only be applied at creation time; there is no way to apply one after the fact.
3. Know the property keys and their formats. Use `API-list-properties` or `API-get-property` to inspect before setting values — especially for `date` fields, which are stored in UTC regardless of the user's local timezone. Always confirm the expected format before writing.
4. Call `API-create-object` with `space_id`, `type_key`, `name`, and any property values. Include `template_id` if the user selected a template.
5. For select/multi_select properties, you need tag IDs — use `API-list-tags` to find existing ones or `API-create-tag` to make new ones first.

> **Date/time fields**: Anytype stores all date values in UTC. When the user provides a local time, convert it to UTC before writing (e.g. CST 22:30 → UTC 14:30). Always show both the UTC value and the local equivalent when confirming with the user.

### Updating objects

- Use `API-update-object` — pass only the fields you want to change; everything else is preserved.
- For moving objects into/out of a list, use `API-add-objects-to-list` / `API-remove-object-from-list`.

### Working with types and properties

- Always call `API-get-types` before creating a type — never create one that already exists.
- Properties are **global** across a space: one key has one format everywhere. Check with `API-get-properties` before creating.
- Use `API-get-type` for a detailed look at a specific type's layout and properties.

### Search strategy

- `API-search-objects-in-space` — searches within a specific space by content query. Best for "find notes about X".
- `API-search-objects` (global) — searches across all spaces. Use when the user hasn't specified a space.

## Key constraints

**Tags for select/multi_select**: When setting a select or multi_select property, you must pass tag IDs (not display names). Always `API-list-tags` for the relevant property first and match by name, or `API-create-tag` if the desired tag doesn't exist.

**Pagination**: `API-get-objects` and list views return paginated results. If the user asks "how many X do I have", fetch all pages (follow `has_more` / `offset`) before counting.

**API version**: The server sends `Anytype-Version: 2025-11-08` by default. If a tool returns unexpected 400/422 errors, this version header may be mismatched with the running Anytype app — advise the user to check their Anytype desktop version.

**No markdown body write**: The current anytype-mcp server maps the OpenAPI spec directly. The `body` / markdown content field is read-only on some endpoints depending on API version — if updating body content fails, inform the user this may not be supported yet.

**Members are read-only**: `API-list-members` and `API-get-member` are available, but there are no tools to add or remove members.

**Templates are read-only**: You can list and get templates, but not create them via MCP.

## Error handling

- `404` on a space/object ID → confirm the ID with the user; IDs change between API versions.
- `401` / `403` → the API key may be expired or the object is in a space the key doesn't have access to.
- Empty results from `API-get-objects` with a type filter → the type key might be wrong; use `API-get-types` to check exact keys.
- Property update silently ignored → the property key might not exist on that type; check with `API-get-type`.

## Communicating results to the user

- When listing objects, show name, type, and key properties (status, due date, etc.) in a compact table or list.
- When creating or updating, confirm what was changed with the object name and ID.
- If an operation fails, explain clearly and suggest the recovery step (e.g. "the tag 'waiting' doesn't exist yet — I can create it first").
