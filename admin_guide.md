# SlimeCore Admin Guide

From an administration point of view, SlimeCore is a datapack that loads and *manages* other datapacks. In order to manage (disable, enable, uninstall) SlimeCore-loaded packs, it must be done through SlimeCore. However, SlimeCore intentionally does not provide any player-facing features (e.g. chat messages, dialogs, user functions, etc.) on its own. Instead, it is designed such that *other* datapacks can provide such functionality, primarily so *you*, as the user, can choose how you interact with it. These datapacks are referred to as SlimeCore **frontends**. While not technically *strictly* required, a SlimeCore frontend is essential for easy administration. A frontend is installed like any other datapack.

[Scdev](https://github.com/sixslimemc/scdev) is a very minimal frontend/utility created by the author of SlimeCore; this guide will reference it for demonstration. However, any frontend should provide similar functionality with obvious steps--as long as you have one, you should be good to go.

*Sections with titles starting with "(Scdev)" are Scdev specific and should only be regarded if you are using Scdev.*

### (Scdev) Get Status Messages

World admins should have the tag `scdev.

## Installing Datapacks
To simply install and play with SlimeCore-loaded datapack(s), it is likely that all you need to do is make sure your world has the [SlimeCore datapack](https://github.com/sixslimemc/slimecore/releases) installed and enabled, and the datapack(s) should just work. However, if you have SlimeCore installed and your datapack(s) still do not work/load, it is most likely because they require the installation of other datapacks, **dependencies**. This is *okay*, SlimeCore helps you out it this.


