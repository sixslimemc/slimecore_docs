# Using Scdev

This section exclusively covers interacting with SlimeCore through the [SCDev](https://github.com/sixslimemc/scdev) frontend.

## Getting Setup

SCDev interfaces entirely through chat messages, sending them to **listeners**. \
To make yourself a listener, add the tag `scdev.listener` to yourself.
```mcfunction
tag @s add scdev.listener
```

Verify that you are a listener by running `/reload`--you should recieve a [load summary](#load-summaries).

### Clickable Text

Many SCDev messages contain clickable/hoverable text. If you want more information about something mentioned in a message, try hovering and/or clicking on it.

## Load Summaries

Upon every world reload, a **load summary** is sent to listeners.

Load summaries include:
- **Preload Entrypoints:** All enabled preload entrypoints in their calling order
- **Packs:** All SlimeCore-loaded packs in their loading order
- **Entrypoints:** All enabled entrypoints in their calling order

For most administrative purposes, only the "Packs" section is relevant.

*Example of a load summary:*

![Screenshot of load summary](../_assets/images/load_summary.png)

If a [rebuild](./key_concepts.md#rebuilding) fails, the subsequent load summary will be supressed in order to bring attention to the [rebuild error message(s)](#rebuild-messages).

## Rebuild Messages

Upon [rebuilding](./key_concepts.md#rebuilding), a "Rebuilding..." message will be sent, followed by a "Rebuild success." message if rebuilding was successful. If rebuilding failed, a descriptive error message will be sent instead.

*Refer to [this section](./troubleshooting.md#rebuild-errors) for resolving rebuild errors.*

If "Rebuilding..." is sent but no messages are sent afterward, this may indicate an [unfinished rebuild](./troubleshooting.md#unfinished-loadingrebuilding). However, it is normal for the rebuild process to take [some time](./troubleshooting.md#very-long-rebuilding).

*Example of successful rebuild indication:*

![Screenshot of successful rebuild messages](../_assets/images/rebuild_success.png)

*All "Rebuild success." messages should be directly followed by a load summary.*

## Info Functions

### List Functions

###