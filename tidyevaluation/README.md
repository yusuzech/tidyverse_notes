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
    ## Please use a list of either functions or lambdas: 
    ## 
    ##   # Simple named list: 
    ##   list(mean = mean, median = median)
    ## 
    ##   # Auto named with `tibble::lst()`: 
    ##   tibble::lst(mean, median)
    ## 
    ##   # Using lambdas
    ##   list(~ mean(., trim = .2), ~ median(., na.rm = TRUE))
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

2.Programming with dplyr(symbols, expressions, quosures and strings conversion and evaluation)
----------------------------------------------------------------------------------------------

When Programming with **tidyverse/dplyr/ggplot**, Sometimes we need
conversion between symbols, expressions, quosures and strings. It can be
really annoying if we don’t know what to do. Luckily,with `rlang`
package, we can perform these calculations easily.

Based on my personal understanding the difference between them are:

<table>
<thead>
<tr class="header">
<th style="text-align: left;">Type</th>
<th style="text-align: left;">How to Create</th>
<th style="text-align: left;">Notes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">string</td>
<td style="text-align: left;">var &lt;- “a”</td>
<td style="text-align: left;"></td>
</tr>
<tr class="even">
<td style="text-align: left;">symbols</td>
<td style="text-align: left;">symbol1 &lt;- sym(‘a’);</td>
<td style="text-align: left;">Convert string to symbol</td>
</tr>
<tr class="odd">
<td style="text-align: left;">expression</td>
<td style="text-align: left;">expr1 &lt;- expr(a + 1)</td>
<td style="text-align: left;">Expression will not be evaluated until using <code>eval_tidy()</code></td>
</tr>
<tr class="even">
<td style="text-align: left;">quosure</td>
<td style="text-align: left;">quo1 &lt;- quo(a + 1)</td>
<td style="text-align: left;">Quosure has its original environment attatched, it will always be evaluated in attached environment</td>
</tr>
</tbody>
</table>

Tidy Evaluation in global environment
-------------------------------------

### `sym` and `syms`

`sym` and `syms` Convert string(s) to symbol(s).

#### Singular

    myvar <- 1
    # Convert string to symbol
    sym("myvar")

    ## myvar

    # Evaluate
    eval_tidy(sym("myvar"))

    ## [1] 1

#### Plural

    myvar1 <- 1
    myvar2 <- 2
    # convert strings to symbols
    syms(list("myvar1","myvar2"))

    ## [[1]]
    ## myvar1
    ## 
    ## [[2]]
    ## myvar2

    # evaluate
    purrr::map(syms(list("myvar1","myvar2")),eval_tidy)

    ## [[1]]
    ## [1] 1
    ## 
    ## [[2]]
    ## [1] 2

#### Assignment

##### Assignment using `sym()` and `eval_bare()`

    var_name <- "a"
    var_value <- "assign value using sym()"

    # construct assignment expression
    myexpr <- call2("<-",sym(var_name),var_value)
    myexpr

    ## a <- "assign value using sym()"

    # evaluate
    invisible(eval_bare(myexpr)) # eval_bare prints to console, so use invisible to hide it
    a

    ## [1] "assign value using sym()"

### `expr` and `exprs`

`expr` and `exprs` presrve expressions, it will be evaluated only when
evaluated with `eval_tidy()`

    expr1 <- expr(a + 1) # a hasn't be defined yet
    cat("Expression:\n")

    ## Expression:

    print(expr1)

    ## a + 1

    cat("Evaluate:\n")

    ## Evaluate:

    a <- 1
    eval_tidy(expr1)

    ## [1] 2

    my_expres <- exprs(a+1,b+1)
    cat("Expressions:\n")

    ## Expressions:

    my_expres

    ## [[1]]
    ## a + 1
    ## 
    ## [[2]]
    ## b + 1

    cat("Evaluate:\n")

    ## Evaluate:

    a <- 1 
    b <- 2
    purrr::map(my_expres,eval_tidy)

    ## [[1]]
    ## [1] 2
    ## 
    ## [[2]]
    ## [1] 3

**Assign values using expression**

### `quo` and `quos`
