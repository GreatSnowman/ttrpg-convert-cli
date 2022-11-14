# Json 5e convert CLI 
![GitHub all releases](https://img.shields.io/github/downloads/ebullient/json5e-convert-cli/total?color=success)

<table><tr>
<td>Jump</td>
<td><a href="#download-and-run">⬇ Download</a></td>
<td><a href="#build-and-run-optional">⚙️ Build</a></td>
<td><a href="#conventions">📝 Conventions</a></td>
</tr><tr>
<td><a href="#recommended-plugins">🔌 Plugins</a></td>
<td><a href="#use-with-5etools-json-data">📖 5etools Data</a></td>
<td><a href="#templates">🎨 Templates</a></td>
<td><a href="#changes-that-impact-generated-templates-and-files">🚜 Migration</a></td>
</tr></table>

I use [Obsidian](https://obsidian.md) to keep track of my campaign notes. This project parses json sources for materials that I own from the 5etools mirror to create linked and formatted markdown that I can reference in my notes.

> 🔥 **1.0.12 makes significant changes** to file names, and minor changes for custom templates. [See details below](#changes-that-impact-generated-templates-and-files).

## Download and run

1. Install JBang: https://www.jbang.dev/documentation/guide/latest/installation.html

2. Install the pre-built release: 

    ```shell
    jbang app install --name 5e-convert --force --fresh https://github.com/ebullient/json5e-convert-cli/releases/download/1.0.17/json5e-convert-cli-1.0.17-runner.jar
    ```
    
    If you want the latest unreleased snapshot: 
    
    ```shell
    jbang app install --name 5e-convert --force --fresh https://jitpack.io/dev/ebullient/json5e-convert-cli/199-SNAPSHOT/json5e-convert-cli-199-SNAPSHOT-runner.jar
    ```    

    There may be a pause if you download the snapshot, as it is rebuilt on demand.

    > 🔹 Feel free to use an alternate alias by replacing the value specified as the name: `--name 5e-convert`, and adjust the commands shown below accordingly.

3. Verify the install by running the command: 

    ```shell
    5e-convert --help
    ```

## Build and run (optional)

1. Clone this repository
2. Build this project: `quarkus build` or `./mvnw install`
3. Verify the build: `java -jar target/json5e-convert-cli-199-SNAPSHOT-runner.jar --help`

To run commands listed below, either: 

- Replace `5e-convert` with `java -jar target/json5e-convert-cli-199-SNAPSHOT-runner.jar`, or
- Use JBang to create an alias that points to the built jar: 

    ```shell
    jbang app install --name 5e-convert --force --fresh ~/.m2/repository/dev/ebullient/json5e-convert-cli/199-SNAPSHOT/json5e-convert-cli-199-SNAPSHOT-runner.jar
    ```

    > 🔹 Feel free to use an alternate alias by replacing the value specified as the name: `--name 5e-convert`, and adjust the commands shown below accordingly.

## Conventions

- **Links.** Documents generated by this plugin will use markdown links rather than wiki links. A css snippet can make these links less invasive in edit mode by hiding the URL portion of the string.

- **File names.** To avoid conflicts and issues with different operating systems, all file names are slugified (all lower case, symbols stripped, and spaces replaced by dashes). This is a familiar convention for those used to jekyll, hugo, or other blogging systems.

  File names for resources outside of the core books (PHB, MM, and DMG) have the abbreviated source name appended to the end to avoid file collisions. 

- **Organization.** Files are generated in two roots: `compendium` and `rules`. The location of these roots is configurable (see below).   

  The following directories may be created in the `compendium` directory depending on what sources you have enabled: `backgrounds`, `bestiary` (with contents organized by monster type), `classes` (separate documents for classes and subclasses), `deities`, `feats`, `items`, `names`, `races`, and `spells`.

- **Styles.** Every document has a `cssclass` attribute that you can use to further tweak how page elements render: `json5e-background`, `json5e-deity`, `json5e-monster`, `json5e-class`, `json5e-feat`, `json5e-item`, `json5e-names`, `json5e-race`, and `json5e-spell`.

  `css-snippets` has some snippets you can use to customize elements of the compendium.

## Recommended plugins 

- **[Admonitions](obsidian://show-plugin?id=obsidian-admonition)**: One of the templates for rendering monsters uses the code-style format supported by the admonitions plugin to render human-readable text (and avoid blockquote line wrapping).
  - Create a custom admonition called `statblock` (I recommend a dragon icon)
  - Create a custom admonition called `flowchart` (I recommend a map icon)
  - Either set the appearance/icon for the style in the plugin, or use/modify [this snippet](css-snippets/admonition_callout.css) to style the admonition.
  - The [compendium snippet](css-snippets/compendium.css) defines styles for the contents of the statblock, including setting the size of the token image based on the #token anchor in the link.

- **[Force note view mode by front matter](obsidian://show-plugin?id=obsidian-view-mode-by-frontmatter)**: I use this plugin to treat these generated notes as essentially read only. Specifically, I ensure the plugin has the following options enabled: "Ignore open files" (so that if I have toggled to edit mode, it doesn't fight with me over it), and "Ignore force view when not in front matter" (so that the setting isn't applied to documents that don't have the front matter tag).

- **[TTRPG Statblocks](obsidian://show-plugin?id=obsidian-5e-statblocks)**: Templates for rendering monsters can define a `statblock` in the document body or provide a full or abridged yaml monster in the document header to update monsters in the plugin's bestiary.
  - Note that in the process of creating this converter, I've discovered some corner cases that the TTRPG plugin does not handle well. I've adapted as best I can to work within what the TTRPG statblock format understands. Information will be rendered correctly in the plain statblock text for these cases.

- **[Initiative Tracker](obsidian://show-plugin?id=initiative-tracker)**: Templates for rendering monsters can include information in the header to define monsters that initiative tracker can use when constructing encounters.

## Use with 5eTools JSON data

1. Download a release of the 5e tools mirror, or create a shallow clone of the repo (which can/should be deleted afterwards):

    ```shell
    git clone --depth 1 https://github.com/5etools-mirror-1/5etools-mirror-1.github.io.git
    ```

2. Invoke the CLI. In this first example, let's generate indexes and use only SRD content:

    ```shell
    5e-convert \
      --index \
      -o dm \
      5etools-mirror-1.github.io/data
    ```

    - `--index` Create `all-index.json` containing all of the touched artifact ids, and `src-index.json` that shows the filtered/allowed artifact ids. These files are useful when tweaking exclude rules (as shown below).
    - `-o dm` The target output directory. Files will be created in this directory.

    The rest of the command-line specifies input files: 

    - `5etools-mirror-1.github.io/data` Path to the data directory containing 5etools files (a clone or release of the mirror repo)

3. Invoke the command again, this time including sources and custom items:

    ```shell
    5e-convert \
        --index \
        -o dm \
        -s PHB,DMG,SCAG \
        5etools-mirror-1.github.io/data \
        5etools-mirror-1.github.io/data/adventure/adventure-lox.json \
        5etools-mirror-1.github.io/data/book/book-aag.json \
        my-items.json dm-sources.json
    ```
    
    - `-s PHB,DMG,SCAG` Will include content from the Player's Handbook, the Dungeon Master's Guide, and the Sword Coast Adventurer's Guide, all of which I own. 
        > 🔸 **Source abbreviations** are found in the [source code (around line 138)](https://github.com/ebullient/json5e-convert-cli/blob/main/src/main/java/dev/ebullient/json5e/tools5e/CompendiumSources.java)
     
    - Books (`/book/book-aag.json`) and adventures (`/adventure/adventure-lox.json`) should be listed explicitly
    - `my-items.json` Custom items that I've created for my campaign that follow 5etools JSON format.
    - `dm-sources.json` Additional parameters (shown in detail below)

> 💭 I recommend running the CLI against a separate directory, and then using a comparison tool of your choice to preview changes before you copy or merge them in.
> 
> You can use `git diff` to compare arbitrary directories:
> ```
> git diff --no-index vault/compendium/bestiary generated/compendium/bestiary
> ```

### Additional parameters

I use a json file to provide detailed configuration for sources, as doing so with command line arguments becomes tedious and error-prone. I use something like this:

```json
{
  "from": [
    "AI",
    "PHB",
    "DMG",
    "TCE",
    "LMoP",
    "ESK",
    "DIP",
    "XGE",
    "FTD",
    "MM",
    "MTF",
    "VGM"
  ],
  "paths": {
    "rules": "/compendium/rules/"
  },
  "excludePattern": [
    "race|.*|dmg"
  ],
  "exclude": [
    "monster|expert|dc",
    "monster|expert|sdw",
    "monster|expert|slw"
  ]
}
```

- `from` defines the array of sources that should be included. Only include content from sources you own. If you omit this parameter (and don't specify any other sources on the command line), this tool will only include content from the SRD.  

    > 🔸 **Source abbreviations** are found in the [source code (around line 138)](https://github.com/ebullient/json5e-convert-cli/blob/main/src/main/java/dev/ebullient/json5e/tools5e/CompendiumSources.java)

- `paths` allows you to redefine vault paths for cross-document links, and to link to documents defining conditions, and weapon/item properties. By default, items, spells, monsters, backgrounds, races, and classes are in `/compendium/`, while files defining conditions and weapon properties are in `/rules/`. You can reconfigure either of these path roots in this block: 

    ```json
    "paths": {
      "compendium": "/compendium/",
      "rules": "/rules/"
    },
    ```
    > 🔹 Note: the leading slash indicates the path starting at the root of your vault.

- `exclude` and `excludePattern`: Exclude a single identifier (as listed in the generated index files), or all identifiers matching a pattern. In the above example, I'm excluding all of the race variants from the DMG, and the monster-form of the expert sidekick from the Essentials Kit. As it happens, I own these materials, but I don't want these variants in the formatted bestiary.

- `include` (as of 1.0.13): Include a single identifier (as listed in the generated index files). 
This allows you to include a specific resource without including the whole source and excluding everything else. Useful for single resources (classes, backgrounds, races, items, etc.) purchased from D&D Beyond. To include the Changeling race from _Mordenkainen Presents: Monsters of the Multiverse_, for example, you would add the folowing: 

    ```json
    "include": [
        "race|changeling|mpmm"
    ]
    ```

- `convert` (as of 1.0.18): specify books or adventures to import into the compendium (which will allow cross-linking, etc.). Either provide the full relative path to the adventure or book json file, or specify its Id (as found in the [source code (around line 138)](https://github.com/ebullient/json5e-convert-cli/blob/main/src/main/java/dev/ebullient/json5e/tools5e/CompendiumSources.java)): 

    ```json
    "convert": {
        "adventure": [
            "WBtW",
            "tftyp-wpm", 
        ],
        "book": [
            "5etools-mirror-1.github.io/data/book/book-phb.json"
        ]
    }
    ```

Note that some adventures, like _Tales from the Yawning Portal_, are treated as a collection of standalone modules. The generated index contains these as `reference` items, but it can help you find the sources you own. The three sources shown in the example above are listed in the index as `reference|adventure-wbtw`, `reference|adventure-tftyp-wpm`, and `reference|book-phb` respectively.


#### Additional example

To generate player-focused reference content for a Wild Beyond the Witchlight campaign, I constrained things further. I am pulling from a smaller set of sources. I included Elemental Evil Player's Companion (Genasi) and Volo's Guide to Monsters (Tabaxi), but also used `exclude` and `excludePattern` to remove elements from these sourcebooks that I don't want my players to use in this campaign (some simplification for beginners). 

The JSON looks like this:

```json
{
  "from": [
    "PHB",
    "DMG",
    "XGE",
    "TCE",
    "EEPC",
    "WBtW",
    "VGM"
  ],
  "includeGroups": [
    "familiars"
  ],
  "excludePattern": [
    ".*sidekick.*",
    "race|.*|dmg",
    "race|(?!tabaxi).*|vgm",
    "subrace|.*|aasimar|vgm",
    "item|.*|vgm",
    "monster|.*|tce",
    "monster|.*|dmg",
    "monster|.*|vgm",
    "monster|.*|wbtw",
    "monster|animated object.*|phb"
  ],
  "exclude": [
    "race|aarakocra|eepc",
    "feat|actor|phb",
    "feat|artificer initiate|tce",
    "feat|athlete|phb",
    "feat|bountiful luck|xge",
    "feat|chef|tce",
    "feat|dragon fear|xge",
    "feat|dragon hide|xge",
    "feat|drow high magic|xge",
    "feat|durable|phb",
    "feat|dwarven fortitude|xge",
    "feat|elven accuracy|xge",
    "feat|fade away|xge",
    "feat|fey teleportation|xge",
    "feat|fey touched|tce",
    "feat|flames of phlegethos|xge",
    "feat|gunner|tce",
    "feat|heavily armored|phb",
    "feat|heavy armor master|phb",
    "feat|infernal constitution|xge",
    "feat|keen mind|phb",
    "feat|lightly armored|phb",
    "feat|linguist|phb",
    "feat|lucky|phb",
    "feat|medium armor master|phb",
    "feat|moderately armored|phb",
    "feat|mounted combatant|phb",
    "feat|observant|phb",
    "feat|orcish fury|xge",
    "feat|piercer|tce",
    "feat|poisoner|tce",
    "feat|polearm master|phb",
    "feat|prodigy|xge",
    "feat|resilient|phb",
    "feat|second chance|xge",
    "feat|shadow touched|tce",
    "feat|skill expert|tce",
    "feat|slasher|tce",
    "feat|squat nimbleness|xge",
    "feat|tavern brawler|phb",
    "feat|telekinetic|tce",
    "feat|telepathic|tce",
    "feat|weapon master|phb",
    "feat|wood elf magic|xge",
    "item|iggwilv's cauldron|wbtw"
  ]
```

## Templates

This application uses the [Qute Templating Engine](https://quarkus.io/guides/qute). You can make simple customizations to markdown output by copying a template from `src/main/resources/templates`, making the desired modifications, and then specifying that template on the command line.

```
5e-convert 5etools \
  --background src/main/resources/templates/background2md.txt \
  --index -o dm dm-sources.json ~/git/dnd/5etools-mirror-1.github.io/data my-items.json
```

> 🔹 Not everything is customizable. In some cases, formatting headings, indenting and organizing text accurately is easier to do inline as a big blob. The example templates show what is available to tweak.

### Built-in / example templates

- [Default templates](https://github.com/ebullient/json5e-convert-cli/tree/main/src/main/resources/templates)

Of particular note are the varied monster templates: 

- Admonition codeblock: [monster2md.txt](https://github.com/ebullient/json5e-convert-cli/tree/main/src/main/resources/templates/monster2md.txt)
- Admonition codeblock with alternate score layout: [monster2md-scores.txt](https://github.com/ebullient/json5e-convert-cli/tree/main/src/main/resources/templates/monster2md-scores.txt)
- TTRPG statblock in the body: [monster2md-yamlStatblock-body.txt](https://github.com/ebullient/json5e-convert-cli/tree/main/src/main/resources/templates/monster2md-yamlStatblock-body.txt)
- Admonition codeblock in the body with minimal TTRPG/Initiative tracker YAML metadata in the header: [monster2md-yamlStatblock-header.txt](https://github.com/ebullient/json5e-convert-cli/tree/main/src/main/resources/templates/monster2md-yamlStatblock-header.txt)

## Changes that impact generated templates and files

## 1.0.18: You can put more things in json input now!

Use `convert` to import source text for books and adventures that you own: 

```json
  "convert": {
    "adventure": [
      "WBtW"
    ],
    "book": [
      "PHB"
    ]
  }
```

Specify templates in json:

```json
  "template": {
    "background": "path/to/template.txt",
  }
```

Be careful of paths here. Relative paths will be resolved depending on where the command is run from. Absolute paths will be machine specific (most likely). Use forward slashes for path segments, even if you're working on windows. 

You can place this configuration one file or several, your choice. 

### 1.0.16: Sections in Spell text

Text for changes to spells at higher levels is added to spells a little differently depending on how complicated the spell is.

Some spells effectively have subsections. Create or Destroy Water, from the PHB, has one subsection describing how water is created, and another describing how it is destroyed. In many layouts, there is just a bit of bold text to visually highlight this information. I've opted to make these proper sections (with a heading) instead, because you can then embed/transclude just the variant you want into your notes where that is relevant.

If a spell has sections, then "At Higher Levels" will be added as an additional section. Otherwise, it will be appended with `**At Higher Levels.**` as leading eyecatcher text.

The [default spell template](https://github.com/ebullient/json5e-convert-cli/tree/main/src/main/resources/templates/spell2md.txt) has also been amended. It will test for sections in the spell text, and if so, now inserts a `## Summary` header above the Classes/Sources information, to ensure that the penultimate section can be embedded cleanly.

### 1.0.15: Flowcharts, optfeature in text, styled rows

- `optfeature` text is rendered (Tortle package)
- `flowcharts` is rendered as a series of `flowchart` callouts  
    Use the admonition plugin to create a custom `flowchart` callout with an icon of your choice.
- The adventuring gear tables from the PHB have been corrected

### 1.0.14: Ability Scores

As shown in [monster2md-scores.txt](https://github.com/ebullient/json5e-convert-cli/tree/main/src/main/resources/templates/monster2md-scores.txt), you can now access ability scores directly to achieve alternate layouts in templates, for example: 

```
- STR: {resource.scores.str} `dice: 1d20 {resource.scores.strMod}`
- DEX: {resource.scores.dex} `dice: 1d20 {resource.scores.dexMod}`
- CON: {resource.scores.con} `dice: 1d20 {resource.scores.conMod}`
- INT: {resource.scores.int} `dice: 1d20 {resource.scores.intMod}`
- WIS: {resource.scores.wis} `dice: 1d20 {resource.scores.wisMod}`
- CHA: {resource.scores.cha} `dice: 1d20 {resource.scores.chaMod}`
```

### 1.0.13: Item property tags are now sorted

Property tags on items are now sorted (not alphabetically) to stabilize their order in generated files. This should be a one-time bit of noise as you cross this release (using a version before to using some version after).

### 🔥 1.0.12: File name changes

Each file name will now contain an abbreviation of the primary source to avoid conflicts (for anything that does not come from phb, mm, dmg).

***If you use the Templater plugin***, you can use [a templater script](https://github.com/ebullient/json5e-convert-cli/blob/main/migration/json5e-cli-renameFiles-1.0.12.md) to rename files in your vault before merging with freshly generated content. View the contents of the template before running it, and adjust parameters at the top as necessary.

### 🔥 1.0.12: Deity symbols and Bestiary Tokens

Symbols and tokens have changed in structure. Custom templates will need a bit of adjustment.

For bestiary tokens:

```
{#if resource.token}
![{resource.token.caption}]({resource.token.path}#token){/if}
```

For deities:

```
{#if resource.image}
![{resource.image.caption}]({resource.image.path}#symbol){/if}
```

## Other notes

This project uses Quarkus, the Supersonic Subatomic Java Framework. If you want to learn more about Quarkus, please visit its website: https://quarkus.io/.

This project is a derivative of [fc5-convert-cli](https://github.com/ebullient/fc5-convert-cli), which focused on working to and from FightClub5 Compendium XML files. It has also stolen some bits and pieces from [pockets-cli](https://github.com/ebullient/pockets-cli).
