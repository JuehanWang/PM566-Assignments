---
title: "Assignment04"
author: "Juehan Wang"
date: "11/15/2021"
output: 
    html_document:
      toc: yes 
      toc_float: yes
      keep_md : yes 
    github_document:
      html_preview: false
always_allow_html: true
---





# HPC

## Problem 1: Make sure your code is nice

Rewrite the following R functions to make them faster. It is OK (and recommended) to take a look at Stackoverflow and Google


```r
# Total row sums
fun1 <- function(mat) {
  n <- nrow(mat)
  ans <- double(n) 
  for (i in 1:n) {
    ans[i] <- sum(mat[i, ])
  }
  ans
}

fun1alt <- function(mat) {
  ans <- rowSums(mat)
  ans
}

# Cumulative sum by row
fun2 <- function(mat) {
  n <- nrow(mat)
  k <- ncol(mat)
  ans <- mat
  for (i in 1:n) {
    for (j in 2:k) {
      ans[i,j] <- mat[i, j] + ans[i, j - 1]
    }
  }
  ans
}

fun2alt <- function(mat) {
  ans <- t(apply(mat, 1, cumsum))
  ans
}


# Use the data with this code
set.seed(2315)
dat <- matrix(rnorm(200 * 100), nrow = 200)

# Test for the first
microbenchmark::microbenchmark(
  fun1(dat),
  fun1alt(dat), unit = "relative", check = "equivalent"
)
```

```
## Unit: relative
##          expr      min       lq     mean   median       uq       max neval cld
##     fun1(dat) 7.080224 8.130512 6.492715 8.266989 8.172635 0.5677914   100   b
##  fun1alt(dat) 1.000000 1.000000 1.000000 1.000000 1.000000 1.0000000   100  a
```

```r
# Test for the second
microbenchmark::microbenchmark(
  fun2(dat),
  fun2alt(dat), unit = "relative", check = "equivalent"
)
```

```
## Unit: relative
##          expr     min       lq     mean   median       uq       max neval cld
##     fun2(dat) 3.52856 2.228512 1.548358 2.036395 1.931973 0.1812428   100   b
##  fun2alt(dat) 1.00000 1.000000 1.000000 1.000000 1.000000 1.0000000   100  a
```

From the test of speed, both of the new codes run faster.

## Problem 2: Make things run faster with parallel computing

The following function allows simulating PI


```r
sim_pi <- function(n = 1000, i = NULL) {
  p <- matrix(runif(n*2), ncol = 2)
  mean(rowSums(p^2) < 1) * 4
}

# Here is an example of the run
set.seed(156)
sim_pi(1000) # 3.132
```

```
## [1] 3.132
```

In order to get accurate estimates, we can run this function multiple times, with the following code:


```r
# This runs the simulation a 4,000 times, each with 10,000 points
set.seed(1231)
system.time({
  ans <- unlist(lapply(1:4000, sim_pi, n = 10000))
  print(mean(ans))
})
```

```
## [1] 3.14124
```

```
##    user  system elapsed 
##   1.915   0.865   2.788
```

Rewrite the previous code using parLapply() to make it run faster. Make sure you set the seed using clusterSetRNGStream():


```r
cl <- makePSOCKcluster(4L)
clusterSetRNGStream(cl,1231)
clusterExport(cl,"sim_pi")
system.time({
  ans <- unlist(parLapply(cl, rep(4000,10000),sim_pi))
  print(mean(ans))
})
```

```
## [1] 3.141521
```

```
##    user  system elapsed 
##   0.004   0.000   0.842
```

Compared to the previous code, the code we rewrite runs faster.

# SQL

Setup a temporary database by running the following chunk



## Question 1

How many movies is there available in each rating category.


```r
dbGetQuery(con, "
SELECT rating AS Rating, COUNT (*) AS Count
FROM film
GROUP BY rating")
```

```
##   Rating Count
## 1      G   180
## 2  NC-17   210
## 3     PG   194
## 4  PG-13   223
## 5      R   195
```

The result of the number of movies available in each rating category is shown in the table.

## Question 2

What is the average replacement cost and rental rate for each rating category.


```r
dbGetQuery(con, "
SELECT rating AS Rating, AVG(replacement_cost) AS Replacement_cost_avg,
AVG(rental_rate) AS Rental_rate_avg
FROM film
GROUP BY rating
LIMIT 10")
```

```
##   Rating Replacement_cost_avg Rental_rate_avg
## 1      G             20.12333        2.912222
## 2  NC-17             20.13762        2.970952
## 3     PG             18.95907        3.051856
## 4  PG-13             20.40256        3.034843
## 5      R             20.23103        2.938718
```

The result of the average replacement cost and rental rate for each rating category is shown in the table.

## Question 3

Use table film_category together with film to find the how many films there are with each category ID


```r
dbGetQuery(con, "SELECT category_id AS Category_ID,
  COUNT (*) as Count
FROM film_category
GROUP BY category_id
LIMIT 10")
```

```
##    Category_ID Count
## 1            1    64
## 2            2    66
## 3            3    60
## 4            4    57
## 5            5    58
## 6            6    68
## 7            7    62
## 8            8    69
## 9            9    73
## 10          10    61
```

The result of the number of films with each first 10 category ID is shown in the table.

## Question 4

Incorporate table category into the answer to the previous question to find the name of the most popular category.


```r
dbGetQuery(con, "
SELECT film_category.category_id AS Category_id, category.name AS Name, COUNT(*) AS Count
FROM film_category
  INNER JOIN film ON film_category.film_id = film.film_id
  INNER JOIN category ON film_category.category_id = category.category_id
GROUP BY category.category_ID
ORDER BY count DESC
LIMIT 5
")
```

```
##   Category_id        Name Count
## 1          15      Sports    74
## 2           9     Foreign    73
## 3           8      Family    69
## 4           6 Documentary    68
## 5           2   Animation    66
```


```r
dbDisconnect(con)
```

The result of the number of films with each first 10 category name is shown in the table.

Therefore, the most popular category is Sports.
