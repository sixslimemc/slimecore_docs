# Beneficial Practices

From the [Mission Statement](../description.md#mission-statement):

> The primary goal of SlimeCore is to support a community-driven, decentralized datapack ecosystem that is accessible to all datapack users and developers.

SlimeCore's loading system provides the framework to achieve this goal, but the content of datapacks--their functional compatibility and usability--matters just as much, if not more. These are elements that only you, the datapack developer(s), can control.

This page contains a handful of development tips, guidelines, and practices that are not strictly defined or required by SlimeCore, but may aide in increasing a datapack's compatibility and/or usability.

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

## Entrypoint Splitting

## Modularization

## Defining Interfaces

## Library Discipline

## Hooks/Events