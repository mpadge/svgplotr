<!-- README.md is generated from README.Rmd. Please edit that file -->
[![Build Status](https://travis-ci.org/mpadge/svgplotr.svg)](https://travis-ci.org/mpadge/svgplotr) [![codecov](https://codecov.io/gh/mpadge/svgplotr/branch/master/graph/badge.svg)](https://codecov.io/gh/mpadge/svgplotr) [![Project Status: Concept - Minimal or no implementation has been done yet.](http://www.repostatus.org/badges/0.1.0/concept.svg)](http://www.repostatus.org/#concept)

svgplotr
========

Fast `svg` plots in **R**. Currently just a demonstration of speed relative to [`svglite`](https://github.com/r-lib/svglite). The function `getdat()` generates a series of random edges with varying colours and line widths to be plotted. Comparison is against a `ggplot2` object with no embellishments, set up with the following code

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
```

One set of random lines can then be generated and plotted like this:

``` r
dat <- getdat (n = 1e4, xylim = 1000)
fig <- ggplot () + ggmin_theme () +
    geom_segment (aes (x = xfr, y = yfr, xend = xto, yend = yto,
                       colour = col, size = lwd), size = dat$lwd, data = dat)
print (fig)
```

![](README-fig-1.png)

Timing Comparison
-----------------

Differences in timing depend on the scale of figures

``` r
require (svglite)
plotgg <- function (fig)
{
    svglite ("lines.svg")
    print (fig)
    graphics.off ()
}

do1test <- function (n = 1e3, nreps = 5)
{
    dat <- getdat (n = n)
    fig <- ggplot () + ggmin_theme () +
        geom_segment (aes (x = xfr, y = yfr, xend = xto, yend = yto,
                           colour = col, size = lwd),
                      size = dat$lwd, data = dat)
    rbenchmark::benchmark (
                           plotgg (fig),
                           svgplot (dat, filename = "lines"),
                           order = "test",
                           replications = nreps)$relative [1]
}

n <- 10 ^ (20:50 / 10)
y <- sapply (n, do1test)
```

Then plot the results

``` r
plot (n, y, log = "xy", main = "relative performance of svgplotr vs svglite")
lines (n, y, col = "gray")
```

![](README-plot-timings-1.png)

And efficiency gains will at some stage be lost, but for up to around 10,000 edges, `svgplotr` generally remains an order of magnitude faster than `svglite`. This figure is nevertheless logarithically scaled, indicating that efficiency gains decrease exponentially with increasing size, so maybe it's not worth it after all?
