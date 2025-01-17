---
title: 03. system.json
description: 
published: true
date: 2022-10-13T16:46:23.674Z
tags: 
editor: markdown
dateCreated: 2020-09-23T00:35:35.124Z
---

> **Updated for Foundry v10**
>
> This section of the system development tutorial has been updated for Foundry v10. Other pages in the tutorial may still be in progress.
{.is-info}

Once you've created your system, you'll need to make some changes to system.json. Here's what it looks like for the Boilerplate System by default:

```json
{
  "id": "boilerplate",
  "title": "Boilerplate",
  "description": "The Boilerplate system for FoundryVTT!",
  "version": "1.0.0",
  "compatibility": {
    "minimum": 10,
    "verified": "10.288",
    "maximum": 10
  },
  "authors": [{
    "name": "Asacolips"
  }],
  "esmodules": ["module/boilerplate.mjs"],
  "styles": ["css/boilerplate.css"],
  "scripts": [],
  "packs": [],
  "languages": [
    {
      "lang": "en",
      "name": "English",
      "path": "lang/en.json"
    }
  ],
  "gridDistance": 5,
  "gridUnits": "ft",
  "primaryTokenAttribute": "health",
  "secondaryTokenAttribute": "power",
  "url": "",
  "manifest": "",
  "download": "",
  "license": "https://example.com/LICENSE.txt",
  "changelog": "https://example.com/CHANGELOG.md",
  "bugs": "https://example.com/issues",
}

```

For a full breakdown of each of those properties head to [https://foundryvtt.com/article/system-development/](https://foundryvtt.com/article/system-development/). But there a few important ones to take a look at in detail:

* **id**: Your system's machine-safe name, such as `mysystemname` or `dnd5e`. In previous versions of Foundry, this was called `name`.
* **version**: Your system's version. This should match the git tag you create for this release if you're using git, and see the **Managing Releases** section below for more details on how to best manage versions for your system.
* **compatibility**: An object with your system's compatibility requirements. The properties on this object are:
	* **minimum**: The minimum core version of Foundry required to install this system. If there are API changes in Foundry itself that you have to make significant updates for, you'll want to update this number. This can be a general version like "10" or a specific version, like "10.288".
	* **verified**: The most recently tested version of Foundry for this system. If Foundry is newer than the version listed here, users will receive a warning when trying to install or use your system. You'll need to update this frequently, but it can be done through foundryvtt.com's package manager so that you don't have to create new releases. It's recommended to put specific versions here, like "10.288".
  * **maximum**: The maximum compatible version of Foundry this should be allowed to run in. For example, if you set this to "10", it can't be run in Foundry v11 at all. If you'd like to allow users to attempt to run this in future versions of Foundry, you can leave this field out.
* **esmodules**: An array of Javascript files to import as ES modules. If you need to add multiple files, you can do a comma separated list inside the `[]` brackets.
* **styles**: An array of CSS files to use for your system's styling.
* **scripts**: An array of Javascript files to include in your system. These files will not use the export/import syntax that ES modules use.
* **packs**: Compendium packs. Each pack will be an object where you specify the name, system, path, and entity type. The `dnd5e` system has several great examples of creating a compendium, and more info can be found at [https://foundryvtt.com/article/compendium/](https://foundryvtt.com/article/compendium/).
* **languages**: Any language definitions that you create will need to be listed here. Boilerplate System comes with an included `en.json` language file.
* **url**: The link to your system's homepage, such as the Github or Gitlab page for it.
* **manifest**: The raw link to this `system.json` file. Foundry uses this to find out information about your system for update and install purposes.
* **download**: The raw link to download a zip file of your system. This can be built via custom build tools, but Github and Gitlab both generate zip files from your master branch that you can use here.
* **license**: URL to your system's license information, if any. It's recommended that you provide some sort of license (such as GPL, MIT, or general copyright notices) to remove ambiguity in case others want to contribute to your code or take over maintenance in the future. For example, the Boilerplate system used in this tutorial is using the MIT license.

> **Managing Releases**
> To properly manage releases, you need to do three things:
> (1) Use the link to the latest version of your system in the `manifest` property of system.json. This will allow updates to be found by Foundry. If you have multiple release tracks such as `v9` and `v10` for versions compatible with the respective Foundry versions, use that in the manifest URL.
> (2) Use the link to the **tagged** version of your system in the package listing on foundryvtt.com when making a new release. This will allow Foundry find and install specific versions (which is important for users who don't update immediately and stay on the previous stable version).
> (3) Use the link to the **tagged** version of your system in the `download` property of your system.json, which allow the manifest to find and download a specific version (and works in tandem with #2 above about using tagged versions on foundryvtt.com).
>
> The example **system.json** snippet above shows an example of how this structure would look. Review it closely when working on your own system.json file during releases.
{.is-info}


---

* **Prev:** [Stuff to be aware of](https://foundryvtt.wiki/en/development/guides/SD-tutorial/SD02-Stuff-to-be-aware-of)
* **Next:** [template.json](https://foundryvtt.wiki/en/development/guides/SD-tutorial/SD04-templatejson)