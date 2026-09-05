# slidev-theme-vzhyx

[![NPM version](https://img.shields.io/npm/v/slidev-theme-vzhyx?color=3AB9D4&label=)](https://www.npmjs.com/package/slidev-theme-vzhyx)

A (...) theme for [Slidev](https://github.com/slidevjs/slidev).

<!--
  Learn more about how to write a theme:
  https://sli.dev/guide/write-theme.html
--->

<!--
  run `npm run dev` to check out the slides for more details of how to start writing a theme
-->

<!--
  Put some screenshots here to demonstrate your theme

  Live demo: [...]
-->

## Install

Add the following frontmatter to your `slides.md`. Start Slidev then it will prompt you to install the theme automatically.

<pre><code>---
theme: <b>vzhyx</b>
---</code></pre>

Learn more about [how to use a theme](https://sli.dev/guide/theme-addon#use-theme).

## Layouts

| Layout | Description |
|--------|-------------|
| `cover` | Title slide with logo, meetup, and date meta |
| `intro` | Speaker introduction with photo slot |
| `simple-slide` | Content slide with heading and body area |

## Components

| Component | Props | Description |
|-----------|-------|-------------|
| `LineChart` | `series`, `annotations?`, `xLabel?`, `yLabel?`, `width?`, `height?`, `xMin/xMax/yMin/yMax?` | Multi-series SVG line chart. Supports dashed lines, filled areas under curves, and vertical annotation lines with staggered labels. |
| `GroupedBarChart` | `criteria`, `frameworks`, `width?`, `height?`, `maxValue?` | Grouped SVG bar chart. Renders one bar group per criterion, one bar per framework, with rotated x-axis labels and a legend row. |

### LineChart series format
```ts
{ id: string; label: string; color: string; dash?: boolean; fill?: boolean; data: { x: number; value: number }[] }
```

### GroupedBarChart criteria/frameworks format
```ts
// criteria
{ id: string; label: string; values: Record<frameworkId, number> }
// frameworks
{ id: string; label: string; color: string }
```

## Contributing

- `npm install`
- `npm run dev` to start theme preview of `example.md`
- Edit the `example.md` and style to see the changes
- `npm run export` to generate the preview PDF
- `npm run screenshot` to generate the preview PNG
