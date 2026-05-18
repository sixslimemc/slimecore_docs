# Full Dev Guide

This guide is from the perspective of creating a new datapack, but should be able to be easily adapted to converting a non-SlimeCore-loaded datapack into a SlimeCore-loaded datapack. If you are converting a datapack, it is highly advised to make a backup beforeso.

## Structure Setup

A SlimeCore-loaded datapack adheres to the following file structure:
```
<datapack>
├── data
│   ├── slimecore/tags/function
│   │   └── manifest.json
│   ├── <pack ID>/tags/function
│   │   ├── entrypoint
│   │   │   └── (empty)
│   │   ├── preload_entrypoint
│   │   │   └── (empty)
│   │   ├── disable.json
│   │   ├── load.json
│   │   ├── safe_mode.json
│   │   └── uninstall.json
│   └── <secondary namespaces...>
└── pack.mcmeta
```

