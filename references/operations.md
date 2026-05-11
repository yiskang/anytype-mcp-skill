# anytype-mcp Operations Reference

All tools exposed by the `anytype-mcp` MCP server. Each is a direct mapping of an Anytype REST API endpoint.

## Auth

| Tool | Description |
|---|---|
| *(env var)* | Auth uses `OPENAPI_MCP_HEADERS` with `Bearer <API_KEY>`. No tool needed — the server handles it. |

Getting an API key: `npx -y @anyproto/anytype-mcp get-key` or via Anytype Desktop → Settings → API Keys.

---

## Spaces & Members

| Tool | Parameters | Notes |
|---|---|---|
| `API-list-spaces` | *(none)* | Returns all spaces the API key has access to. Always call this first if you don't have a space_id. |
| `API-get-space` | `space_id` | Returns space metadata (name, description, icon). |
| `API-list-members` | `space_id` | Lists all members of a space with their roles. |
| `API-get-member` | `space_id`, `member_id` | Returns one member's profile and role. |

Members are **read-only** — no add/remove available.

---

## Objects

| Tool | Parameters | Notes |
|---|---|---|
| `API-get-objects` | `space_id`, `type?`, `offset?`, `limit?` | Lists objects. Filter by type key. Paginated — follow `has_more`. |
| `API-get-object` | `space_id`, `object_id` | Full object with properties and body content (markdown). |
| `API-create-object` | `space_id`, `type_key`, `name`, `properties?`, `body?`, `icon?`, `template_id?` | Creates a new object. |
| `API-update-object` | `space_id`, `object_id`, `name?`, `properties?`, `body?`, `icon?` | Partial update — only supplied fields change. |
| `API-delete-object` | `space_id`, `object_id` | Soft-deletes (archives) the object. |
| `API-search-objects` | `query`, `types?`, `offset?`, `limit?` | Global search across all spaces. |
| `API-search-objects-in-space` | `space_id`, `query`, `types?`, `offset?`, `limit?` | Searches within a specific space. Best for most queries. |

**Property format when writing**: pass as an array `[{ "key": "status", "select": "<tag_id>" }, { "key": "due_date", "date": "2025-06-01T00:00:00Z" }]`. Format depends on the property type — see property types below.

### Property value formats

| Format | Write as | Read as |
|---|---|---|
| `text`, `url`, `email`, `phone` | `{ "text": "value" }` | string |
| `number` | `{ "number": 42 }` | number |
| `checkbox` | `{ "checkbox": true }` | boolean |
| `date` | `{ "date": "ISO 8601 string" }` | string |
| `select` | `{ "select": "<tag_id>" }` | tag key string |
| `multi_select` | `{ "multi_select": ["<tag_id>", ...] }` | array of tag keys |
| `objects` (relation) | `{ "objects": ["<object_id>", ...] }` | array of object IDs |

---

## Types

| Tool | Parameters | Notes |
|---|---|---|
| `API-get-types` | `space_id` | Lists all types in the space (including system types). Check this before creating. |
| `API-get-type` | `space_id`, `type_key` | Full type detail: layout, properties with formats. |
| `API-create-type` | `space_id`, `key`, `name`, `plural_name?`, `layout?`, `icon?`, `properties?` | Creates a new type. Idempotent — check first with `API-get-types`. |

Common type layouts: `basic`, `todo`, `profile`, `note`, `bookmark`, `collection`, `set`.

---

## Properties

| Tool | Parameters | Notes |
|---|---|---|
| `API-get-properties` | `space_id` | Lists all properties in the space. Properties are global — one key = one format everywhere. |
| `API-get-property` | `space_id`, `property_key` | Returns a single property's key, name, and format. |
| `API-create-property` | `space_id`, `key`, `name`, `format` | Creates a new property. Fails if key already exists with a different format. |

Valid property formats: `text`, `number`, `date`, `checkbox`, `url`, `email`, `phone`, `select`, `multi_select`, `objects`.

---

## Tags

| Tool | Parameters | Notes |
|---|---|---|
| `API-list-tags` | `space_id`, `property_key` | Lists all tag options for a select/multi_select property. |
| `API-create-tag` | `space_id`, `property_key`, `name`, `color?` | Creates a new tag option. Returns the tag ID to use in property writes. |

Valid tag colors: `grey`, `yellow`, `orange`, `red`, `pink`, `purple`, `blue`, `ice`, `teal`, `lime`.

**Always list tags before writing a select/multi_select property.** The API requires tag IDs, not display names.

---

## Templates

| Tool | Parameters | Notes |
|---|---|---|
| `API-list-templates` | `space_id`, `type_key` | Lists templates available for a type. |
| `API-get-template` | `space_id`, `template_id` | Returns a template's full content. |

Templates are **read-only** via MCP — you can use a template's ID when calling `API-create-object` via `template_id`, but you cannot create or modify templates.

---

## Collections & Lists

| Tool | Parameters | Notes |
|---|---|---|
| `API-get-list-views` | `space_id`, `list_id` | Returns all views defined on a collection/list object (e.g. GTD views: Inbox, Waiting, Someday). |
| `API-get-list-view-objects` | `space_id`, `list_id`, `view_id`, `offset?`, `limit?` | Fetches objects as filtered/sorted by a specific view. Critical for GTD-style filtered task lists. |
| `API-add-objects-to-list` | `space_id`, `list_id`, `object_ids` | Adds one or more objects to a collection. `object_ids` is an array. |
| `API-remove-object-from-list` | `space_id`, `list_id`, `object_id` | Removes a single object from a collection. |
| `API-create-object` (with type `collection`) | `space_id`, `type_key: "collection"`, `name` | Create a new collection by using type_key `"collection"`. |

**GTD pattern**: Your GTD "My Tasks" list has multiple views (Inbox, Waiting, Someday, etc.). Use `API-get-list-views` once to discover all view IDs, then `API-get-list-view-objects` with the correct `view_id` to fetch each filtered bucket.

---

## First-time discovery

Run this sequence **once** whenever the user's space ID, list ID, or view IDs are not already in context. Do not repeat it on every call — if IDs were established earlier in the conversation, use them directly.

### Step 1 — Discover the space

```
API-list-spaces  (no parameters)
```

- If one space is returned, use it silently.
- If multiple spaces are returned, show the list and ask the user which one to use.
- Hold the `space_id` in context for the rest of the conversation.

### Step 2 — Discover lists / collections (only if the user works with lists or GTD views)

```
API-get-objects  space_id=<from step 1>  type="collection"
```

Show the names to the user and ask which one is their tasks list (e.g. "My Tasks"). Hold the object ID as `list_id`.

If the user mentions a specific list name, filter by name rather than presenting all of them.

### Step 3 — Discover views (only if the user needs a specific view like Inbox or Waiting)

```
API-get-list-views  space_id=<from step 1>  list_id=<from step 2>
```

Show the view names and their IDs. The user can then refer to views by name ("show me my Inbox") and you map the name to the ID you just discovered.

### Persisting IDs across conversations

These IDs are stable — they don't change between sessions. Suggest that the user save them so discovery doesn't need to repeat:

> "I found your space and task list. If you'd like, I can remember these IDs so you don't need to rediscover them next time. Just say 'remember my Anytype setup' and I'll note them down."

If the user agrees, store the IDs in Claude's memory (or wherever the deployment supports persistence) with a note like:
```
Anytype space_id: <id>
Anytype tasks list_id: <id>  (name: "My Tasks")
Anytype views: Inbox=<id>, Waiting=<id>, Someday=<id>, ...
```
