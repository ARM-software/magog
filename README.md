# Mobile GPU Profiling and Optimization eBook

This project is an eBook that teaches developers how to profile and optimize their mobile applications using an Arm GPU, helping them to get the best rendering performance out of an Arm GPU.

The latest version of the book is published on the [project GitHub Pages site ↗](https://arm-software.github.io/magog/).

## Setup

The eBook is implemented as a documentation website, built using the static site generator, Hugo. To use Hugo you will need to install Go and Hugo, and add
make both available on your `PATH`.

* [Go ↗](https://go.dev/)
* [Hugo ↗](https://gohugo.io/)

We are using the Hextra theme for Hugo, which provides a feature-rich page layout designed for documentation. Hextra user documentation can be found on the theme website:

* [Hugo: Hextra theme ↗](https://imfing.github.io/hextra/)

## Building for local development

```sh
# Install modules
hugo mod tidy

# Start the server for local development
hugo server --disableFastRender -p 1313
```

## Building for publication

```sh
# Install modules
hugo mod tidy

# Build the site for remote deployment
hugo build -b <remote site root URL>
```

## Authoring guidelines

### Writing style

Prose uses US English.

Our readers are an international audience, so we avoid very long sentences and use of idiomatic phrasing, which both make it harder to translate.

### Technical style

Paragraphs should be written on a single line, without manual line wrapping. This makes it easier to review changes to existing content, because there is no patch noise due to reflowing content that is otherwise unchanged.

### Images

Images are 680 pixels wide when viewed in the default reading column width.

Whenever possible we use vector `.svg` images, to support variable resolution presentation. If raster images are needed, we use the `.webp` file format to reduce bandwidth.

A color palette for Inkscape is provided in `./assets/inkscape`. Copy the GPL
file into your Inkscape user palettes directory, which you can find using "Edit > Preferences > System" in Inkscape.

### Markdown snippets

Special characters:

* Registered trademark: ®
* Trademark: ™
* Endash: –
* Emdash: —

Hugo Markdown snippets:

* Glossary references: `{{< term "ASTC" >}}`
* Relative links: `[Foo]({{< relref "01_mobile_devices.md" >}})`
* Multi-line alert with a title:
  ```
  > [!NOTE] Title text
  > Body text
  ```
* Single line callout without a title:
  ```
  {{< callout type="info" >}}
  Body text
  {{< /callout >}}
  ```

- - -

_Copyright © 2026, Arm Limited and contributors._
