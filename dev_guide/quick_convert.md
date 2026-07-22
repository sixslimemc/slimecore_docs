# Quick Convert

Make a backup of your datapack before making any changes.

## 1. Identify Your Pack ID

Your datapack must only have 1 namespace where it defines **new** resources in. The name of this namespace is referred to as your **pack ID**.

*If a resource (registry item/file) would not exist without your datapack (e.g. creating a block tag `#foo:my_blocks`), than it is new; if your datapack modifies/overwrites an existing resource (e.g. appending to block tag `#minecraft:infiniburn_overworld`), than it is not considered new, and may stay as-is.*

