
<!-- README.md is generated from README.Rmd. Please edit that file -->

# chisel

Functions to facilitate work across The Carpentries repositories.

Please note, this is not intended for general use. It contains functions
that are designed to work with the idiosyncracies of The Carpentries
Lesson Infrastructure. If you have questions, please contact
\[@zkamvar\](<https://github.com/zkamvar>)

## Working with this repository

### setup

This repository is an R package, but it is designed to be used while in
an R package development environment, so in order to use this
repository, you will need the following tools:

-   Git
-   [An R interpreter](https://r-project.org)
    ([RStudio](https://posit.co/download/rstudio-desktop/) is a good
    starting place)
-   [The {devtools} package](https://r-lib.github.io/devtools)

[R Packages by Wickham and Bryan](https://r-pkgs.org) is a *really* good
resource for the ins and outs of R package development.

To install devtools, open R and run:

``` r
install.packages("devtools")
```

Once you have that, you can clone this repository to your computer with

``` bash
git clone https://github.com/carpentries/chisel.git
```

### loading the package

From insid the chisel repository, open R and load the package with the
keyboard shortcut CTRL + SHIFT + L (on Mac, it’s CMD + SHIFT + L), or
you can type this:

``` r
devtools::load_all()
```

This will load all functions into the workspace. If you need to edit a
function, you can do so and make sure to re-load the package.

## Lesson Release

To create a lesson release for a lesson that uses Jekyll, you must first
insert the global mailmap file shared in the Core Curriculum Team into
`inst/mailmap/mailmap.txt`

After you do that, you will need to open [`R/release.R`](R/release.R),
which will contain the majority of the code for lesson release.

Examples of past lesson releases can be found if you search for the
`# INTERACTIVE PART` comments. In between those two comments, you will
find something that looks like this:

``` r
if (FALSE) {

# ... a LOT of lines of R code 

}
```

François used this trick to avoid these examples from being included in
the package code. These examples will often look like this:

    res <- tibble::tribble(
        ~name,  ~owner, ~repo,
        "main", "datacarpentry", "image-processing",
        "template", "carpentries", "styles"
      ) %>%
        generate_zenodo_json(local_path = "~/Documents/DataCarpentry/image-processing",
          editors = c("tobyhodges", "uschille", "bobturneruk", "quist00", "K-Meech"),
          ignore = c("francois.michonneau@gmail.com", "zkamvar@carpentries.org"))

The main function is `generate_zenodo_json()`, which takes a table where
each row represents a repository.

These tables are constructed using the `tibble::tribble()` function,
which allows us to create tables as we would read them.

This table is then piped `%>%` into the function.

### The lesson table

The FIRST row is always the repository that we want to create the lesson
release for. The subsequent rows are the template repositories used to
generate the release. The columns are in this format:

| column | purpose                                                                        |
|--------|--------------------------------------------------------------------------------|
| name   | name of the branch (for the first row), name of template (for subsequent rows) |
| owner  | github owner name                                                              |
| repo   | github repo name                                                               |

Thus, the above example is creating a lesson release for the
[datacarpentry/image-processing](https://github.com/datacarpentry/image-processing)
lesson, using
[carpentries/styles](https://github.com/carpentries/styles) as the base:

| name     | owner         | repo             |
|----------|---------------|------------------|
| main     | datacarpentry | image-processing |
| template | carpentries   | styles           |

### Translated lessons

The Spanish lessons have a *different* history because they translated
styles first, so their config looks like this:

| name     | owner       | repo                  |
|----------|-------------|-----------------------|
| main     | swcarpentry | r-novice-gapminder-es |
| source   | swcarpentry | r-novice-gapminder    |
| template | swcarpentry | styles-es             |
