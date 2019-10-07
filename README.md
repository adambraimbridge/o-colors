# o-colors [![Build Status](https://circleci.com/gh/Financial-Times/o-colors.png?style=shield&circle-token=0d3884dacba2fdd8aec8afbd2eaa84c79040ddd5)](https://circleci.com/gh/Financial-Times/o-colors) [![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](#licence)

The color palette for the FT masterbrand and sub-brand products.

- [Usage](#usage)
	- [Sass](#sass)
	- [Markup](#markup)
	- [CSS Variables](#css-variables)
- [Migration guide](#migration-guide)
	- [Upgrading from v3.x.x to v4.x.x](#upgrading-from-v3xx-to-v4xx)
- [Contact](#contact)
- [Licence](#licence)


## Usage

There are a number of ways of using colors in your component or product. o-colors can be used via the [Build Service](https://www.ft.com/__origami/service/build/v2/), but it is recommended you import the Sass into your project to make use of the many Sass mixins and functions available.


### Sass:

We recommend Sass projects use the [mixins and functions](#mixins-and-functions) `o-colors` provides directly in their own Sass. However it is also possible to output all `o-colors` CSS classes including [CSS custom properties](#css-variables) using the `oColors` mixin.

```scss
@import 'o-colors/main';
@include oColors();
```

The `oColors` mixin accepts an `$opts` argument to granularly include `o-colors` CSS:

```scss
@include oColors($opts: (
	'palette-custom-properties': true // output just css custom properties "css variables"
));
```

| Feature                   | Description                                                                                                                         |
|---------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| palette-custom-properties | Output a [CSS custom property (CSS Variable)](#css-variables) for each colour in the palette.                                       |
| palette-classes           | Output CSS classes which apply a background for each palette colour, see the [markup section](#markup) for a list of these classes. |

#### Colors and accessibility

o-colors has been built to help bridge the gap between design and development by providing functionality to help create colors dynamically from a central palette as well as generate contrasted text colors based on an elements background color.

You can create tints of a color with the [`oColorsGetTint`](#tint-palette-colors) function. The function takes a palette color name and a brightness value (based on HSB color) to return a tint of the palette color.

To work with text colors the [`oColorsFor`](#use-case-mixin) mixin and [`oColorsGetTextColor`](#generated-text-colors) function will output a text color based on the background color, which will be a mix of either black or white with the background at the percentage requested. You can also mix two colors manually using the [`oColorsMix`](#mix-colors) function, providing two colors (either hex or palette color names) and a percentage at which to mix them.

When working with the `oColorsFor` and `oColorsGetTextColor` features, the Sass will also automatically test the background color with the generated text color to see if the combination passes Web Content Accessibility Guidelines (WCAG). If the combination fails to pass at least WCAG AA you will see an error, if the combination passes AA but only at a larger font size (18px+), there will be a warning.

For manually testing color contrasts, you can use [Lea Verou's Contrast Ratio tool](http://leaverou.github.io/contrast-ratio/).

#### Mixins and functions

o-colors has a number of mixins and functions for you to access the color palette in your project, here they are listed from the most to least preferred methods of working with o-colors.

##### Use case mixin

Use the `oColorsFor` mixin to add color-related properties to your ruleset:

```scss
.my-thing {
	@include oColorsFor(custom-box box, background text, 80);
}
```

Will output:

```css
.my-thing {
	background-color: #f2dfce;
	color: #302d29; // black mixed with #f2dfce at 80%
}
```

It takes three arguments:

* **Use case list**: a list of color use cases in order of preference.  The first one that is defined will be returned.
* **Property list**: a list of all the properties you want the color applied to (background, border, text). They each correspond to `background-color`, `border-color` and `color`. Default is _all_ which includes all three properties.
* **Text Level**: the opacity (1-100) for the text color derived from the background color of the use case. _If a text use case exists already, this will have no effect_.

In the example above, the background and text colors are set, preferably from the *custom-box* use case, and if either of those properties are not available for that use case, from the *box* use case. If the text use case is not set, the function will output a text color based on either black or white (depending on the brightness of the background) blended with the background color at the text level specified.

### Use case function

If you need to use a color value as part of a more complex CSS rule, e.g. a border color for just one side, or a gradient background, use the `oColorsGetColorFor` function:

```scss
.my-thing {
	color: oColorsGetColorFor(article-life-arts-body article-body body, text, (default: blue));
}
```

The `oColorsGetColorFor` function takes three arguments:

* **Use case list**: a list of color use cases in order of preference.  The first one that is defined for the specified property will be returned
* **Property**: The property that you want to use the color for (background, border, or text).  Note that in contrast to the `oColorsFor` mixin, you must specify only one property.   Options are `background`, `border`, `text`, and `all`.
* **Options**: A Sass *map* of additional options, all of which are optional, and may comprise:
	* **default**: The name of a palette color to return if none of the specified use cases are defined for the desired property.  May also be set to `null` or `undefined` to return that instead of the built in default (which is transparent)

_This function will not generate a text color based on the use case like `oColorsFor` to get a text color based on a use case, use `oColorsGetTextColor`_

### Palette color function

If you have a color use case not covered by those built into the colors module, consider defining a custom use case (see below) and then using the use case mixin or function described above.  However, if you need to use a particular color in one single place only, it may be worth using the `oColorsByName` function, which returns the CSS color for a palette color name:

```scss
.my-thing {
	color: oColorsByName('white-60');
}
```

#### Generated text colors

`oColorsGetTextColor` will return a text color based on the background and an opacity specified. The base of the text color is either black or white depending on the brightness of the background color and then mixed with the background at the specified opacity using `oColorsMix`.

_Warning_: if the combination of background and text color created by the function would not pass WCAG AA level, o-colors will throw an error.

Usage:

```scss
.o-colors-palette-teal {
	color: oColorsGetTextColor(oColorsByName('teal'), 80);
}
```

Output:

```css
.o-colors-palette-teal {
	color: #cce3e5;
}
```

#### Tint palette colors

`oColorsGetTint` will return a tinted palette color based on a specified brightness.
The function takes the name of a palette color and an HSB brightness value between 0-100. For every colour except black, increasing the HSB brightness makes it appear lighter (ie claret becomes much pinker). Increasing the HSB brightness value of black will make it blacker. Take a look at the [registry demo](http://registry.origami.ft.com/components/o-colors) for a visual demo of this.

Usage:

```scss
.o-colors-tinted-color {
	background-color: oColorsGetTint('jade', 90);
}
```

Output:

```css
.o-colors-tinted-color {
	background-color: #177ee6;
}
```

#### Mix colors

`oColorsMix` will mix two colors based on a percentage. This gives the impression of the base color appearing at the percentage opacity over the background color. `oColorsMix` will accept either a color value or the name of an o-colors palette color as arguments.

Usage:

```scss
.o-colors-palette-white {
	border: 1px solid oColorsMix(black, white, 20);
}
```

Output:

```css
.o-colors-palette-white {
	border: 1px solid #cccccc;
}
```

## Usecases

`USECASE` | `PROPERTY` |
---|---
focus |                 outline
page |                  background
box |                   background
link |                  text
link-hover |            text
link-title |            text
link-title-hover |      text
tag-link |              text
tag-link-hover |        text
opinion-tag-link |      text
opinion-tag-link-hover |text
title |                 text
body |                  text
muted |                 text
opinion |               background
hero |                  background
hero-opinion |          background
hero-highlight |        background
**Section colors** |
section-life-arts |     all
section-life-arts-alt | all
section-magazine |      all
section-magazine-alt |  all
section-house-home |    all
section-house-home-alt |all
section-money |         all
section-money-alt |     all

## Defining custom usecases

You can add usecases for your particular component or product. This is done using the `oColorsSetUseCase` mixin:

```scss
@include oColorsSetUseCase(email, text, 'black-60');
```

It takes three arguments:

* **Usecase**: your particular usecase
* **Property**: the property for which the color should be used for (background, border, or text)
* **Color**: a color from the palette

If you are creating a usecase for a component, you *must* namespace your usecase name with the name of your component.

You can also use `oColorsByUsecase` to retrieve the palette color name (eg `paper`) defined against a usecase. This can be useful when you need the palette color name to use with another Sass mixin.


### Markup

When using the [Build Service](https://origami.ft.com/docs/services/#build-service) or the [`oColors` mixin](#sass), `o-colors` provides you with helper classes to access the color palette. All palette colors are available as `.o-colors-palette-[NAME]` (which style just `background-color`).

### CSS Variables

When using the [Build Service](https://origami.ft.com/docs/services/#build-service) or the [`oColors` mixin](#sass), `o-colors` will output all the palette colors as [CSS custom properties (CSS Variables)](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_variables). These will use the format `--o-colors-{name}` (e.g. `--o-colors-black` and `--o-colors-teal`).

## Migration

State | Major Version | Last Minor Release | Migration guide |
:---: | :---: | :---: | :---:
✨ active | 5 | N/A | [migrate to v5](MIGRATION.md#migrating-from-v4-to-v5) |
⚠ maintained | 4 | 4.10 | [migrate to v4](MIGRATION.md#migrating-from-v3-to-v4) |
╳ deprecated | 3 | 3.6 | [migrate to v3](MIGRATION.md#migrating-from-v2-to-v3) |
╳ deprecated | 2 | 2.5 | [migrate to v2](MIGRATION.md#migrating-from-v1-to-v2) |
╳ deprecated | 1 | 1.1 | [migrate to v1](MIGRATION.md#migrating-from-v0-to-v1) |
╳ deprecated | 0 | 0.2 | N/A |


## Contact

If you have any questions or comments about this component, or need help using it, please either [raise an issue](https://github.com/Financial-Times/o-colors/issues), visit [#ft-origami](https://financialtimes.slack.com/messages/ft-origami/) or email [Origami Support](mailto:origami-support@ft.com).


----

## Licence

This software is published by the Financial Times under the [MIT licence](http://opensource.org/licenses/MIT).
