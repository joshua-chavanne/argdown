# Release Notes 2019

## v1.1.0 (March 2019)

**Changes in:** @argdown/core, @argdown/node, @argdown/vscode, @argdown/sandbox, @argdown/language-server

This is a big release that extends the Argdown packages with a number of useful new features and configuration options like

- the new [GraphML export for yEd](#graphml-export),
- [closable groups](#closable-groups),
- support for [rank assignment](#dot-configuration-rank-assignment) in dot,
- improved [hover quick info](#vscode-hover-quick-info) in VSCode,
- a [dark Argdown theme](#vscode-new-themes) for VSCode

However by far the most effort was put into bug fixes and improvements in code quality and stability. Nearly all bug issues on Github are now closed and all three Argdown apps should have gotten better in handling invalid Argdown code.

### GraphML Export

Maps can now be exported to [GraphML](http://graphml.graphdrawing.org/). The exported maps can directly be used in the powerful and free [yEd](https://www.yworks.com/products/yed) graph editor. This makes it now possible to completely customize the graph layout and design of the map.

The map in GraphML will not be layouted, all nodes have the same position. After the import in yEd you have to apply a graph layout (this only takes a few clicks).

### Closable Groups

A group can be closed now to hide all its children. This is useful to reduce complexity in huge maps.

#### Usage with Data Flag:

```argdown
[s1]

<a4>
    -> <a2>

# A Closed Group  {isClosed: true}

[s1]
    - <a2> {isInGroup: true}
    + <a3>
```

#### Usage in Group Configuration:

The new `group.sections` setting allows to define which sections are closed and which are not groups:

```argdown
===
group:
    sections: {"A Closed Group": {isClosed: true}, "Just a Heading": {isGroup: false}}
===

# A Closed Group

<s1>: I am hidden.
    -> [s2]

# Just a Heading

[s2]: I am free!
```

This allows to use headings as groups without "polluting" the Argdown document with data flags.

#### Usage in Regroup Configuration:

```argdown
===
group:
    regroup: [{
            title: "first group",
            statements: ["s1"], arguments: ["a1"],
            children: [
                {
                    title: "some other group",
                    isClosed: true,
                    arguments: ["a2"]
                }
            ]
            }]
===


[s1]: I am a proud member of the first group
    - <a1>: I am a proud member of the first group
        - <a2>: The other group is much more exclusive!

<a3>: Who needs groups?
    -> [s1]
```

#### Ignoring Group Data flags

Group data flags can now be ignored by using `group.ignoreIsGroup` and `group.ignoreIsClosed`

### Dot Configuration: Rank Assignment

For the Dot export and VizJs map you can now use the `sameRank` option to declare that certain nodes should be at the same level in the layout (see this [Stackoverflow question](https://stackoverflow.com/questions/25734244/how-do-i-place-nodes-on-the-same-level-in-dot) for an example).

#### Usage of the `sameRank` Setting in Dot Configuration

```argdown
===
dot:
    sameRank:
        - {arguments: ["a1", "a2"], statements: ["s1", "s2"]}
===

<a1>
    - <a2>
    + <a3>

[s1]
    - [s2]
    + [s3]
```

#### Usage of the `rank` Property in Element Data

```argdown
<a1> {rank: "r1"}
    - <a2>  {rank: "r1"}
    + <a3>  {rank: "r2"}

[s1]  {rank: "r1"}
    - [s2]  {rank: "r1"}
    + [s3]  {rank: "r2"}
```

If you want to put nodes of different groups (clusters) into the same rank, you might want to try the `newrank` [graphviz setting](https://graphviz.gitlab.io/_pages/doc/info/attrs.html#d:newrank):

```argdown
===
dot:
    graphVizSettings:
        newrank: true
===

# New Rank Test

## Heading 1

<a1> {rank: "r1"}
    - <a2>  {rank: "r1"}
    + <a3>  {rank: "r2"}
    -> [s1]

## Heading 2

[s1]  {rank: "r1"}
    - [s2]  {rank: "r1"}
    + [s3]  {rank: "r2"}
```

See this [Stackoverflow question](https://stackoverflow.com/questions/6824431/placing-clusters-on-the-same-rank-in-graphviz) for more information.

### VizJs Configuration: Layout Engines

VizJs can now be configured to use any of its [layout engines](https://github.com/mdaines/viz.js/wiki/Supported-Graphviz-Features):

- circo
- dot (default)
- fdp
- neato
- osage
- twopi

At the moment this will not work on https:argdown.org/changes as the documentation still has to use an outdated VizJs version for technical reasons. If you want to try it out, copy the code below into the browser sandbox or into VSCode.

```argdown
===
vizJs:
    engine: circo
===

<a1>
    - <a2>
    + <a3>
        - <a6>
        - <a7>
        - <a8>
    - <a4>
    + <a5>
        - <a9>
        - <a10>
        - <a11>
```

### Dot/VizJs/Dagre Configuration: Changing the Node Width

The dot export/VizJs map and Dagre map now support two methods of changing the node width.

In both cases the Argdown app has to add line breaks to the node labels to force a certain node width. The node width is changed by customizing the method by which the line breaks are added.

#### The `charactersInLine` Setting

By default a line break is added after a certain maximum number of characters (respecting word boundaries). This behaviour is customized by using the `charactersInLine` setting. This simple method is fast and works surprisingly well in practice.

Here is an example of customizing the dot export with different `charactersInLine` settings for groups, arguments and nodes:

```argdown
===
dot:
    group:
        charactersInLine: 1000
    argument:
        minWidth: 0
        title:
            charactersInLine: 12
        text:
            charactersInLine: 12
    statement:
        minWidth: 0
        title:
            charactersInLine: 30
        text:
            charactersInLine: 30
===

# A group with a very very very long title that does not contain a line break even though it is really long

[A statement with a long title]: This statement states that it is a long statement, longer than many many other claims commonly used in a debate.
    - <An argument with a long title>: A description that goes on for a while and does not stop too soon. Here is another sentence. And another one.

```

Note that you have to define separate `charactersInLine` settings for the title labels and text labels of argument and statement nodes. This is necessary because you can also use different font sizes for them, so you might want to set `charactersInLine` lower if the font size is larger.

### The `measureLineWidth` Setting

Alternatively, you can try a more exact and slower method by turning on `measureLineWidth`. In this case the actual pixel width of each word is measured and a line break is added after a certain maximum number of pixels (once again respecting word boundaries).

This behaviour is customized by using the `lineWidth` setting. Here is an example of turning on and customizing `measureLineWidth` for the VizJs map:

```argdown
===
dot:
    measureLineWidth: true
    group:
        lineWidth: 1000
    argument:
        lineWidth: 80
        minWidth: 0
    statement:
        lineWidth: 220
        minWidth: 0
===

# A group with a very very very long title that does not contain a line break even though it is really long

[A statement with a long title]: This statement states that it is a long statement, longer than many many other claims commonly used in a debate.
    - <An argument with a long title>: A description that goes on for a while and does not stop too soon. Here is another sentence. And another one.
```

In this case there are no separate settings for title and text as the line width is measured by taking different font sizes into account.

For the text measurement we use the [string-pixel-width](https://github.com/adambisek/string-pixel-width) library to be able to measure text outside of the browser. Please note that the library only supports measuring a limited number of fonts, so this option might not work with your font. Please consult the library's [Readme](https://github.com/adambisek/string-pixel-width#readme) for further information.

### Dot Configuration: Using `minWidth` to Set a Uniform Minimum Node Width

In the dot export (and the VizJs map by extension) the `minWidth` property will be used as a minimum node width. If `minWidth = 0` the default GraphViz behaviour will be used, which means that the node width is set to `maxLineWidthInLabel + horizontalMargin`. This is how nodes widths were determined before in the Argdown VizJs map. This behaviour saves space, but it has the downside that nodes have different widths, depending on their label content. This lack of uniformity might make the map look messy.

In the new version, Argdown sets the minWidth for argument and statement nodes by default to `180` which means that the node width is now set to `max(180, maxLineWidthInLabel) + horizontalMargin`. If you want the old behaviour back, you have to set `minWidth` to 0.

Here is the new default behaviour:

```argdown
<a1>
    - <A veeeeeeeery long title>
```

Here is how you get the old default behaviour back:

```argdown
===
dot:
    argument:
        minWidth: 0
===

<a1>
    - <A veeeeeeeery long title>
```

### Dot Configuration: Setting Margins for Nodes and Groups

You can now use the `margin` dot setting to customize the sizing of statements, arguments and groups in dot (and the VizJs map):

```argdown
===
map:
    argumentLabelMode: title
dot:
    argument:
        minWidth: 0
        margin: "0.8, 0"
    statement:
        minWidth: 0
        margin: "0, 0.8"
    group:
        margin: "80"
===

# A group

[s1]: A statement
    - <argument>: An argument
```

### GraphML Configuration: Node Size

Because the GraphML export is not used in a live preview, performance is not as important so it always uses the line measurement method.

In contrast to the Dot export and the Dagre map, node width can be set directly. Together with the `horizontalPadding` setting it determines the node's line width.

Here are the node size configuration options for GraphML:

```argdown
===
graphml:
    argument:
        width: 100
        horizontalPadding: 20
        verticalPadding: 10
    statement:
        width: 150
        horizontalPadding: 20
        verticalPadding: 10
    group:
        horizontalPadding: 20
        verticalPadding: 20
===

...
```

### Dot/GraphML/Dagre Configuration: Font Style

Font size, font and boldness of text in the Dot/GraphML exports and VizJs/Dagre maps can now be customized for groups, arguments and statements:

```argdown
===
dot:
    group:
        fontSize: 25
        font: impact
        bold: true
    statement:
        title:
            fontSize: 14
            bold: true
            font: impact
        text:
            font: times new roman
            fontSize: 12
    argument:
        title:
            fontSize: 16
            bold: true
            font: impact
        text:
            font: times new roman
            fontSize: 12
===

[Statement]: Some text
    - <Argument>: Some text
```

Please note that VizJs only supports a [small number of fonts](https://github.com/mdaines/viz.js/wiki/Caveats#Fonts) and the same is true for the library that is used for [text width measurement](#The-measureLineWidth-Setting).

### Color configuration: Custom Relation & Edge Colors

Relation colors can now be customized. Here is how you can turn all attack edges in your map pink:

```argdown
===
color:
    relationColors:
        attack: "#FFC0CB"
===

<a1>
    - <a2>
    - <a3>
```

### HTML Export: Creating a Header From Document Metadata

The HTML export now uses frontmatter metadata to create a document header with a title and optionally a subtitle, authors, date and abstract:

```argdown
===
title: A Document with a Header
subTitle: Having Fun With HTML
author: Christian Voigt
# if there are several authors you can use an array:
# author: ["Christian Voigt", "Gregor Betz"]
date: 14/3/2019
abstract: >
    Demonstrating the use
    of document meta data
    in the HTML export
===

...
```

If you want to define metadata, but not generate a HTML header section, you can deactivate this feature in the html settings:

```argdown
===
title: A Document Without a Header
author: Christian Voigt
html:
    createHeaderFromMetadata: false
===

...
```

### PDF Export Configuration

The PDF export for the VizJs map is now fixed. The size and padding of the pdf can now be changed. The map will be scaled accordingly.

```argdown
===
svgToPdf:
    width: 800
    height: 600
    padding: 20
===

...
```

For VSCode users: please note that only the currently visible part of the map will be visible in the pdf file. If you have zoomed in, some parts of the map will be cut off. Reload the map to make all nodes visible again and then export to pdf without zooming.

### Dot Configuration: `mapBgColor` Setting

To support VSCode dark themes the VizJs map's background color is now set to `transparent`. You can change the background color with the `mapBgColor` setting.

Here is how you turn your VizJs map background pink:

```argdown
===
dot:
    mapBgColor: "#FFC0CB"
===

[s1]
    + <a1>
        - <a2>
        - <a3>
```

### Changes in Section Assignment

The behaviour of the section assignment has slightly changed: If an equivalence class has no definition and an argument has no definition _and_ no premise-conclusion-structure the section of their first reference is assigned to them:

```argdown
# A section

[A statement without definition]

# Another section

[A statement without definition]
    - <An argument without definition>
```

Previoulsy no section was assigned in these cases.

### VSCode: New Themes

The VSCode Argdown extension now comes with two themes:

- Argdown Light, based on the default light theme
- Argdown Dark, based on the default dark theme

Additionally the extension's README now contains instructions how to use any VSCode theme with Argdown by adding custom token colors to your VSCode configuration.

The Argdown preview will now also use your current theme's colors for its styles, so that the extension will be seamlessly integrated into VSCode.

### VSCode: Hover Quick Info

Hovering with the mouse over statement and argument titles will show an improved quick info view that will show explicit and now also _implicit_ relations that can be derived from equivalence classes and premise-conclusion-structures.

### VSCode: Persistent Preview

The Argdown preview will now serialize its state and recreate it on VSCode restarts, allowing you to continue work where you left off. This also applies to its zoom state.

### Minor changes

- @argdown/vscode: added argdown.preview.minUpdateDelay setting so that the user can increase the delay between preview updates if performance is an issue
- @argdown/core: simple configuration data sanitization system for better stability in previews if configuration is invalid
- @argdown/core: checkResponseInputFields helper for easier response validation in plugins
- @argdown/core: added error code to plugin exceptions for easier testing of exceptions handling
- @argdown/language-server: increased general stability (exception handling)
- @argdown/sandbox: increased general stability of preview (catching exceptions), VizJs is now automatically reinstantiated if rendering fails
- @argdown/vscode: now uses Viz.Js inside the webview, hopefully improving performance
- @argdown/sandbox: now uses a web worker for the VizJsMap that should improve performance
- @argdown/map-views: refactoring of map views, changed Dagre labels from foreign-objects to pure svg to avoid browser bugs
- @argdown/vscode: using @argdown/map-views now, sharing code with @argdown/sandbox
- @argdown/sandbox: using @argdown/map-views now, sharing code with @argdown/vscode
- @argdown/core now exports deriveImplicitRelations helper for getting inferences derivable from pcs (used in hover provider of language server).
- @argdown/core now exports jsonReplacer and stringifyArgdownData for stringifying Argdown data.
- @argdown/sandbox removed settings view (as all configuration should be done via frontmatter)
- updated all dependencies
- @argdown/core (and others): replaced dependency on lodash with dependencies on single lodash functions

### Breaking changes

- Process order: Map colorization now happens in the ColorPlugin, so it needs to be run after the MapPlugin
- renamed dotToSvg to vizJs settings
- removed some cli options in favor of configuration through config file or frontmatter section (see cli help for remaining options)
- VizJsMap now uses the external full.render.js that can be used as a web worker. This file should not be processed by a bundler and has to be made accessible to the map view.
- DotToSvgPlugin runs now asynchronous (using app.run instead of app.runAsync will not run it)

### Bug fixes

- #106 vizjs map can not handle untitled arguments
- #104 Extension host stops in VSCODE
- #103 @argdown/sandbox: dagre nodes twice too large in Chrome
- #99 @argdown/codemirror-mode: highlighting of indented inferences
- #97 Language Server should not send parser errors to log
- #90 argdown-vscode: improve quick info on hover
- #87 Incorrect arrows with: isInMap: false and mode:strict bug
- #82 html export: improve anchor link behavior
- #57 redundant edges in map
- #51 pdf export: map is cut off

For older relase notes visit [this page](https://argdown.org/changes/2018.html).
