## 1. Create(`muate()`) sum and percentage of sum for columns
==========================================================

Instead of doing:

``` r
mtcars %>%
    group_by(cyl) %>%
    transmute(mpg_sum = sum(mpg),
           disp_sum = sum(disp),
           mpg_pct = mpg/sum(mpg),
           disp_pct = disp/sum(disp)) %>%
    head()
```

    ## # A tibble: 6 x 5
    ## # Groups:   cyl [3]
    ##     cyl mpg_sum disp_sum mpg_pct disp_pct
    ##   <dbl>   <dbl>    <dbl>   <dbl>    <dbl>
    ## 1     6    138.    1283.  0.152    0.125 
    ## 2     6    138.    1283.  0.152    0.125 
    ## 3     4    293.    1156.  0.0777   0.0934
    ## 4     6    138.    1283.  0.155    0.201 
    ## 5     8    211.    4943.  0.0885   0.0728
    ## 6     6    138.    1283.  0.131    0.175

Do:

``` r
mtcars %>%
    group_by(cyl) %>%
    transmute_at(vars(mpg,disp),
              funs(sum = sum(.),
                   pct = ./sum(.))) %>%
    head()
```

    ## # A tibble: 6 x 5
    ## # Groups:   cyl [3]
    ##     cyl mpg_sum disp_sum mpg_pct disp_pct
    ##   <dbl>   <dbl>    <dbl>   <dbl>    <dbl>
    ## 1     6    138.    1283.  0.152    0.125 
    ## 2     6    138.    1283.  0.152    0.125 
    ## 3     4    293.    1156.  0.0777   0.0934
    ## 4     6    138.    1283.  0.155    0.201 
    ## 5     8    211.    4943.  0.0885   0.0728
    ## 6     6    138.    1283.  0.131    0.175
