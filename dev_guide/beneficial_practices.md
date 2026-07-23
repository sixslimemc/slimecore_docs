# Beneficial Practices

From the [Mission Statement](../description.md#mission-statement):

> The primary goal of SlimeCore is to support a community-driven, decentralized datapack ecosystem that is accessible to all datapack users and developers.

SlimeCore's loading system provides the framework to achieve this goal, but the content of datapacks--their functional compatibility and usability--matters just as much, if not more. Even so, SlimeCore intentionally does not try and control such elements, as they are things that *you*, the datapack developer(s), should be in control over.

That said, this page contains a handful of development tips, guidelines, and practices that are not strictly defined or required by SlimeCore, but may aide in increasing a datapack's compatibility and/or usability.

## Namespacing

Because SlimeCore essentially garuntees that a datapack's pack ID is unique within the world it is installed in (at least respective to other SlimeCore-loaded datapacks), pack IDs can be used to **namespace** (prefix) the names/identifiers of in-game artifacts defined by datapacks (NBT storage data, scoreboard objectives, entity tags, etc.), similar to how files/resources are inherently namespaced by your pack ID. This greatly reduces the chance of naming conflicts across datapacks.

Examples of effective namespacing within a datapack:
- Only using NBT storage locations that start with `<pack ID>:`.
- Only creating scoreboard objectives that start with `<pack ID>.`
- Only using entity tags that start with `<pack ID>.`.
- Only adding data to the `minecraft:custom_data` component at paths that start with `<pack ID>.` (`{<pack ID>:{...}}`).

In general, if you can choose the name of a technical identifier, it should be namespaced.

## Public and Private Resources

Datapacks should make it clear in their documentation--and in their structure, if possible--which resources are meant to be accessible by other datapacks and which ones are not.

For resources that are files, a simple and effective approach is to have a consistently-named directory in each registry that exclusively contains all private resources of that registry (e.g. `<registry>/private/...` or `<registry>/_/...`).

For in-game artifacts, one approach is to prefix identifiers with `_` if they are meant to be private, in a similar fashion to [namespacing](#namespacing), (`_` in particular works well alongside namespacing because pack IDs cannot start with `_`).

Regardless of any scheme used, the distinction between public and private resources should be **documented**.

## Entrypoint Separation

As stated in [Entrypoints](./full_guide.md#entrypoints):

> Defining a single entrypoint may be sufficient for most datapacks, but if your datapack does multiple conceptually independent blocks of work in its tick loop, consider defining multiple entrypoints and giving each block of work its own entrypoint.

To provide a practical example, imagine a datapack that converts all zombies and skeletons into some custom mob(s) (every tick), but also runs a tick loop on all instances of the custom mob(s) for its behavior. While both of these things *can* be implemented in a single looping entrypoint, they are conceptually and functionally independent from eachother. In this case, it would be a good idea to create 2 entrypoints, one for the conversion (e.g. ID `convert`), and the other for the behavior loop (e.g. ID `behavior`). Along with adding general clarity, having 2 entrypoints allows other datapacks to order their own entrypoint/behavior inbetween them--for whatever reason they may have.

## Modularization

Without SlimeCore, it is a common/sensible practice to try and make datapacks "all-in-one", reducing the responsibility that comes with dependencies for both the developer and the user. However, as you may have already figured, this is exactly the responsibility that SlimeCore attempts to alleviate, aiming to provide opportunity for developers include cross-datapack interaction without fear.

If your datapack includes multiple conceptually independent features (or sets of features), consider splitting it into **modules** that users can install and enable independently. Further, if said modules share some resources or implementation between them, consider making the shared elements into a **library** (or set of libraries) that your modules use as a dependency.

That said, splitting you datapack into modules implies that it makes sense for a user to install some modules and not others. If doing so would lead to a diminished or nonsensical experience for the user, then it is best to keep your datapack unsplit (or split into larger modules). 

For example, imagine a datapack with pack ID `foo`, that drastically changes combat and PvE mechanics. It changes the behavior of all weapons and armor in the game, as well as the behavior of all hostile mobs. Given these features are not heavily interdependent on eachother, it would be reasonable to split them into modules with pack IDs (following [naming guidelines](./full_guide.md#pack-ids)): `foo-weapons`, `foo-armor`, and `foo-mobs` respectively, as well as `foo-lib` for shared resources.

## Defining Interfaces

## Library Discipline

## Hooks/Events