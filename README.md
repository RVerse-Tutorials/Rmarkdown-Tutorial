rep-res-coho-example
================
03 April, 2017

-   [General Goals](#general-goals)
-   [Data](#data)
    -   [Reading Data In](#reading-data-in)

<!-- README.md is generated from README.Rmd. Please edit that file -->
General Goals
-------------

This is a simple RStudio project that Eric Anderson has put together as an example/template for students to get a sense of what he expects them to do in terms of assembling their data for the 2017 EEB 295 course, "[Case Studies in Reproducible Research](https://eriqande.github.io/rep-res-eeb-2017/)".

The general goals of this project (for our purposes at the moment) are fairly straightforward. We want to use pedigree information to summarize the ages and distributions of family sizes of coho returning to Klamath River hatcheries. etc. etc.

Data
----

The data for this project are all housed in the `./data` directory. There are two main types of files:

1.  There are three files which are output files from the program [SNPPIT](https://github.com/eriqande/snppit). These are `snppit_output_ParentageAssignments_2013Juvs.txt`, `snppit_output_ParentageAssignments_2014Juvs.txt`, and `snppit_output_ParentageAssignments_2015Juvs.txt`. These are TAB-delimited text files which give the inferred trios (Father-Mother-Offspring) of hatchery coho salmon in our Shasta River Project. These files use "---" to denote columns that have missing data. These result from analyses made on the genetic data. In a real reproducible example, we would have started from the genotype data and actually run the SNPPIT analyses reproducibly, as well. But, for an example, it will be simpler to start from these simple, intermediate files.

2.  There is one data file of extra metadata that should include all the individuals in the snppit output files (and probably a few extra ones as well.) The main key between this file and the other ones is the NFMS\_DNA\_ID which is part of the ID in the Kid, Ma, and Pa columns in the `snppit_output*` files.

### Reading Data In

#### SNPPIT files

The SNPPIT files can be read in with `read_tsv()` making note of the missing data "---".

``` r
library(tidyverse)
#> Loading tidyverse: ggplot2
#> Loading tidyverse: tibble
#> Loading tidyverse: tidyr
#> Loading tidyverse: readr
#> Loading tidyverse: purrr
#> Loading tidyverse: dplyr
#> Conflicts with tidy packages ----------------------------------------------
#> filter(): dplyr, stats
#> lag():    dplyr, stats

snppit2013 <- read_tsv(file = "data/snppit_output_ParentageAssignments_2013Juvs.txt", 
                       na = "---")
#> Parsed with column specification:
#> cols(
#>   .default = col_integer(),
#>   OffspCollection = col_character(),
#>   Kid = col_character(),
#>   Pa = col_character(),
#>   Ma = col_character(),
#>   PopName = col_character(),
#>   FDR = col_double(),
#>   Pvalue = col_double(),
#>   LOD = col_double(),
#>   P.Pr.C_Se_Se = col_double(),
#>   P.Pr.Max = col_double(),
#>   MaxP.Pr.Relat = col_character(),
#>   MendIncLoci = col_character()
#> )
#> See spec(...) for full column specifications.
```

Of course, if we wanted to read them all in at once and make a tidy frame of all of them we would do:

``` r
# read all three into a list
years <- 2013:2015
names(years) <- years
snppit <- lapply(years, function(y) {
  read_tsv(file = paste("data/snppit_output_ParentageAssignments_", y, "Juvs.txt", sep = ""),
           na = "---")
})
#> Parsed with column specification:
#> cols(
#>   .default = col_integer(),
#>   OffspCollection = col_character(),
#>   Kid = col_character(),
#>   Pa = col_character(),
#>   Ma = col_character(),
#>   PopName = col_character(),
#>   FDR = col_double(),
#>   Pvalue = col_double(),
#>   LOD = col_double(),
#>   P.Pr.C_Se_Se = col_double(),
#>   P.Pr.Max = col_double(),
#>   MaxP.Pr.Relat = col_character(),
#>   MendIncLoci = col_character()
#> )
#> See spec(...) for full column specifications.
#> Parsed with column specification:
#> cols(
#>   .default = col_integer(),
#>   OffspCollection = col_character(),
#>   Kid = col_character(),
#>   Pa = col_character(),
#>   Ma = col_character(),
#>   PopName = col_character(),
#>   FDR = col_double(),
#>   Pvalue = col_double(),
#>   LOD = col_double(),
#>   P.Pr.C_Se_Se = col_double(),
#>   P.Pr.Max = col_double(),
#>   MaxP.Pr.Relat = col_character(),
#>   MendIncLoci = col_character()
#> )
#> See spec(...) for full column specifications.
#> Parsed with column specification:
#> cols(
#>   .default = col_integer(),
#>   OffspCollection = col_character(),
#>   Kid = col_character(),
#>   Pa = col_character(),
#>   Ma = col_character(),
#>   PopName = col_character(),
#>   FDR = col_double(),
#>   Pvalue = col_double(),
#>   LOD = col_double(),
#>   P.Pr.C_Se_Se = col_double(),
#>   P.Pr.Max = col_double(),
#>   MaxP.Pr.Relat = col_character(),
#>   MendIncLoci = col_character()
#> )
#> See spec(...) for full column specifications.

# note that we can check the column specifications for any one of them
spec(snppit$`2014`)
#> cols(
#>   OffspCollection = col_character(),
#>   Kid = col_character(),
#>   Pa = col_character(),
#>   Ma = col_character(),
#>   PopName = col_character(),
#>   SpawnYear = col_integer(),
#>   FDR = col_double(),
#>   Pvalue = col_double(),
#>   LOD = col_double(),
#>   P.Pr.C_Se_Se = col_double(),
#>   P.Pr.Max = col_double(),
#>   MaxP.Pr.Relat = col_character(),
#>   TotPaNonExc = col_integer(),
#>   TotMaNonExc = col_integer(),
#>   TotUnkNonExc = col_integer(),
#>   TotPairsNonExc = col_integer(),
#>   KidMiss = col_integer(),
#>   PaMiss = col_integer(),
#>   MaMiss = col_integer(),
#>   MI.Kid.Pa = col_integer(),
#>   MI.Kid.Ma = col_integer(),
#>   MI.Trio = col_integer(),
#>   MendIncLoci = col_character()
#> )

# and we can put them together into a single data frame with
# a juvie_year column like this
snppit_all <- bind_rows(snppit, .id = "juvie_year") %>%
  mutate(juvie_year = as.integer(juvie_year))
```

#### The Meta Data

This file is actually an interesting example because it has column names with single quotes (who did that?) and also with "\#" symbols. If we read this with base R's `read.csv()` it will mangle those names. The `readr` functions never do that. There are also clearly some problems, which we will get to later

``` r
meta <- read_csv("data/shasta-meta-and-genos.csv")
#> Parsed with column specification:
#> cols(
#>   .default = col_character(),
#>   `PIT/Tag#` = col_integer(),
#>   LENGTH = col_integer(),
#>   BirthYear = col_integer(),
#>   `Collection Year` = col_integer()
#> )
#> See spec(...) for full column specifications.
#> Warning: 107 parsing failures.
#>  row      col               expected                                                                actual
#> 1146 LENGTH   no trailing characters  (originally 440mm, 64cm on envelope - handwriting difficult to read)
#> 1554 PIT/Tag# no trailing characters  (corrected)                                                         
#> 1576 PIT/Tag# no trailing characters  (correct)                                                           
#> 2841 PIT/Tag# no trailing characters .90E+13                                                              
#> 2861 PIT/Tag# no trailing characters .89E+14                                                              
#> .... ........ ...................... .....................................................................
#> See problems(...) for more details.
```

To see a tibble of the problems we can do:

``` r
problems(meta)
#> # A tibble: 107 × 4
#>      row      col               expected
#>    <int>    <chr>                  <chr>
#> 1   1146   LENGTH no trailing characters
#> 2   1554 PIT/Tag# no trailing characters
#> 3   1576 PIT/Tag# no trailing characters
#> 4   2841 PIT/Tag# no trailing characters
#> 5   2861 PIT/Tag# no trailing characters
#> 6   2862 PIT/Tag# no trailing characters
#> 7   2863 PIT/Tag# no trailing characters
#> 8   2864 PIT/Tag# no trailing characters
#> 9   2865 PIT/Tag# no trailing characters
#> 10  2866 PIT/Tag# no trailing characters
#> # ... with 97 more rows, and 1 more variables: actual <chr>
```
