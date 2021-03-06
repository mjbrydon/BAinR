# Filtering Data {#filter}

We often want to "subset" our data.  That is, we only want to look at data for a certain year, or from a certain class products or customers.  We generally call this process "filtering" in Excel or "selection" in SQL.  The key idea is that we use some criteria to extract a subset of rows from our data and use only those rows in subsequent analysis.

There are two ways to subset data in R:

1.  Use R's built in data manipulation tools.  These are easily identified by their square bracket `[]` syntax.
2.  Use the dplyr library. Think of dplyr as "data pliers" (where pliers are very useful tools around the house).

I personally find dplyr much easier to use than the square bracket notation, so that is what we will use.

## Preliminaries

### Import data
Import the Bank data in the normal way in R Studio.  You can either use Tools --> Import Dataset from within R Studio or run the command line version of the import functions from the tidyverse.  I typically use the menu the first time but then save the command line version created by R Studio.


```r
library(tidyverse)
Bank <- read_csv("Data/Bank.csv")
```

Click on the Bank tibble in the panel at the top right of R Studio to inspect the contents of the imported file.

## Filters

### Using a logical critereon

The easiest way to filter is to call dplyr's filter function to create a new, smaller tibble:
`<new tibble> <- filter(<tibble>, <critereon>)`

For example:


```r
FemaleEmployees <- dplyr::filter(Bank, Gender=="Female")  ## "dplyr::" not required
View(FemaleEmployees)
```

The new tibble is called FemaleEmployees (although you can call it anything).  The source tibble is, of course, the Bank tibble.  The logical criterion is `Gender=="Female"`.  A few things to note about the logical criterion:

1.  Gender is the name of a column in the Bank tibble.
2.  The logical comparison operator for equals is `==`, not `=`.  This is the convention in many computer programming languages in which the single equals sign is the _assignment_ operator.  In R, `<-` is the assignment operator and `==` is the equals comparison operator.  If you make a mistake in filtering, it is almost always because you use `=` instead of `==`.
3.  "Female" is a literal string.  It means: Only keep rows in which the value of Gender is exactly equal to "Female".  The string "female" is not close enough.  A literal string is a literal string.

### Filtering Using a List

One very powerful trick in R is to extract rows that match a list of values.  For example, say we wanted to extract a list of managers.  In this dataset, managers have a value of JobGrade >= 4, so we could use a logical criterion:


```r
filter(Bank, JobGrade >= 4)
```

```
## Warning: `...` is not empty.
## 
## We detected these problematic arguments:
## * `needs_dots`
## 
## These dots only exist to allow future extensions and should be empty.
## Did you misspecify an argument?
```

```
## # A tibble: 63 x 9
##    Employee EducLev JobGrade YrHired YrBorn Gender YrsPrior PCJob Salary
##       <dbl>   <dbl>    <dbl>   <dbl>  <dbl> <chr>     <dbl> <chr>  <dbl>
##  1      146       5        4      90     62 Male          3 No      44.5
##  2      147       5        4      91     65 Male          1 No      41  
##  3      148       5        4      89     58 Male          3 No      44  
##  4      149       5        4      89     65 Male          0 No      44  
##  5      150       5        4      90     63 Female        4 No      42.5
##  6      151       5        4      88     58 Female        3 No      40.3
##  7      152       5        4      90     66 Male          1 No      44.5
##  8      153       1        4      82     45 Female        9 No      35.5
##  9      154       5        4      89     66 Male          0 No      42.5
## 10      155       5        4      88     63 Female        0 No      44  
## # ... with 53 more rows
```

Note that there is no assignment operator here, so I have not created a new tibble.  R simply summarizes the results in the console window.

The problem with this approach is that it requires job grades to be numeric (and thus ordinal).  I could accomplish the same thing in a more general way using a list of the job grades I want to include:

1.  Create a new vector of managerial job grades using the "combine" function, `c()`.  I call the resulting vector "Mgmt".
2.  Use the `is.element()` function to test membership in the list for each employee.  The full syntax is: `is.element(x, y)`.  The function returns TRUE if `x` is a member of `y` and FALSE otherwise.


```r
Mgmt <- c(4,5,6)
filter(Bank, is.element(JobGrade, Mgmt))
```

```
## Warning: `...` is not empty.
## 
## We detected these problematic arguments:
## * `needs_dots`
## 
## These dots only exist to allow future extensions and should be empty.
## Did you misspecify an argument?
```

```
## # A tibble: 63 x 9
##    Employee EducLev JobGrade YrHired YrBorn Gender YrsPrior PCJob Salary
##       <dbl>   <dbl>    <dbl>   <dbl>  <dbl> <chr>     <dbl> <chr>  <dbl>
##  1      146       5        4      90     62 Male          3 No      44.5
##  2      147       5        4      91     65 Male          1 No      41  
##  3      148       5        4      89     58 Male          3 No      44  
##  4      149       5        4      89     65 Male          0 No      44  
##  5      150       5        4      90     63 Female        4 No      42.5
##  6      151       5        4      88     58 Female        3 No      40.3
##  7      152       5        4      90     66 Male          1 No      44.5
##  8      153       1        4      82     45 Female        9 No      35.5
##  9      154       5        4      89     66 Male          0 No      42.5
## 10      155       5        4      88     63 Female        0 No      44  
## # ... with 53 more rows
```

I did not have to put the members of Mgmt in quotation marks because JobGrade is an integer.  If my list contains text I have to use quotation marks:

`Animals <- c("cat", "dog", "horse", "pig")`

## Syntatic sugar

Many computer languages offer "syntactic sugar": shortcuts that make long or complex commands a bit easier to type.  The tidyverse packages offers a couple of sweeteners.  The important thing to remember about these shortcuts is that they (generally) only work in tidyverse packages.

### Membership

Instead of remembering the syntax of `is.element(x, y)`, you can use the alternative `%in%`.  This makes the filter syntax a bit more readable.  As you see from the output, the results are identical to the un-sweetened version.


```r
filter(Bank, JobGrade %in% Mgmt)
```

```
## Warning: `...` is not empty.
## 
## We detected these problematic arguments:
## * `needs_dots`
## 
## These dots only exist to allow future extensions and should be empty.
## Did you misspecify an argument?
```

```
## # A tibble: 63 x 9
##    Employee EducLev JobGrade YrHired YrBorn Gender YrsPrior PCJob Salary
##       <dbl>   <dbl>    <dbl>   <dbl>  <dbl> <chr>     <dbl> <chr>  <dbl>
##  1      146       5        4      90     62 Male          3 No      44.5
##  2      147       5        4      91     65 Male          1 No      41  
##  3      148       5        4      89     58 Male          3 No      44  
##  4      149       5        4      89     65 Male          0 No      44  
##  5      150       5        4      90     63 Female        4 No      42.5
##  6      151       5        4      88     58 Female        3 No      40.3
##  7      152       5        4      90     66 Male          1 No      44.5
##  8      153       1        4      82     45 Female        9 No      35.5
##  9      154       5        4      89     66 Male          0 No      42.5
## 10      155       5        4      88     63 Female        0 No      44  
## # ... with 53 more rows
```

### Pipes

Pipes are use to solve the problem of nested function calls.  A nested function occurs whenever the argument of `f()` is itself a function `g()`.  As you have probably discovered, it is hard to keep the parentheses straight when you write long statements of the form: `f(g(x))`.

A _pipe_ takes the result of the interior function _then_ pass it along to the exterior function.  So `f(g(x))` can be rewritten using a pipe: `g(x) %>% f`.  This can be helpful for very long, multi-line statements in R.  Just read the pipe operator `%>%` as "THEN".

To illustrate, start with the tibble Bank THEN filter it THEN view it:


```r
Bank %>% filter(JobGrade %in% Mgmt) %>% View
```
