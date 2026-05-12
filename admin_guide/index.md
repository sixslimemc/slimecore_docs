# SlimeCore Admin Guide

SlimeCore is a datapack that loads and manages other datapacks, *that's it*. It intentionally does not provide any player-facing features on its own. Instead, it is designed such that *other* datapacks, **frontends**, can provide such functionality. This is primarily so *you*, as the user/admin, can customize your experience. While not strictly required, a frontend is essential for easy in-game administration. Frontends are not "special", they are installed like any other datapack.

Because of this paradigm, frontends define how you interact with SlimeCore and SlimeCore-loaded datapacks. it is primarily the responsibilty of your chosen frontend's author to document its usage. With that said, most frontends should share a base level of functionality and adhere to the concepts covered in [Key Concepts](#essential-concepts).

If you just want to get started quickly, you can install [Scdev](https://github.com/sixslimemc/scdev), a minimal frontend created by the author of SlimeCore, and skip to the [Using Scdev](#using-scdev) section.


## Basic Troubleshooting



---

## Using Scdev

This section explains common operations using the [Scdev](https://github.com/sixslimemc/scdev) frontend.

### Scdev Messages

Scdev provides necessary information through chat messages, sent to players with the `scdev.listen` tag.

```mcfunction
tag @s add scdev.listen
```

#### Rebuild Messages

Scdev sends messages upon [rebuild](#builds) (usually when datapacks change).

If a rebuild fails, the messages will explain the reason. (See [Rebuild Errors](#rebuild-errors) for a full list of erros that can occur and their fixes.)

If rebuilding starts but never seems to finish, see [this section](#unfinished-or-very-long-rebuilding).

#### Load Messages

Scdev sends messages upon [load](#loading) (e.g. `/reload`).

The messages contains a list of all datapacks enabled (i.e. in the [current build](#builds)), as well as their entrypoints, in the order that they are loaded/called. (Entrypoint list is primarily for datapack development is not necessary administration knowledge.)

If a datapack is not on this list but you think it should be (or vice versa), a rebuild may have failed. Check the rebuild messages (above/before the load messages).

A "Loading finished." message should be sent after each load. If not, it is an indication of an [unfinished load](#unfinished-loading).

### Managing 

TODO: make Scdev better.