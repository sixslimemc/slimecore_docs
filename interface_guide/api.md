# API Reference

## General Info

### Rebuilding

### Loading

### Safe Mode

## Hooks

SlimeCore includes function tags matching format `#slimecore:hook/.../<name>` called **hooks** that are called automatically during key events. Datapacks may freely add their own functions to hook tags, but **MUST NOT** call SlimeCore's hook tags themselves. Some hooks have input data that includes further details about events; such input data will always be a struct at NBT storage `slimecore:hook` `<name>`. Hook inputs are set by SlimeCore just before the hook is called and **MUST NOT** be modified.

Here is an example of how these inputs will be described:

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `baz` | String | Example key. | 
| `qux` | String | Example key. | 

If this input is for hook `#slimecore:hook/foo/bar`, then you would retrieve these inputs like so:

```mcfunction
data get storage slimecore:hook bar.baz
data get storage slimecore:hook bar.qux
```

--- 

### Rebuild Hooks

Hooks called during the rebuild process.

#### `#slimecore:hook/rebuild/start`

Called just before a rebuild starts.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `explicit` | *(matches input of [explicit rebuild function](#explicit-rebuild-function))* (or none) | Matches the input given to the [explicit rebuild function](#explicit-rebuild-function) if an explicit rebuild was initiated. Is not present if rebuild was triggered automatically via reload. | 

#### `#slimecore:hook/rebuild/end`

Called just after a rebuild finishes.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `result` | *(matches `result` key of [explicit rebuild](#explicit-rebuild-function) output)* | The result of the rebuild. | 

---

### Load Process Hooks

Hooks called the load process.

#### `#slimecore:hook/load/start`

Called just before a load starts.

#### `#slimecore:hook/load/preload_entrypoints`

Called just before preload entrypoint tags start getting called.

#### `#slimecore:hook/load/loads`

Called just before load tags start getting called.

#### `#slimecore:hook/load/entrypoints`

Called just before entrypoint tags start getting called.

#### `#slimecore:hook/load/entrypoints`

Called just after a load ends.

---

### Tag Call Hooks

Hooks called when individual pack tags are called.

*Each of these hooks have a `pre` and `post` variant that are called just before or after the relavent pack tag is called, respectively.*

#### `#slimecore:hook/call/<pre|post>/load`

Called just before/after a datapack's load tag is called.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | String (pack ID) | The pack ID of the datapack that the tag belongs to. | 

#### `#slimecore:hook/call/<pre|post>/disable`

Called just before/after a datapack's disable tag is called.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | String (pack ID) | The pack ID of the datapack that the tag belongs to. | 

#### `#slimecore:hook/call/<pre|post>/uninstall`

Called just before/after a datapack's uninstall tag is called.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | String (pack ID) | The pack ID of the datapack that the tag belongs to. | 

#### `#slimecore:hook/call/<pre|post>/safe_mode`

Called just before/after a datapack's safe mode tag is called.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | String (pack ID) | The pack ID of the datapack that the tag belongs to. | 

#### `#slimecore:hook/call/<pre|post>/entrypoint`

Called just before/after an entrypoint's tag is called.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | String (pack ID) | The pack ID of the datapack that the entrypoint belongs to. | 
| `id` | String (entrypoint ID) | The ID of the entrypoint. | 

#### `#slimecore:hook/call/<pre|post>/preload_entrypoint`

Called just before/after a preload entrypoint's tag is called.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | String (pack ID) | The pack ID of the datapack that the preload entrypoint belongs to. | 
| `id` | String (preload entrypoint ID) | The ID of the preload entrypoint. | 

---

### Safe Mode Hooks

---

### SlimeCore Uninstall Hook

---

## Data

### Build Data

### World Data

## Explicit Rebuild Function

## Eval Functions

### Eval Build

### Eval Pack

### Eval Version Requirement