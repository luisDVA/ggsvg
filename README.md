
<!-- README.md is generated from README.Rmd. Please edit that file -->

# ggsvg - Use SVG as points in ggplot <img src="man/figures/logo-ggsvg.png" align="right" width="230"/>

### With the novel capability of aesthetic mappings to *any* SVG feature

<!-- badges: start -->

![](https://img.shields.io/badge/cool-useless-green.svg)
[![R-CMD-check](https://github.com/coolbutuseless/ggsvg/workflows/R-CMD-check/badge.svg)](https://github.com/coolbutuseless/ggsvg/actions)
<!-- badges: end -->

`ggsvg` is an extension to ggplot to use SVG image for plotted points.

The SVG may be styled by either:

-   Mapping aesthetic values to CSS.
-   Parameterising the SVG by converting it to a [`{glue}`]()-compatible
    string.

<hr />

### Note: `{ggsvg}` is currently undergoing rapid development.

### Keep an eye on this README and the vignettes to see examples of current usage.

<hr />

## What’s in the box

-   `geom_point_svg()` is equivalent to `geom_point()` but the glyphs
    are defined by SVG rather than points.
-   `scale_svg_default()` will assign reasonable default scales to
    arbitrary aesthetics which adhere to the [preferred aesthetic naming
    scheme](#preferred)
-   `scale_svg_*` a complete set of compatible scale functions for
    controlling the mapping of values to arbitrary named aesthetics.
    Having a good knowledge of scales work in ggplot2 will be a big
    advantage in customing SVG appearing in plots.

## Installation

Install from [GitHub](https://github.com/coolbutuseless/ggsvg).

The [`{rsvg}`](https://github.com/ropensci/rsvg) package is used to
convert SVG into an R raster object. This requires at least rsvg(\>=
2.3.0).

``` r
# install.package('remotes')
remotes::install_github('coolbutuseless/ggsvg')
```

## <a name="preferred"></a> Preferred naming for custom aesthetics

Custom aesthetics should be named with a `_[type]` suffix in order to
keep `{ggsvg}` working well with `{ggplot2}`. E.g:

-   Use `shade_fill` rather than `shade` or `fill_shade`
-   Use `rect_size` rather than `size_rect` or `rect_bigness`
-   Use `path_alpha` rather than `transparency`

`type` can be any one of the standard ggplot aesthetics (fill, colour,
size etc).

<details>
<summary>
Click here for more details
</summary>

`{ggplot2}` doesn’t know how to deal with a new aesthetic is hasn’t seen
before. The `ggplot2` packages knows about a certain set of aesthetics:
alpha, colour, fill, linetype, shape, size etc. And it knows how to find
appropriate default *scales* for each of these aesthetics in order to
convert, say, numeric values in a data.frame into colours for points.

In `ggsvg` novel aesthetics are being created during plot creation in
order to colour/style the SVG. Since `ggplot2` won’t know how to find an
appropriate default scale to go with these aesthetics, the user may name
things carefully in order to trigger some automatic detection within the
ggsvg package.

1.  Name aesthetics to be of the form `[blah]_[type]` with an underscore
    -   e.g. “circle_radius”, “rect_colour”, “cat_fill”
2.  Apply a general default scale for these specially named aesthetics
    by adding `scale_svg_default()` to the plot.
3.  To get a *good* plot, you’ll probably still need to add specific
    scales to carefully control the value mapping
    e.g. `scale_svg_fill_brewer(aesthetics = "rect_fill")`

Note that any CSS properties that are specified as text
(e.g. `font-family`) do not currently have any built-in scale. Instead
you will have to specify the possible values for such a scale using
e.g. `scale_svg_discrete_manual()`.

</details>

# Simple plot

``` r
car_url <- 'https://www.svgrepo.com/download/114837/car.svg'
car_svg <- paste(readLines(car_url), collapse = "\n")
```

``` r
grid::grid.draw( svg_to_rasterGrob(car_svg) )
```

<img src="man/figures/README-unnamed-chunk-4-1.png" width="100%" />

``` r
ggplot(mtcars) + 
  geom_point_svg(aes(mpg, wt), svg = car_svg) + 
  theme_bw()
```

<img src="man/figures/README-unnamed-chunk-5-1.png" width="100%" />

# Simple plot with mapped `size` aesthetic

``` r
ggplot(mtcars) + 
  geom_point_svg(aes(mpg, wt, size = mpg), svg = car_svg) + 
  theme_bw() 
```

<img src="man/figures/README-unnamed-chunk-6-1.png" width="100%" />

## Styling SVG with CSS Aesthetics (Example 1)

The 3rd path element in the SVG is the body of the car - which
corresponds to the CSS selector `path:nth-child(3)`.

We use the `css()` helper function define how to map `fill` to a CSS
element.

The general form of the `css()` function call is:
`css(selector, property = value)`

In this case we want to select the element at `path:nth-child(3)`, and
set the CSS property `fill` to be mapped from `as.factor(cyl)`. I.e.

`css("path:nth-child(3)", fill = as.factor(cyl))`

``` r
ggplot(mtcars) + 
  geom_point_svg(
    aes(mpg, wt, css("path:nth-child(3)", fill = as.factor(cyl))),
    svg = car_svg
  ) +
  theme_bw() + 
  scale_svg_default()
```

<img src="man/figures/README-unnamed-chunk-7-1.png" width="100%" />

## Styling SVG with CSS Aesthetics (Example 2)

In this example, the `stroke-width` and the `stroke` of the rectangle
are to be mapped to some values in the data.

Since `ggplot2` has never heard of these custom aesthetics, use
`scale_svg_default()` to get good initial defaults.

``` r
svg_text <- '
  <svg viewBox="0 0 100 100 ">
    <rect width="100" height="100" fill="#555555" 
        style="stroke: black; stroke-width: 20"/>
    <circle class="pale" cx="50" cy="50" r="20" fill="white" />
  </svg>
  '

grid::grid.draw( svg_to_rasterGrob(svg_text) )
```

<img src="man/figures/README-unnamed-chunk-8-1.png" width="100%" />

``` r
ggplot(mtcars) + 
  geom_point_svg(
    aes(
      mpg, wt, 
      css("rect", "stroke-width" = mpg), 
      css("rect", stroke = as.factor(cyl))
    ), 
    svg = svg_text) +
  theme_bw() + 
  scale_svg_default() 
```

<img src="man/figures/README-unnamed-chunk-9-1.png" width="100%" />

Be default, the size scaling for the `stroke-width` aesthetic isn’t
readily apparent, so add a specific `scale_svg_*()` function to map the
values onto thicker strokes

``` r
ggplot(mtcars) + 
  geom_point_svg(
    aes(
      mpg, wt, 
      css("rect", "stroke-width" = mpg), 
      css("rect", stroke = as.factor(cyl))
    ), 
    svg = svg_text) +
  theme_bw() + 
  scale_svg_default() +
  scale_svg_size_continuous(aesthetics = css("rect", "stroke-width" = mpg), range = c(5, 50))
```

<img src="man/figures/README-unnamed-chunk-10-1.png" width="100%" />

## Styling SVG with CSS Aesthetics (Example 3)

This world map SVG has CSS classes corresponding to the countries
e.g. the CSS selector “.Canada” will select all elements having
`class = "Canada"` in the SVG.

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# World map from: https://simplemaps.com/resources/svg-world
# Slightly modified to have a large <rect> background element
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
map_svg <- paste(readLines("man/figures/world-bg.svg"), collapse = "\n")

grid::grid.draw( svg_to_rasterGrob(map_svg) )
```

<img src="man/figures/README-unnamed-chunk-11-1.png" width="100%" />

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Dummy data about Canada and Australia
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
N <- 10
value_df <-  data.frame(
  country = rep(c("Canada", 'Australia'), each = N),
  value   = c(rnorm(N, mean = 5), rnorm(N, mean= 7)),
  fill    = rep(c('brown', 'navy'), each = N)
)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Boxplot comparison with map inset
# Set x_abs,y_abs, hjust, vjust to put the SVG in the top-right corner
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ggplot(value_df) +
  geom_boxplot(aes(x=country, y = value, colour=I(fill)))+
  geom_point_svg(
    mapping = NULL,
    x_abs = 0.99, y_abs = 0.99, hjust = 1, vjust = 1,
    css(".Canada"   , fill = 'brown'),  
    css(".Australia", fill = 'navy'),
    css("rect", fill='white'),       # Style the inset frame
    css("rect", stroke = '#555'),   # Style the inset frame
    css("rect", `stroke-width` = 5), # Style the inset frame
    svg = map_svg,
    size = 75
  ) +
  theme_bw() 
```

<img src="man/figures/README-unnamed-chunk-13-1.png" width="100%" />

## Styling SVG by parameterising as a `{glue}` string

SVG appearance may be styled by editing the SVG and inserting
glue-compatible parameters which can then be mapped as aesthetics.

### (1) Create base SVG Image

The following simple SVG constructed by hand is just a square and a
circle.

``` r
library(ggplot2)
library(ggsvg)

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Define simple SVG
#   - Square with rounded corners and a circle inside it.
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
simple_text <- '
  <svg viewBox="0 0 100 100 ">
    <rect width="100" height="100" fill="#88ccaa" />
    <circle cx="50" cy="50" r="40" fill="white" />
  </svg>
  '

grid::grid.draw( svg_to_rasterGrob(simple_text) )
```

<img src="man/figures/README-simple_svg-1.png" width="100%" />

### (2) Parameterise the SVG

Introduce parameters in the SVG using
[`{glue}`](https://cran.r-project.org/package=glue) syntax with double
curly braces i.e. `{{}}`

``` r
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Define simple SVG
#   - Square with rounded corners and a circle inside it.
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
parameterised_svg <- '
  <svg viewBox="0 0 100 100 ">
    <rect width="100" height="100" fill="{{rect_fill}}" />
    <circle cx="50" cy="50" r="{{circle_size}}" fill="white" />
  </svg>
  '

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
# Test the glue parameters
#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
rect_fill   <- 'brown'
circle_size <- 10

final_text <- glue::glue(parameterised_svg, .open = "{{", .close = "}}")

grid::grid.draw( svg_to_rasterGrob(final_text) )
```

<img src="man/figures/README-unnamed-chunk-14-1.png" width="100%" />

## (3) Static aesthetics with parameterised SVG

``` r
ggplot(mtcars) +
  geom_point_svg(
    mapping     = aes(mpg, wt),
    svg         = parameterised_svg,
    rect_fill   = 'navy',
    circle_size = 20
  ) +
  theme_bw() + 
  labs(title = "{ggsvg}") 
```

<img src="man/figures/README-unnamed-chunk-15-1.png" width="100%" />

## (3) Map ggplot2 aesthetics to the parameterised SVG

``` r
ggplot(mtcars) +
  geom_point_svg(
    mapping  = aes(mpg, wt, circle_size = as.factor(cyl), rect_fill = disp),
    svg      = parameterised_svg
  ) +
  theme_bw() + 
  labs(title = "{ggsvg}") + 
  scale_svg_default() +
  scale_svg_size_discrete(
    aesthetics = 'circle_size',
    range = c(10, 45),
    guide = guide_legend(override.aes = list(size = 7))
  )
#> Warning: Using size for a discrete variable is not advised.
```

<img src="man/figures/README-unnamed-chunk-16-1.png" width="100%" />

## Acknowledgements

-   R Core for developing and maintaining the language.
-   CRAN maintainers, for patiently shepherding packages onto CRAN and
    maintaining the repository
