1.Create(`muate()`) sum and percentage of sum for columns
---------------------------------------------------------

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
