Extract Tables in Excel Sheet
================
Yifu Yan
3/23/2019

# Extract multiple talbes in the same excel sheet

Many people use excel for analysis and have multiple different tables in
the same worksheet. And it causes plenty of inconveniences reading frome
sheet like this. So I wrote a bunch of functions to help extracting all
tales in the same sheet.

## Utility functions

``` r
# utility function to get rle as a named vector
vec_rle <- function(v){
    temp <- rle(v)
    out <- temp$values
    names(out) <- temp$lengths
    return(out)
}

# utility function to map table with their columns/rows in a bigger table
make_df_index <- function(v){
    table_rle <- vec_rle(v)
    divide_points <- c(0,cumsum(names(table_rle)))
    table_index <- map2((divide_points + 1)[1:length(divide_points)-1],
                        divide_points[2:length(divide_points)],
                        ~.x:.y)
    return(table_index[table_rle])
}

# split a large table in one direction if there are blank columns or rows
split_direction <- function(df,direction = "col"){
    if(direction == "col"){
        col_has_data <- unname(map_lgl(df,~!all(is.na(.x))))
        df_mapping <- make_df_index(col_has_data)
        out <- map(df_mapping,~df[,.x])
    } else if(direction == "row"){
        row_has_data <- df %>% 
            mutate_all(~!is.na(.x)) %>%
            as.matrix() %>% 
            apply(1,any)
        df_mapping <- make_df_index(row_has_data)
        out <- map(df_mapping,~df[.x,])
    }
    return(out)
}

# sometimes different tables are close to each other without empty space in between
# in thoses cases we need to manually insert rows and columns in the dataframe in
# order to make split_df() function to work
insert_row <- function(df,at,on = "below"){
    df <- as_tibble(df)
    if(!is.integer(at)) stop("at should be a integer")
    if(on == "below"){
        
    } else if(on == "above"){
        at <- at - 1
    } else {
        stop("on should be either 'below' or above")
    }
    
    if(at > 0){
        df1 <- df[1:at,]
        df2 <- df[at+1:nrow(df)-at,]
        out <- df1 %>%
            add_row() %>%
            bind_rows(df2)  
    } else if(at == 0) {
        df1 <- df[0,]
        df2 <- df[at+1:nrow(df)-at,]
        out <- df1 %>%
            add_row() %>%
            bind_rows(df2)
    } else {
        stop("Positon 'at' should be at least 1")
    }
    return(out)
}

insert_column <- function(df,at,on = "right"){
    df <- as_tibble(df)
    if(!is.integer(at)) stop("at should be a integer")
    if(on == "right"){
        
    } else if(on == "left"){
        at <- at - 1
    } else {
        stop("on should be either 'right' or left")
    }
    
    if(at > 0){
        df1 <- df[,1:at]
        df1[,1+ncol(df1)] <- NA
        df2 <- df[,at+1:ncol(df)-at]
        out <- df1 %>%
            bind_cols(df2)  
    } else if(at == 0) {
        df1 <- df[,0]
        df1[,1] <- NA
        df2 <- df
        out <- df1 %>%
            bind_cols(df2)
    } else {
        stop("Positon 'at' should be at least 1")
    }
    return(out)
}
```

## Main functions

``` r
# split a large table into smaller tables if there are blank columns or rows
# if you still see entire rows or columns missing. Please increase complexity
split_df <- function(df,showWarnig = TRUE,complexity = 1){
    if(showWarnig){
        warning("Please don't use first row as column names.")
    }
    
    out <- split_direction(df,"col")
    
    for(i in 1 :complexity){
        out <- out %>%
            map(~split_direction(.x,"row")) %>%
            flatten() %>%
            map(~split_direction(.x,"col")) %>%
            flatten()
    }
    return(out)
    
}

# set first row as column names for a data frame and remove the original first row
set_1row_colname <- function(df){
    colnames(df) <- as.character(df[1,])
    out <- df[-1,]
    return(out)
}

#display the rough shape of table in a sheet with multiple tables
display_table_shape <- function(df){
    colnames(df) <- 1:ncol(df)
    
    out <- df %>%
        map_df(~as.numeric(!is.na(.x))) %>%
        gather(key = "x",value = "value") %>%
        mutate(x = as.numeric(x)) %>%
        group_by(x) %>%
        mutate(y = -row_number()) %>%
        ungroup() %>%
        filter(value == 1) %>%
        ggplot(aes(x = x, y = y,fill = value)) +
        geom_tile(fill = "skyblue3") +
        scale_x_continuous(position = "top") +
        theme_void() +
        theme(legend.position="none",
              panel.border = element_rect(colour = "black", fill=NA, size=2))
    return(out)
}
```

## Testing

``` r
all_table <- read_excel("multiple_tables_sheet.xlsx")
split_df(all_table)
```

    ## Warning in split_df(all_table): Please don't use first row as column names.

    ## [[1]]
    ## # A tibble: 3 x 2
    ##   `Metric 1`   `1`
    ##   <chr>      <dbl>
    ## 1 Metric 2    33  
    ## 2 Metric 3     2.5
    ## 3 Metric 4     0.5
    ## 
    ## [[2]]
    ## # A tibble: 7 x 8
    ##    X__7 X__8      X__9      X__10     X__11     X__12    X__13    X__14    
    ##   <dbl> <chr>     <chr>     <chr>     <chr>     <chr>    <chr>    <chr>    
    ## 1    NA A         B         C         D         E        F        G        
    ## 2     1 0.192982~ 0.952270~ 0.880762~ 0.462242~ 0.14485~ 0.42462~ 0.687622~
    ## 3     2 0.579638~ 0.109031~ 0.870814~ 0.322282~ 0.77117~ 0.84536~ 0.934803~
    ## 4     3 0.146197~ 0.735944~ 0.802584~ 0.209307~ 0.16583~ 0.60413~ 0.711807~
    ## 5     4 0.407581~ 0.763435~ 0.702325~ 0.123866~ 0.55333~ 0.57815~ 0.393359~
    ## 6     5 0.496029~ 0.809481~ 0.107272~ 0.736098~ 0.92598~ 0.44667~ 8.426431~
    ## 7     6 0.273132~ 0.635392~ 0.280252~ 0.848825~ 0.35606~ 0.22248~ 0.730208~
    ## 
    ## [[3]]
    ## # A tibble: 5 x 4
    ##    X__5  X__6   X__7 X__8                 
    ##   <dbl> <dbl>  <dbl> <chr>                
    ## 1 0.913 0.847 0.0562 8.9856029499354229E-2
    ## 2 0.306 0.214 0.372  0.6193422103078946   
    ## 3 0.631 0.917 0.276  0.42525120597926203  
    ## 4 0.486 0.326 0.0146 0.30828287612998106  
    ## 5 0.676 0.179 0.946  0.26995561282722147  
    ## 
    ## [[4]]
    ## # A tibble: 5 x 2
    ##   X__14                X__15
    ##   <chr>                <dbl>
    ## 1 <NA>                NA    
    ## 2 0.37161583002087606  0.516
    ## 3 0.50773871789825931  0.154
    ## 4 0.25603147667005288  0.480
    ## 5 <NA>                NA    
    ## 
    ## [[5]]
    ## # A tibble: 5 x 4
    ##   X__10               X__11             X__12             X__13            
    ##   <chr>               <chr>             <chr>             <chr>            
    ## 1 0.62342375649369253 0.96597050194001~ 0.85123985341450~ 0.90694876344258~
    ## 2 0.10482950199074381 0.38701538991483~ 0.44115009542577~ 0.44844181894589~
    ## 3 7.6249795725892966~ 0.25259528040197~ 0.48569495685354~ 0.33711026017650~
    ## 4 0.82568682830384832 0.73779932235336~ 0.38725776704937~ 0.89235522648844~
    ## 5 0.55853586859970705 0.78964666068133~ 0.56849423172288~ 0.86940781979959~

``` r
set_1row_colname(split_df(all_table,showWarnig = F)[[2]])
```

    ## # A tibble: 6 x 8
    ##    `NA` A         B         C         D         E        F        G        
    ##   <dbl> <chr>     <chr>     <chr>     <chr>     <chr>    <chr>    <chr>    
    ## 1     1 0.192982~ 0.952270~ 0.880762~ 0.462242~ 0.14485~ 0.42462~ 0.687622~
    ## 2     2 0.579638~ 0.109031~ 0.870814~ 0.322282~ 0.77117~ 0.84536~ 0.934803~
    ## 3     3 0.146197~ 0.735944~ 0.802584~ 0.209307~ 0.16583~ 0.60413~ 0.711807~
    ## 4     4 0.407581~ 0.763435~ 0.702325~ 0.123866~ 0.55333~ 0.57815~ 0.393359~
    ## 5     5 0.496029~ 0.809481~ 0.107272~ 0.736098~ 0.92598~ 0.44667~ 8.426431~
    ## 6     6 0.273132~ 0.635392~ 0.280252~ 0.848825~ 0.35606~ 0.22248~ 0.730208~

``` r
display_table_shape(all_table)
```

![](read_excel_tables_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
insert_column(mtcars,2L,"right")
```

    ## # A tibble: 32 x 14
    ##      mpg   cyl V3     mpg1  cyl1  disp    hp  drat    wt  qsec    vs    am
    ##    <dbl> <dbl> <lgl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
    ##  1  21       6 NA     21       6  160    110  3.9   2.62  16.5     0     1
    ##  2  21       6 NA     21       6  160    110  3.9   2.88  17.0     0     1
    ##  3  22.8     4 NA     22.8     4  108     93  3.85  2.32  18.6     1     1
    ##  4  21.4     6 NA     21.4     6  258    110  3.08  3.22  19.4     1     0
    ##  5  18.7     8 NA     18.7     8  360    175  3.15  3.44  17.0     0     0
    ##  6  18.1     6 NA     18.1     6  225    105  2.76  3.46  20.2     1     0
    ##  7  14.3     8 NA     14.3     8  360    245  3.21  3.57  15.8     0     0
    ##  8  24.4     4 NA     24.4     4  147.    62  3.69  3.19  20       1     0
    ##  9  22.8     4 NA     22.8     4  141.    95  3.92  3.15  22.9     1     0
    ## 10  19.2     6 NA     19.2     6  168.   123  3.92  3.44  18.3     1     0
    ## # ... with 22 more rows, and 2 more variables: gear <dbl>, carb <dbl>

``` r
insert_column(mtcars,1L,"left")
```

    ## # A tibble: 32 x 12
    ##    V1      mpg   cyl  disp    hp  drat    wt  qsec    vs    am  gear  carb
    ##    <lgl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
    ##  1 NA     21       6  160    110  3.9   2.62  16.5     0     1     4     4
    ##  2 NA     21       6  160    110  3.9   2.88  17.0     0     1     4     4
    ##  3 NA     22.8     4  108     93  3.85  2.32  18.6     1     1     4     1
    ##  4 NA     21.4     6  258    110  3.08  3.22  19.4     1     0     3     1
    ##  5 NA     18.7     8  360    175  3.15  3.44  17.0     0     0     3     2
    ##  6 NA     18.1     6  225    105  2.76  3.46  20.2     1     0     3     1
    ##  7 NA     14.3     8  360    245  3.21  3.57  15.8     0     0     3     4
    ##  8 NA     24.4     4  147.    62  3.69  3.19  20       1     0     4     2
    ##  9 NA     22.8     4  141.    95  3.92  3.15  22.9     1     0     4     2
    ## 10 NA     19.2     6  168.   123  3.92  3.44  18.3     1     0     4     4
    ## # ... with 22 more rows

``` r
insert_row(mtcars,2L,"below")
```

    ## # A tibble: 35 x 11
    ##      mpg   cyl  disp    hp  drat    wt  qsec    vs    am  gear  carb
    ##    <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
    ##  1  21       6   160   110  3.9   2.62  16.5     0     1     4     4
    ##  2  21       6   160   110  3.9   2.88  17.0     0     1     4     4
    ##  3  NA      NA    NA    NA NA    NA     NA      NA    NA    NA    NA
    ##  4  21       6   160   110  3.9   2.62  16.5     0     1     4     4
    ##  5  21       6   160   110  3.9   2.88  17.0     0     1     4     4
    ##  6  22.8     4   108    93  3.85  2.32  18.6     1     1     4     1
    ##  7  21.4     6   258   110  3.08  3.22  19.4     1     0     3     1
    ##  8  18.7     8   360   175  3.15  3.44  17.0     0     0     3     2
    ##  9  18.1     6   225   105  2.76  3.46  20.2     1     0     3     1
    ## 10  14.3     8   360   245  3.21  3.57  15.8     0     0     3     4
    ## # ... with 25 more rows

``` r
insert_row(mtcars,1L,"above")
```

    ## # A tibble: 33 x 11
    ##      mpg   cyl  disp    hp  drat    wt  qsec    vs    am  gear  carb
    ##    <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
    ##  1  NA      NA   NA     NA NA    NA     NA      NA    NA    NA    NA
    ##  2  21       6  160    110  3.9   2.62  16.5     0     1     4     4
    ##  3  21       6  160    110  3.9   2.88  17.0     0     1     4     4
    ##  4  22.8     4  108     93  3.85  2.32  18.6     1     1     4     1
    ##  5  21.4     6  258    110  3.08  3.22  19.4     1     0     3     1
    ##  6  18.7     8  360    175  3.15  3.44  17.0     0     0     3     2
    ##  7  18.1     6  225    105  2.76  3.46  20.2     1     0     3     1
    ##  8  14.3     8  360    245  3.21  3.57  15.8     0     0     3     4
    ##  9  24.4     4  147.    62  3.69  3.19  20       1     0     4     2
    ## 10  22.8     4  141.    95  3.92  3.15  22.9     1     0     4     2
    ## # ... with 23 more rows

# All functions in one place

``` r
# utility function to get rle as a named vector
vec_rle <- function(v){
    temp <- rle(v)
    out <- temp$values
    names(out) <- temp$lengths
    return(out)
}

# utility function to map table with their columns/rows in a bigger table
make_df_index <- function(v){
    table_rle <- vec_rle(v)
    divide_points <- c(0,cumsum(names(table_rle)))
    table_index <- map2((divide_points + 1)[1:length(divide_points)-1],
                        divide_points[2:length(divide_points)],
                        ~.x:.y)
    return(table_index[table_rle])
}

# split a large table in one direction if there are blank columns or rows
split_direction <- function(df,direction = "col"){
    if(direction == "col"){
        col_has_data <- unname(map_lgl(df,~!all(is.na(.x))))
        df_mapping <- make_df_index(col_has_data)
        out <- map(df_mapping,~df[,.x])
    } else if(direction == "row"){
        row_has_data <- df %>% 
            mutate_all(~!is.na(.x)) %>%
            as.matrix() %>% 
            apply(1,any)
        df_mapping <- make_df_index(row_has_data)
        out <- map(df_mapping,~df[.x,])
    }
    return(out)
}

# sometimes different tables are close to each other without empty space in between
# in thoses cases we need to manually insert rows and columns in the dataframe in
# order to make split_df() function to work
insert_row <- function(df,at,on = "below"){
    df <- as_tibble(df)
    if(!is.integer(at)) stop("at should be a integer")
    if(on == "below"){
        
    } else if(on == "above"){
        at <- at - 1
    } else {
        stop("on should be either 'below' or above")
    }
    
    if(at > 0){
        df1 <- df[1:at,]
        df2 <- df[at+1:nrow(df)-at,]
        out <- df1 %>%
            add_row() %>%
            bind_rows(df2)  
    } else if(at == 0) {
        df1 <- df[0,]
        df2 <- df[at+1:nrow(df)-at,]
        out <- df1 %>%
            add_row() %>%
            bind_rows(df2)
    } else {
        stop("Positon 'at' should be at least 1")
    }
    return(out)
}

insert_column <- function(df,at,on = "right"){
    df <- as_tibble(df)
    if(!is.integer(at)) stop("at should be a integer")
    if(on == "right"){
        
    } else if(on == "left"){
        at <- at - 1
    } else {
        stop("on should be either 'right' or left")
    }
    
    if(at > 0){
        df1 <- df[,1:at]
        df1[,1+ncol(df1)] <- NA
        df2 <- df[,at+1:ncol(df)-at]
        out <- df1 %>%
            bind_cols(df2)  
    } else if(at == 0) {
        df1 <- df[,0]
        df1[,1] <- NA
        df2 <- df
        out <- df1 %>%
            bind_cols(df2)
    } else {
        stop("Positon 'at' should be at least 1")
    }
    return(out)
}

# split a large table into smaller tables if there are blank columns or rows
# if you still see entire rows or columns missing. Please increase complexity
split_df <- function(df,showWarnig = TRUE,complexity = 1){
    if(showWarnig){
        warning("Please don't use first row as column names.")
    }
    
    out <- split_direction(df,"col")
    
    for(i in 1 :complexity){
        out <- out %>%
            map(~split_direction(.x,"row")) %>%
            flatten() %>%
            map(~split_direction(.x,"col")) %>%
            flatten()
    }
    return(out)
    
}

# set first row as column names for a data frame and remove the original first row
set_1row_colname <- function(df){
    colnames(df) <- as.character(df[1,])
    out <- df[-1,]
    return(out)
}

#display the rough shape of table in a sheet with multiple tables
display_table_shape <- function(df){
    colnames(df) <- 1:ncol(df)
    
    out <- df %>%
        map_df(~as.numeric(!is.na(.x))) %>%
        gather(key = "x",value = "value") %>%
        mutate(x = as.numeric(x)) %>%
        group_by(x) %>%
        mutate(y = -row_number()) %>%
        ungroup() %>%
        filter(value == 1) %>%
        ggplot(aes(x = x, y = y,fill = value)) +
        geom_tile(fill = "skyblue3") +
        scale_x_continuous(position = "top") +
        theme_void() +
        theme(legend.position="none",
              panel.border = element_rect(colour = "black", fill=NA, size=2))
    return(out)
}
```
