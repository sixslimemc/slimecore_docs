# Beneficial Practices

From the [Mission Statement](../description.md#mission-statement):

> The primary goal of SlimeCore is to support a community-driven, decentralized datapack ecosystem that is accessible to all datapack users and developers.

While SlimeCore seeks to be a significant step toward this goal, intentional design in datapacks' content toward compatibility and usability can contribute just as much, if not more. Even so, SlimeCore does not try and control such elements, as they are things that *you*, the datapack developer(s), should be in control over.

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

## Configuration

Datapacks should generally allow users to configure content to their preferences via in-game methods, opposed to hard-coding them within the datapack. As a rule of thumb, if an element of your datapack can *easily* be made configurable/dynamic and it would not be unreasonable for a user to want to change it, then it should be configurable.

Some examples of likely easily configurable elements:
- Stats of custom items, mobs, effects, etc.
- Behavior that can be toggled on/off.
- Intensity of vfx/sfx.

Importantly, if your datapack is configurable, it should install with a sensible default/standard configuration already set--configuration should be an *option* to the user, not a *responsibility*.

A simple and effective approach for implementing configuration is to use an NBT storage location (e.g. `<pack ID>:config`) that users are allowed to directly modify. Store the default configuration values in that location on datapack install, and have your feature implementations read the values in that location.

## Hooks/Events

## Entrypoint Separation

From [Entrypoints](./full_guide.md#entrypoints):

> Defining a single entrypoint may be sufficient for most datapacks, but if your datapack does multiple conceptually independent blocks of work in its tick loop, consider defining multiple entrypoints and giving each block of work its own entrypoint.

To provide a practical example, imagine a datapack that converts all zombies and skeletons into some custom mob(s) (every tick), but also runs a tick loop on all instances of the custom mob(s) for its behavior. While both of these things *can* be implemented in a single looping entrypoint, they are conceptually and functionally independent from eachother. In this case, it would be a good idea to create 2 entrypoints, one for the conversion (e.g. ID `convert`), and the other for the behavior loop (e.g. ID `behavior`). Along with adding general clarity, having 2 entrypoints allows other datapacks to order their own entrypoint/behavior inbetween them--for whatever reason they may have.

## Modularization

Without SlimeCore, it is a common/sensible practice to try and make datapacks "all-in-one", reducing the responsibility that comes with dependencies for both the developer and the user. However, as you may have already figured, this is exactly the responsibility that SlimeCore attempts to alleviate.

If your datapack includes multiple conceptually independent features (or sets of features), consider splitting it into **modules** that users can install and enable independently. Further, if said modules share some resources or implementation between them, consider making the shared elements into a **library** (or set of libraries) that your modules use as a dependency.

That said, splitting you datapack into modules implies that it makes sense for a user to install some modules and not others; if doing so would lead to a diminished or nonsensical experience for the user, then it is best to keep your datapack unsplit (or split into larger modules). 

For example, imagine a datapack with pack ID `foo`, that drastically changes combat and PvE mechanics. It changes the behavior of all weapons and armor in the game, as well as the behavior of all hostile mobs. Given that these features are not heavily interdependent on eachother, it would be reasonable to split them into modules with pack IDs (following [naming guidelines](./full_guide.md#pack-ids)): `foo-weapons`, `foo-armor`, and `foo-mobs` respectively, as well as `foo-lib` for shared resources.

## Library Discipline

## Defining Interfaces

From [Abstract Interfaces](./full_guide.md#abstract-interfaces):

> Abstract interfaces represent "contracts" that are declared by one datapack, and must be fulfilled/implemented by another. The terms of said "contracts" are to be documented/explained by the author of the declaring datapack, abstract interfaces only *represent* them. Concretely, for every abstract interface that a datapack *declares*, **exactly one** other datapack must specify that it *implements* it.

While it is likely that abstract interfaces are not applicable in most datapacks, it is worth knowing when they can be useful, as well as how to go about designing an API for them.

Below is a simplified and focused dissection of the [DeathDef](https://github.com/sixslimemc/deathdef) datapack, which should provide a good example of an effective abstract interface.

### DeathDef Example

[DeathDef](https://github.com/sixslimemc/deathdef) is a datapack that provides an API for custom player-death behavior. It does this by disabling default death behavior (dropping items/xp), detecting when a player dies, then passing the death information as input (location, items, xp, etc.) to an *unimplemented function*, `death`. It defines one abstract interface; it is documented by DeathDef that, a datapack should implement the interface (in their manifest) *if and only if* they provide an implementation for `death`--this is the "contract" of the interface.

DeathDef does not care what happens when `death` is called; DeathDef just calls with the right inputs when the player dies. Conversely, the datapack that implements `death` only cares about making player-death behavior given the inputs; it does not need to worry about the details of death detection.

Concretely, DeathDef stores death information in NBT storage location `deathdef:abstract/in` just before calling the function tag `#deathdef:abstract/death`. The datapack that implements `death` adds its own internal function to `#deathdef:abstract/death` and uses the data stored in `deathdef:abstract/in` to provide a proper implementation.

Tying it all together now: because DeathDef defines an abstract interface, SlimeCore requires that exactly one datapack is installed/enabled that implements it, and given that the documented contract of the interface is adhered to (implementing `death`), there will never be any cases where player-death is not implemented, nor any cases where player-death is implemented multiple times.

Design wise, player-death is something that should reasonably have exactly one implementation, thus is a good candidate for an abstract interface. For cases where you want to allow *any* amount of external datapacks to provide implementation or response to an internal event, [hooks](#hooksevents) are better suited. 

In most cases, when you create a datapack that declares abstract interface(s), you should also create datapack(s) that provide "default" or "standard" implementations and reference them in the declaring datapack's documentation. This is so that, in the case that a datapack uses the declaring datapack as a dependency (for its other features) but does not implement the abstract interface(s), users have default implementations to fall back on. *For DeathDef, this default implementation is [DeathDefault](https://github.com/sixslimemc/deathdefault).*