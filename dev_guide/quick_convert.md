# Quick Convert

Make a backup of your datapack before making any changes.

## 1. Make a Pack ID

Your datapack must only have one namespace where it defines **new** resources in. The name of this namespace is referred to as your datapack's **pack ID**.

*If a resource (registry item/file) would not exist without your datapack (e.g. creating a block tag `#foo:my_blocks`), than it is new; if your datapack modifies/overwrites an existing resource (e.g. appending to block tag `#minecraft:infiniburn_overworld`), than it is not considered new, and may stay as-is.*

It is likely that your existing datapack already only has one namespace where it defines new resources in; if that is the case, you may skip to the next section.

If your existing datapack defines new resources in more than one namespace, you have the following options:
- Move all newly defined resources into a single namespace--recommended if resources are tightly coupled.
- Split your existing datapack into multiple datapacks--recommended if your namespaces function independently or have a one-way dependence flow.

*If your pack ID just so happens to be `slimecore`, you must change it to something else.*

## 2. Migrate Function Tags

Move the contents of your datapack's `#minecraft:load` function tag to `#<pack ID>:load`.

Move the contents of your datapack's `#minecraft:tick` function tag to `#<pack ID>:entrypoint/main`. For each function specified in your pack's new `#<pack ID>:entrypoint/main`, make it schedule itself every tick (add the line `schedule function <this function> 1t`).

*Your datapack must no longer write to `#minecraft:load` or `#minecraft:tick`.*

