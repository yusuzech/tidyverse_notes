# Conversion between different number systems

```
library(tidyverse) # for map function
to_decimal <- function(number,origin_system = 2){
    digits <- strsplit(as.character(number),split = "")[[1]]
    digit_dict <- c("A"=10,"B"=11,"C"=12,"D"=13,"E"=14,"F"=15)
    digits <- map_chr(digits,~ifelse(.x %in% as.character(0:10),.x,digit_dict[.x]))
    power <- seq((length(digits)-1),0)
    return(sum(map2_dbl(digits,power,~as.numeric(.x)*origin_system^.y)))
}

to_other_system <- function(number,target_system = 2){
    digit_dict <- c("10"="A","11"="B","12"="C","13"="D","14"="E","15"="F")
    new_digits <- c()
    flag <- TRUE
    result <- number
    while(flag){
        reminder <- result %% target_system
        result <- result %/% target_system
        new_digits <- append(new_digits,reminder)
        if(result == 0){
            flag <- FALSE
        }
    }
    converted_digits <- map_chr(as.character(rev(new_digits)),~ifelse(.x %in% as.character(0:10),.x,digit_dict[.x]))
    return(paste0(converted_digits,collapse = ""))
}

convert_number_system <- function(from,origin_number_system,target_number_system){
    return(to_other_system(to_decimal(from,origin_number_system),target_number_system))
}
```


## Test

```
> convert_number_system("FCAE",16,8)
[1] "176256"
```
