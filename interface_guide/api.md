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

Called just before SlimeCore performs a rebuild.

### Load Process Hooks

### Tag Call Hooks

### Safe Mode Hooks

### SlimeCore Uninstall Hook

## Data

### Build Data

### World Data

## Eval Functions

### Eval Build

### Eval Pack

### Eval Version Requirement