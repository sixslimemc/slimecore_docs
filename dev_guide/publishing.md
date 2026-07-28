# Publishing


## General Guidelines

SlimeCore supports a **decentralized** datapack publishing/distribution model, meaning you are free to use any service or method you'd like to publish and/or distribute your datapack, as long as you can provide a valid [`url`](#the-url-manifest-field) manifest field.

While is generally expected that you include the following in your datapack's primary information/landing page:
- The [SlimeCore badge](#the-slimecore-badge), or some indication that your datapack requires SlimeCore (and its required version).
- 
While not *strictly* required, it is expected that you include in your datapack's primary information/landing page that your datapack is loaded by SlimeCore (and which version of SlimeCore is required). Using the [SlimeCore badge](#the-slimecore-badge) makes this easy and is recommended for any page that supports it. It is also expected that you clearly present your datapack's pack ID and author ID--the recommended format being `<author ID>.<pack ID>`.

Aside from that, how you present, distribute, and publish your datapack is entirely up to you.

## The `url` Manifest Field

*See [this section](./full_guide.md#url) for a general description of the `url` manifest field.*

Due to the nature of the `url` manifest field, you must know in advance what your datapack's direct download URL will be before actually releasing it for download at said URL. This is a known pain point of SlimeCore, however, providing a direct download link in-game greatly benefits the user and any datapack developer intending to use your datapack as a dependency.

If your primary publishing platform gives you control over the direct download URL (e.g. [GitHub](#publishing-with-github)), then setting `url` is mostly straightforward. If not, the following options are recommended:
- Use a service other than your primary publishing platform to provide/mirror downloads. (e.g. Release primarily on Modrinth but use GitHub releases to provide download URLs).
- Set `url` to a redirect URL that you control, then make that URL redirect to your datapack's actual direct download URL after releasing it. The redirect URL should be active and working for at least as long as you intend the corresponding version of your datapack to be relevant.

## The SlimeCore Badge

The SlimeCore badge is a simple markdown badge that is used to indicate that your datapack is SlimeCore-loaded, as well as the SlimeCore version required. The badge can be copied-and-pasted from [here](../BADGE.md).

The version of SlimeCore badge used should match the version specified by your datapack's 
Like most other badges, the SlimeCore badge should be included at (or near) the top of your datapack's primary information/landing page.

## Publishing With GitHub

The following are recommended guidelines if using GitHub as a publishing platform.

#### Repositories
- Each datapack should have its own repository.
- The contents of the repository should be the datapack itself (`pack.mcmeta` should be at the top level in the repository). 
- The name of the repository should be in the format `<author ID>.<pack ID>` or alternatively just `<pack ID>` if the owning GitHub profile/organization name matches the datapack's author ID.

#### README.md
- Your repository should have a `README.md` that includes the following:
    - The [SlimeCore badge](#the-slimecore-badge).
    - Your datapack's display name (matching the manifest field [`display.name`](./full_guide.md#display)).
    - Your datapack's ID in the format `<author ID>.<pack ID>`.
    - If the datapack is intended as a [SlimeCore frontend](../admin_guide/key_concepts.md#frontend-datapacks), its dependencies, their versions, and download/version links should be explicitly provided.

#### Docs/Info
- For datapacks with simple documentation needs, a `docs/` folder may be created for user/developer documentation in markdown format.
- For datapacks with larger documentation/wiki needs, you may alternatively link to an external page in the `README.md`.

#### Releases
- Each public version of the datapack should have its own release.
- Each release should have a tag; both the tag and release name should be in the format `v<major version>.<minor version>.<patch version>`.
- The "binary" attached to the release should be the zipped datapack with the name matching `<author ID>.<pack ID>.<major version>.<minor version>.<patch version>.zip`
    - This zip file should function correctly if put directly in a world's `datapacks/` folder; it should not have to be extracted.
    - The zip file should include things like the `README.md`, `docs/`, and `LICENSE.md` if applicable.
- It is a courtesy to indicate the supported [pack formats](https://minecraft.wiki/w/Pack_format) in release descriptions.

#### Manifest Data
- If using the recommended release format specified [above](#releases), the manifest [`url`](./full_guide.md#url) field should match the format `https://github.com/<username/org>/<repo>/releases/download/v<major version>.<minor version>.<patch version>/<author ID>.<pack ID>.<major version>.<minor version>.<patch version>.zip`.
- Unless you have a dedicated alternative, the [`display.links.info`](./full_guide.md#display) field should point to the repository's main page (`https://github.com/<username/org>/<repo>`).
- The [`display.links.versions`](./full_guide.md#display) field should point to the repository's "releases" page (`https://github.com/<username/org>/<repo>/releases`).

#### Alternative `url` Values
- If for whatever reason you do not use the release format specified [above](#releases), the general format of GitHub release direct download URLs are `https://github.com/<username/org>/<repo>/releases/download/<tag>/<file>`

## Publishing With Modrinth

> TODO: yea getting the manifest `url` using modrinth is crazy. Should def create a helper program.
