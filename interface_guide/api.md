# API Reference

## General Info

### Rebuilding

### Loading

### Safe Mode

## Hooks

The following sections describe function tags matching format `#slimecore:hook/.../<name>` referred to as **hooks** that are called automatically during key events. Datapacks may freely add their own functions to hook tags, but **MUST NOT** call the hook tags themselves. Some hooks have input data that includes further details about events; such input data will always be a struct at NBT storage `slimecore:hook` `<name>`. Hook inputs are set by SlimeCore just before the hook is called and **MUST NOT** be modified.

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

### Rebuild

#### `#slimecore:hook/rebuild/start`

**Call Time:** just before a rebuild starts.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `explicit` | *(matches input of [explicit rebuild function](#explicit-rebuild-function))* (or none) | Matches the input given to the [explicit rebuild function](#explicit-rebuild-function). Not present if rebuild was triggered automatically via reload. | 

#### `#slimecore:hook/rebuild/end`

**Call Time:** just after a rebuild finishes.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `result` | *(matches `result` key of [explicit rebuild](#explicit-rebuild-function) output)* | The result of the rebuild. | 

---

### Load Process

#### `#slimecore:hook/load/start`

**Call Time:** just before a load starts.

*No input.*

#### `#slimecore:hook/load/preload_entrypoints`

**Call Time:** just before preload entrypoint tags start getting called.

*No input.*

#### `#slimecore:hook/load/loads`

**Call Time:** just before load tags start getting called.

*No input.*

#### `#slimecore:hook/load/entrypoints`

**Call Time:** just before entrypoint tags start getting called.

*No input.*

#### `#slimecore:hook/load/entrypoints`

**Call Time:** just after a load ends.

*No input.*

---

### Individual Tag Calls

*Each of these hooks have a `pre` and `post` variant that are called just before or after the relavent pack tag is called, respectively.*

#### `#slimecore:hook/call/<pre|post>/load`

**Call Time:** just before/after a datapack's load tag is called.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | String (pack ID) | The pack ID of the datapack that the tag belongs to. | 

#### `#slimecore:hook/call/<pre|post>/disable`

**Call Time:** just before/after a datapack's disable tag is called.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | String (pack ID) | The pack ID of the datapack that the tag belongs to. | 

#### `#slimecore:hook/call/<pre|post>/uninstall`

**Call Time:** just before/after a datapack's uninstall tag is called.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | String (pack ID) | The pack ID of the datapack that the tag belongs to. | 

#### `#slimecore:hook/call/<pre|post>/safe_mode`

**Call Time:** just before/after a datapack's safe mode tag is called.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | String (pack ID) | The pack ID of the datapack that the tag belongs to. | 

#### `#slimecore:hook/call/<pre|post>/entrypoint`

**Call Time:** just before/after an entrypoint's tag is called.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | String (pack ID) | The pack ID of the datapack that the entrypoint belongs to. | 
| `id` | String (entrypoint ID) | The ID of the entrypoint. | 

#### `#slimecore:hook/call/<pre|post>/preload_entrypoint`

**Call Time:** just before/after a preload entrypoint's tag is called.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | String (pack ID) | The pack ID of the datapack that the preload entrypoint belongs to. | 
| `id` | String (preload entrypoint ID) | The ID of the preload entrypoint. | 

---

### Safe Mode (Reload)

#### `#slimecore:hook/safe_mode/start`

**Call Time:** just after a world reload if safe mode is enabled/triggered, before safe mode tags are called.

*No input; see `safe_mode` key in [world data](#world-data).*

#### `#slimecore:hook/safe_mode/end`

**Call Time:** just after a world reload if safe mode is enabled/triggered, after safe mode tags are called.

*No input; see `safe_mode` key in [world data](#world-data).*

---

### Other

#### `#slimecore:hook/uninstall_slimecore`

**Call Time:** just before SlimeCore is uninstalled (and all SlimeCore-loaded datapacks disabled).

*No input.*

---

## Data

The following sections describe the data that SlimeCore populates NBT storage location `slimecore:data` with. Datapacks are free to read data from this location but **MUST NOT** modify it--it is **read only**.

### Build Data

### World Data

## Explicit Rebuild Function

## Eval Functions

### Eval Build

### Eval Pack

### Eval Version Requirement

## Configuration
