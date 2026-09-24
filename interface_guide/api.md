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

### Rebuild Hooks

#### `#slimecore:hook/rebuild/start`

Called just before a [rebuild](#rebuilding) starts.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `explicit` | *(matches input of [explicit rebuild function](#explicit-rebuild-function))* (or none) | Matches the input given to the [explicit rebuild function](#explicit-rebuild-function) if an explicit rebuild was initiated. Is not present if rebuild was triggered automatically via reload. | 

#### `#slimecore:hook/rebuild/end`

Called just after a [rebuild](#rebuilding) finishes.

**Input:**
| Key | Type | Description |
| :-- | :-- | :-- |
| `result` | *(matches `result` key of [explicit rebuild](#explicit-rebuild-function) output)* | The result of the rebuild. | 

### Load Process Hooks

#### `#slimecore:hook/load/start`

Called just before a [load](#loading) starts.

### Tag Call Hooks

### Safe Mode Hooks

### SlimeCore Uninstall Hook

## Data

### Build Data

### World Data

## Explicit Rebuild Function

## Eval Functions

### Eval Build

### Eval Pack

### Eval Version Requirement