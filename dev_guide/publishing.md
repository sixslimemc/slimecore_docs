# Publishing


## General Guidelines

SlimeCore supports a **decentralized** datapack publishing/distribution model, meaning you are free to use any service or method you'd like to publish and/or distribute your datapack, as long as you can provide a [**direct download URL**](./full_guide.md#url) to your datapack's zip file.

While not *strictly* required, it is expected that you include in your datapack's primary information/landing page that your datapack is loaded by SlimeCore (and which version of SlimeCore is required). Using the [SlimeCore badge](#the-slimecore-badge) makes this easy and is recommended for any page that supports it. It is also expected that you clearly present your datapack's pack ID and author ID--the recommended format being `<author ID>.<pack ID>`.

Aside from that, how you present, distribute, and publish your datapack is entirely up to you.

## The SlimeCore Badge

The SlimeCore badge is a simple markdown badge that is used to indicate that your datapack is SlimeCore-loaded, as well as the SlimeCore version required. The badge can be copied-and-pasted from [here](../BADGE.md).

Like most other badges, the SlimeCore badge should be included at (or near) the top of your datapack's primary information/landing page.

## Publishing With GitHub

The following are some recommended guidelines if using GitHub as a publishing platform.

#### The Repository
- Each datapack should have its own repository.
- The contents of the repository should be the datapack itself (`pack.mcmeta` should be at the top level in the repository). 
- The name of the repository should be in the format `<author ID>.<pack ID>` or alternatively just `<pack ID>` if the owning GitHub profile/organization name matches the author ID.

#### README.md
- Your repository should have a `README.md` that includes the following:
    - The [SlimeCore badge](#the-slimecore-badge) at the very top.
    - Your datapack's display name (matching the manifest field [`display.name`](./full_guide.md#display)).
    - Your datapack's ID in the format `<author ID>.<pack ID>`.
    - If the datapack is intended as a [SlimeCore frontend](../admin_guide/key_concepts.md#frontend-datapacks), its dependencies, their versions, and download/version links should be explicitly provided.

#### Docs/Info
- You may include a `docs/` folder in the datapack that contains user and/or developer documentation in markdown format.
- For datapacks with larger documentation/wiki needs, you may alternatively link to an external page in the `README.md`.

#### Releases
- Each public version of the datapack should have its own release.
- Each release should have a tag; both the tag and release name should be in the format `v<major version>.<minor version>.<patch version>`.
- The "binary" attached to the release should be the zipped datapack with the name matching `<author ID>.<pack ID>.<major version>.<minor version>.<patch version>.zip`
    - This zip file should function correctly if put directly in a world's `datapacks/` folder; it should not have to be extracted.
    - The zip file should include things like the `README.md`, `docs/`, and `LICENSE.md` if applicable.
- It is a courtesy to indicate the supported [pack formats](https://minecraft.wiki/w/Pack_format) in the release description.

#### Manifest Data
- If using the recommended release format specified [above](#releases), the manifest [`url`](./full_guide.md#url) field should match the format `https://github.com/<username/org>/<repo>/releases/download/v<major version>.<minor version>.<patch version>/<author ID>.<pack ID>.<major version>.<minor version>.<patch version>.zip`.
- Unless you have a dedicated alternative, the [`display.links.info`](./full_guide.md#display) field should point to the repository's main page (`https://github.com/<username/org>/<repo>`).
- The [`display.links.versions`](./full_guide.md#display) field should point to the repository's "releases" page (`https://github.com/<username/org>/<repo>/releases`).

> TODO

## Publishing With Modrinth

> TODO: yea getting the manifest `url` using modrinth is crazy. Should def create a helper program.
