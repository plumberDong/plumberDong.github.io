# Tidyprogramming

```r
library(dplyr)
```

我们将Variables分为三类：

- enviroment-variables: 在环境当中，可以直接调用的，不需要特殊交代
- data-variables：一般前面要加`$`才能调用的
- character: 以字符串形式出现的变量名

## 1 输入单个变量

在函数中使用data-variables，要加`{{}}`:


```r
var_summary <- function(data, var){
  data %>%
    summarise(n = n(), min = min({{var}}), max = max({{var}}))
}

mtcars %>%
  group_by(cyl) %>%
  var_summary(mpg)
```

```
## # A tibble: 3 × 4
##     cyl     n   min   max
##   <dbl> <int> <dbl> <dbl>
## 1     4    11  21.4  33.9
## 2     6     7  17.8  21.4
## 3     8    14  10.4  19.2
```

在函数中使用character，要加`.data[[]]`:


```r
var_summary <- function(data, var){
  data %>%
    summarise(n = n(), min = min(.data[[var]]), max = max(.data[[var]]))
}

mtcars %>%
  group_by(cyl) %>%
  var_summary("mpg")
```

```
## # A tibble: 3 × 4
##     cyl     n   min   max
##   <dbl> <int> <dbl> <dbl>
## 1     4    11  21.4  33.9
## 2     6     7  17.8  21.4
## 3     8    14  10.4  19.2
```

## 选择多个变量

对于data-variables：


```r
summarise_mean <- function(data, vars) {
  data %>% summarise(n = n(), across({{ vars }}, mean))
}

mtcars %>% 
  group_by(cyl) %>% 
  summarise_mean(where(is.numeric))  # 对所有数值变量进行选择
```

```
## # A tibble: 3 × 12
##     cyl     n   mpg  disp    hp  drat    wt  qsec    vs    am  gear  carb
##   <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
## 1     4    11  26.7  105.  82.6  4.07  2.29  19.1 0.909 0.727  4.09  1.55
## 2     6     7  19.7  183. 122.   3.59  3.12  18.0 0.571 0.429  3.86  3.43
## 3     8    14  15.1  353. 209.   3.23  4.00  16.8 0     0.143  3.29  3.5
```

```r
mtcars %>% 
  summarise_mean(1:3)  # 对1到3列的变量进行选择
```

```
##    n      mpg    cyl     disp
## 1 32 20.09062 6.1875 230.7219
```

```r
mtcars %>%
  summarise_mean(c(cyl, disp, mpg)) # 对这三个变量进行选择
```

```
##    n    cyl     disp      mpg
## 1 32 6.1875 230.7219 20.09062
```

对于character，加一个`all_of`就可以：


```r
vars <- c("mpg", "vs")
mtcars %>% select(all_of(vars)) %>% head()
```

```
##                    mpg vs
## Mazda RX4         21.0  0
## Mazda RX4 Wag     21.0  0
## Datsun 710        22.8  1
## Hornet 4 Drive    21.4  1
## Hornet Sportabout 18.7  0
## Valiant           18.1  1
```

```r
mtcars %>% select(!all_of(vars)) %>% head() # !是反向选择
```

```
##                   cyl disp  hp drat    wt  qsec am gear carb
## Mazda RX4           6  160 110 3.90 2.620 16.46  1    4    4
## Mazda RX4 Wag       6  160 110 3.90 2.875 17.02  1    4    4
## Datsun 710          4  108  93 3.85 2.320 18.61  1    4    1
## Hornet 4 Drive      6  258 110 3.08 3.215 19.44  0    3    1
## Hornet Sportabout   8  360 175 3.15 3.440 17.02  0    3    2
## Valiant             6  225 105 2.76 3.460 20.22  0    3    1
```

