<!-- README.md is generated from README.Rmd. Please edit that file -->
[![Build Status](https://travis-ci.org/mpadge/svgplotr.svg)](https://travis-ci.org/mpadge/svgplotr) [![codecov](https://codecov.io/gh/mpadge/svgplotr/branch/master/graph/badge.svg)](https://codecov.io/gh/mpadge/svgplotr) [![Project Status: Concept - Minimal or no implementation has been done yet.](http://www.repostatus.org/badges/0.1.0/concept.svg)](http://www.repostatus.org/#concept)

svgplotr
========

Fast `svg` plots in **R**. Currently just a demonstration of speed relative to [`svglite`](https://github.com/r-lib/svglite) based on four functions:

1.  `getlines(n)` which generates a series of `n` random edges tracing a single path with varying colours and line widths;
2.  `getpoints(n)` which generates `n` random points with varying colours;
3.  `svgplot_lines()` to write line data to a `html`-formatted `svg` file.
4.  `svgplot_points()` to write point data to a `html`-formatted `svg` file.

Comparison is against a `ggplot2` object with no embellishments, set up with the following code

``` r
require (ggplot2)
ggmin_theme <- function ()
{
    theme <- theme_minimal ()
    theme$panel.background <- element_rect (fill = "transparent",
                                                     size = 0)
    theme$line <- element_blank ()
    theme$axis.text <- element_blank ()
    theme$axis.title <- element_blank ()
    theme$plot.margin <- margin (rep (unit (0, 'null'), 4))
    theme$legend.position <- 'none'
    theme$axis.ticks.length <- unit (0, 'null')
    return (theme)
}
ggline <- function (dat)
{
    ggplot () + ggmin_theme () +
        geom_segment (aes (x = xfr, y = yfr, xend = xto, yend = yto, size = lwd),
                      col = dat$col, size = dat$lwd / 8, data = dat)
}
ggpoint <- function (dat)
{
    ggplot () + ggmin_theme () +
        geom_point (aes (x = x, y = y), col = dat$col, data = dat)
}
```

One set of random lines can then be generated and plotted via `ggplot2` like this:

``` r
dat <- getlines (n = 1e5, xylim = 1000)
ggline (dat)
```

![](README-fig-1.png)

The equivalent output of `svgplotr` can be directly viewed as a `.html`, or a `.svg` file can be converted to any other format using the `rsvg` package:

``` r
svgplot_lines (dat, file = "junk", html = FALSE) # makes junk.svg
require (rsvg)
png::writePNG (rsvg ("junk.svg"), "junk.png")
```

Timing Comparison
-----------------

`svgplotr` is considerably faster than `svglite`, but speed differences depend on numbers of edges plotted. The following code quantifies the time taken to plot both lines and points by `svglite` in comparison to `svgplotr` as a function of `n`.

``` r
require (svglite)
require (rbenchmark)
plotgg <- function (fig)
{
    svglite ("lines.svg")
    print (fig)
    graphics.off ()
}

testlines <- function (n = 1e3, nreps = 5)
{
    dat <- getlines (n = n)
    fig <- ggline (dat)
    benchmark (
               plotgg (fig),
               svgplot_lines (dat, filename = "lines"),
               order = "test",
               replications = nreps)$relative [1]
}

testpoints <- function (n = 1e3, nreps = 5)
{
    dat <- getpoints (n = n)
    fig <- ggpoint (dat)
    benchmark (
               plotgg (fig),
               svgplot_points (dat, filename = "lines"),
               order = "test",
               replications = nreps)$relative [1]
}

n <- 10 ^ (20:60 / 10)
ylines <- sapply (n, testlines)
ypoints <- sapply (n, testpoints)
dat <- data.frame (n = n, lines = ylines, points = ypoints)
```

Then plot the results

``` r
dat <- tidyr::gather (dat, key = "n")
names (dat) <- c ("n", "type", "y")
ggplot (dat, aes (x = n, y = y, group = type)) +
    theme (panel.grid.minor = element_blank ()) +
    scale_x_log10 (breaks = 10 ^ (2:6)) +
    scale_y_log10 (limits = c (1, max (dat$y)), breaks = c (1:5, 10, 50, 100)) +
    scale_colour_manual (values = c ("red", "blue")) +
    geom_point (aes (colour = type)) +
    geom_smooth (aes (colour = type), method = "loess", se = TRUE) +
    ylab ("time (svgplotr) / time (svglite)") +
    labs (title = "relative performance of svgplotr vs svglite")
```

![](README-plot-timings-1.png)

And efficiency gains initially decrease exponentially, but then flatten out and appear to approach asymptotic limits. Even for the maximum size in this plot of 1 million objects, `svgplotr` is almost 4 times faster than `svglite` for lines, and over 8 times faster for points. The right portion of the graph may also be a second exponential regime, but even if so, parity for lines is only going to be reached at:

``` r
indx <- which (dat$n >= 1e5 & dat$type == "lines")
mod <- as.numeric (lm (log10 (dat$y [indx]) ~ log10 (dat$n [indx]))$coefficients)
format (10 ^ (mod [1] / abs (mod [2])), scientific = TRUE, digits = 2)
#> [1] "1.3e+11"
```

which is 130 billion edges, and obviously enormously more for points. Parity is not really going to happen, and `svgplotr` will always remain faster than `svglite`.
