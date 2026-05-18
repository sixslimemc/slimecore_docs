# Full Dev Guide

This guide is from the perspective of creating a new datapack, but should be able to be easily adapted to converting a non-SlimeCore-loaded datapack into a SlimeCore-loaded datapack. If you are converting a datapack, it is highly advised to make a backup beforeso.

## Setup

### Primary Namespace / Pack ID

Designate a namespace (`<datapack>/data/<namespace>`) as your datapack's *primary namespace*. This namespace must be the only namespace that your datapack defines ***new*** files in. The name of this namespace is your datapack's **pack ID**. 

All other namespaces included in your datapack are considered *secondary namespaces* (e.g. `minecraft`, `slimecore`, primary namespaces of your datapack's dependencies). Your datapack should not define any *new* files within secondary namespaces, but may *overwrite/modify* files in them (e.g. appending to a tag).

### File Structure

A SlimeCore-loaded datapack includes the following files:
```
<datapack>
└── data
    ├── slimecore/tags/function
    │   └── manifest.json
    └── <pack ID>/tags/function
        ├── entrypoint
        │   └── <entrypoint ID...>.json
        ├── preload_entrypoint
        │   └── <preload entrypoint ID...>.json
        ├── disable.json
        ├── load.json
        ├── safe_mode.json
        └── uninstall.json
```

