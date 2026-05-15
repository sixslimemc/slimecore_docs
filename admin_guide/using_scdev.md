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

Load summaries list all SlimeCore-loaded datapacks in the order that they are loaded, as well as all entrypoints/preload-entrypoints within those packs in their calling order.

*Example of a load summary:*
![Screenshot of load summary](../_assets/images/load_summary.png)