# API Reference

## General Info

### Rebuilding

### Loading

### Safe Mode

## Hooks

SlimeCore includes function tags matching format `#slimecore:hook/...` called **hooks** that are called automatically during key events. Datapacks may freely add their own functions to hook tags, but **MUST NOT** call SlimeCore's hook tags themselves. Some hooks have input data that includes further details about events. If a hook has input data, it will always be a struct at NBT storage `slimecore:hook` `<name>`, given the hook tag format `slimecore:hook/<path>/<name>`. Hook inputs are set by SlimeCore just before the hook is called and **MUST NOT** be modified.

For example, if input for hook `#slimecore:hook/foo/bar` is described like so:

| Key | Type | Description |
| :-- | :-- | :-- |
| `baz` | String | Example key. | 
| `qux` | String | Example key. | 

Then you can do the following commands to retrieve such inputs:

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