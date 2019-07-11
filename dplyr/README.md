1.Usage of `mutate_all()`, `mutate_if()` and `mutate_at()`
----------------------------------------------------------

Use scoped variants of `summarise()`, `mutate(`) and `transmute()` when
handling many columns.

1.  `.predicate`:
    1.  `_at()` variants accepts `vars()` for selecting columns.
    2.  `_if()` variants accepts logical conditions for selecting
        columns.
2.  `.funs`:
    1.  Use `funs()`, which support renaming new columns
    2.  Use function name(e.g. sum,mean)
    3.  Use purrr syntax(`~.x`)

<!-- -->

    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(rlang)

Instead of doing:

    mtcars %>%
        group_by(cyl) %>%
        transmute(mpg_sum = sum(mpg),
               disp_sum = sum(disp),
               mpg_pct = mpg/sum(mpg),
               disp_pct = disp/sum(disp)) %>%
        head() %>%
        knitr::kable()

<table>
<thead>
<tr class="header">
<th style="text-align: right;">cyl</th>
<th style="text-align: right;">mpg_sum</th>
<th style="text-align: right;">disp_sum</th>
<th style="text-align: right;">mpg_pct</th>
<th style="text-align: right;">disp_pct</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: right;">6</td>
<td style="text-align: right;">138.2</td>
<td style="text-align: right;">1283.2</td>
<td style="text-align: right;">0.1519537</td>
<td style="text-align: right;">0.1246883</td>
</tr>
<tr class="even">
<td style="text-align: right;">6</td>
<td style="text-align: right;">138.2</td>
<td style="text-align: right;">1283.2</td>
<td style="text-align: right;">0.1519537</td>
<td style="text-align: right;">0.1246883</td>
</tr>
<tr class="odd">
<td style="text-align: right;">4</td>
<td style="text-align: right;">293.3</td>
<td style="text-align: right;">1156.5</td>
<td style="text-align: right;">0.0777361</td>
<td style="text-align: right;">0.0933852</td>
</tr>
<tr class="even">
<td style="text-align: right;">6</td>
<td style="text-align: right;">138.2</td>
<td style="text-align: right;">1283.2</td>
<td style="text-align: right;">0.1548480</td>
<td style="text-align: right;">0.2010599</td>
</tr>
<tr class="odd">
<td style="text-align: right;">8</td>
<td style="text-align: right;">211.4</td>
<td style="text-align: right;">4943.4</td>
<td style="text-align: right;">0.0884579</td>
<td style="text-align: right;">0.0728244</td>
</tr>
<tr class="even">
<td style="text-align: right;">6</td>
<td style="text-align: right;">138.2</td>
<td style="text-align: right;">1283.2</td>
<td style="text-align: right;">0.1309696</td>
<td style="text-align: right;">0.1753429</td>
</tr>
</tbody>
</table>

Do:

    mtcars %>%
        group_by(cyl) %>%
        transmute_at(vars(mpg,disp),
                  funs(sum = sum(.),
                       pct = ./sum(.))) %>%
        head() %>%
        knitr::kable()

    ## Warning: funs() is soft deprecated as of dplyr 0.8.0
    ## please use list() instead
    ## 
    ##   # Before:
    ##   funs(name = f(.))
    ## 
    ##   # After: 
    ##   list(name = ~ f(.))
    ## This warning is displayed once per session.

<table>
<thead>
<tr class="header">
<th style="text-align: right;">cyl</th>
<th style="text-align: right;">mpg_sum</th>
<th style="text-align: right;">disp_sum</th>
<th style="text-align: right;">mpg_pct</th>
<th style="text-align: right;">disp_pct</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: right;">6</td>
<td style="text-align: right;">138.2</td>
<td style="text-align: right;">1283.2</td>
<td style="text-align: right;">0.1519537</td>
<td style="text-align: right;">0.1246883</td>
</tr>
<tr class="even">
<td style="text-align: right;">6</td>
<td style="text-align: right;">138.2</td>
<td style="text-align: right;">1283.2</td>
<td style="text-align: right;">0.1519537</td>
<td style="text-align: right;">0.1246883</td>
</tr>
<tr class="odd">
<td style="text-align: right;">4</td>
<td style="text-align: right;">293.3</td>
<td style="text-align: right;">1156.5</td>
<td style="text-align: right;">0.0777361</td>
<td style="text-align: right;">0.0933852</td>
</tr>
<tr class="even">
<td style="text-align: right;">6</td>
<td style="text-align: right;">138.2</td>
<td style="text-align: right;">1283.2</td>
<td style="text-align: right;">0.1548480</td>
<td style="text-align: right;">0.2010599</td>
</tr>
<tr class="odd">
<td style="text-align: right;">8</td>
<td style="text-align: right;">211.4</td>
<td style="text-align: right;">4943.4</td>
<td style="text-align: right;">0.0884579</td>
<td style="text-align: right;">0.0728244</td>
</tr>
<tr class="even">
<td style="text-align: right;">6</td>
<td style="text-align: right;">138.2</td>
<td style="text-align: right;">1283.2</td>
<td style="text-align: right;">0.1309696</td>
<td style="text-align: right;">0.1753429</td>
</tr>
</tbody>
</table>

2.Programming with dplyr(variable, argument and string conversions)
-------------------------------------------------------------------

When Programming with **tidyverse(dplyr,ggplot)**, we need to make
conversions betwwen string, argument and variable frequently. It can be
really annoying if we don’t know what to do. Luckily,with `rlang`
package, we can perform these calculations easily.

    library(dplyr)
    library(rlang)

### No functions

#### Convert String to Variable

    x <- c(1,2,3) 
    y <- c("a","b","c")
    # convert and evaluate
    print(eval_tidy(sym("x")))

    ## [1] 1 2 3

    # convert a list of string to variable and evaluate
    for(v in syms(list("x","y"))){
        print(eval_tidy(v))
    }

    ## [1] 1 2 3
    ## [1] "a" "b" "c"

#### Convert variable to string

    quo(x) %>%
        as_name()

    ## [1] "x"

    #convert a list of variable to string
    for(x in quos(x,y,z)){
        print(as_name(x))
    }

    ## [1] "x"
    ## [1] "y"
    ## [1] "z"

#### Evaluate string as code

    #evaluate string as code
    print(eval_tidy(parse_expr("sum(c(1,2,3))")))

    ## [1] 6

    # one more conversion is required is the string is saved to a variable
    x <- "sum(c(1,2,3))"
    quo(x) %>%
        eval_tidy() %>%
        parse_expr() %>%
        eval_tidy() %>%
        print()

    ## [1] 6

    # evaluate a "list" of string as code
    x <- 1
    y <- 2
    z <- 3
    for(arg in parse_exprs("x == 1;y > 1;is.numeric(z)")){
        print(eval_tidy(arg)) # default environment: global_env()
    }

    ## [1] TRUE
    ## [1] TRUE
    ## [1] TRUE

    # use custom data
    for(arg in parse_exprs("x == 1;y > 1;is.numeric(z)")){
        print(eval_tidy(arg,data = list(x=2,y=0,z="a"))) # custom data
    }

    ## [1] FALSE
    ## [1] FALSE
    ## [1] FALSE

#### Turn code into string

    x <- 5
    quo(x+5) %>% 
        quo_text()

    ## [1] "x + 5"

    for(arg in quos(x+5,y >2,z != 3)){
        print(quo_text(arg))
    }

    ## [1] "x + 5"
    ## [1] "y > 2"
    ## [1] "z != 3"

### In functions

#### Turn string into bariable

    library(dplyr)
    library(rlang)

    get_avg <- function(g){
        g <- sym(g)
        return(
            iris %>%
                group_by(!!g) %>%
                summarise(avg = mean(Sepal.Length))
        )
    }


    get_avg("Species")

    ## # A tibble: 3 x 2
    ##   Species      avg
    ##   <fct>      <dbl>
    ## 1 setosa      5.01
    ## 2 versicolor  5.94
    ## 3 virginica   6.59
