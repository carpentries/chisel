
<!-- README.md is generated from README.Rmd. Please edit that file -->

# chisel

Functions to facilitate work across The Carpentries repositories.

Please note, this is not intended for general use. It contains functions
that are designed to work with the idiosyncracies of The Carpentries
Lesson Infrastructure. If you have questions, please contact
[@zkamvar](https://github.com/zkamvar)

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
          since = "2021-06-09",
          editors = c("tobyhodges", "uschille", "bobturneruk", "quist00", "K-Meech"),
          ignore = c("francois.michonneau@gmail.com", "zkamvar@carpentries.org"))

The main function is `generate_zenodo_json()`, which takes a table where
each row represents a repository.

These tables are constructed using the `tibble::tribble()` function,
which allows us to create tables as we would read them.

This table is then piped `%>%` into the function.

### Output

When you run the function, likely, there will be some github noreply
emails that are not in our AMY database or in our mailmap file. The
release process will attempt to match the github handles from these
names to AMY data and update the master mailmap file in
`inst/mailmap/mailmap.txt`.

When there are github handles detected, you will see a message like
this:

    Matched 9/12 github handles to AMY data
    These handles had missing information:

      brynnelliott
      raisaionin
      TM612

    # 2023-01-27 10:25:28 ---
    writing 9 new entries to mailmap starting on line 492

If there are zero matches, the mailmap file will not be updated.

#### Output Object

The output object (usually called `res`) contains a data frame and a
json representation.

The output is a `.zenodo.json` file written to the `local_path` (your
copy of the lesson repository), but it also returns a list with two
elements:

-   `$data`: contains the data frame of all contributors. The first
    three columns are going to be:
    -   name: the name they will be published under (determined
        `lesson_publication_consent` and if we can match their email to
        AMY
    -   email: the email from either AMY or GitHub
    -   n: the number of commits associated with this release There is
        also associated AMY data and if you find NAs in
        `person_name_with_middle`, that’s a good indicator that you have
        an email that is missing. In this case, their email will be the
        one used for GitHub (see tips and tricks below for a command
        line way to query the commit)
-   `$json` this is a list that prints as the zenodo JSON file. If this
    looks okay to you, then it’s all good.

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

### Useful trips and ticks

When I run this, It’s handy to keep a copy of `all_people()` handy
because that function makes a call to the API *every* single time.

You can use this to search for github user names, which is handy in this
context becuase if a github user name is missing from the resulting
data, then it’s clear that we could not find their email.

``` r
ap <- all_people()

# some time later ...

ap |> filter(grepl("zkamvar", github)) |> glimpse()
#> Rows: 1
#> Columns: 14
#> $ person_name_with_middle    <chr> "Zhian Namir Kamvar"
#> $ pub_name                   <chr> "Zhian Kamvar"
#> $ email                      <chr> "zkamvar@carpentries.org"
#> $ github                     <chr> "zkamvar"
#> $ twitter                    <chr> "zkamvar"
#> $ orcid                      <chr> "0000-0003-1458-7108"
#> $ url                        <chr> "https://zkamvar.netlify.app"
#> $ country                    <chr> "US"
#> $ iata                       <chr> "PDX"
#> $ latitude                   <dbl> 45.58861
#> $ longitude                  <dbl> -122.5975
#> $ badges                     <dbl> 6713
#> $ publish_profile            <dbl> 1
#> $ lesson_publication_consent <chr> "amy"
```

From here, I would add my name to `inst/mailmap/mailmap.txt` to make
sure it’s recognised.

To see what *kind* of commit a person made, you can use
`git log --author=<name matching>`
