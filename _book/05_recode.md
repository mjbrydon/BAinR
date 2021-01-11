# Recoding Data {#recode}

The purpose of this tutorial is to show you how to recode columns (that is, to create a new column based on the values in one or more other columns).  This is very simple in R and draws on many the same skills you might use when creating a new column in Excel using formulas and functions.

## Preliminaries

As always, we start by making sure we have the bank data loaded as a data frame in R Studio:


```r
library(tidyverse)
Bank <- read_csv("Data/Bank.csv")
```

## Recode a Text Column to a Dummy

A "dummy" or "indicator" variable takes on a value of either 0 or 1. The appeal of these particular values is that they are numerical and can be used with routines that only accept numerical data (such as linear regression). Textual binary variables such as Yes/No or True/False may be easier to read. But they are less amenable to regression.

To add a new column to a tibble like Bank, you just define the column. For example:


```r
Bank$Gender.Dummy <- 0
```

This appends a new column called `Gender.Dummy` to the Bank tibble and sets the value of the new column to 0 for every row. There is nothing special about this new column name: I use the period to remind myself that this is the dummy version of the Gender variable. I could have called it `Gender-Dummy` or anything else.

Of course, setting all the values to 0 is not useful. To get a better value for Gender.Dummy, we have to look at the value of Gender in each row and make a decision. In Excel we use the `if()` function for such decisions. R offers a similar construct called `ifelse()`:


```r
Bank$Gender.Dummy<-ifelse(Bank$Gender=="Male",1,0)
```

If Bank$Gender equals "Male" then the ifelse function returns a value of 1. Otherwise it returns a value of zero. In this way, a textual binary variable can be recoded as a numeric binary variable.

## Recode According to List Membership

Say that we wanted to recode the JobGrade variable, which takes on values 1 through 6, to a _coarser-grained_ binary variable: 0 = non-management and 1 = management. Assume that JobGrade > 4 corresponds to managerial roles.

We could use that condition (JobGrade >) in an ifelse() statement to create a new variable:


```r
Bank$Mgmt.Dummy <- ifelse(Bank$JobGrade > 4, 1, 0)
```

But lets use a more flexible "list membership" approach instead. Create a list of non-management job grades using the combine operator:


```r
nonmgmt=c(1,2,3,4)
```

Now we can use that list plus the is.element() function to distinguish managers from non-managers:


```r
Bank$Mgmt.Dummy <- ifelse(is.element(Bank$JobGrade, nonmgmt),"non-mgmt", "mgmt")
```

The ifelse function checks to see whether Bank$JobGrade is an element of the list nonmgnt (as defined above). If it is, it returns the string "non-mgmt"; otherwise, it returns the string "mgmt". Of course, we could have used 0 and 1 in place of the strings "non-mgmt" and "mgmt".

## Using the tidyverse

Recoding is such a common task in data analysis that the tidyverse's dplyr package has several functions to simplify the process.

### Mutate basics

The following statement appends a binary (dummy, indicator) column to the Bank tibble and then writes the result to the same tibble (in other words, it updates the Bank tibble):


```r
  Bank <- mutate(Bank, Gender.Dummy = if_else(Gender=="Female", 1, 0))
```

Note here that I used the dplyr version of `if_else` rather than R's built-in `ifelse`.  The dplyr version might be a bit safer, but I find they work about the same.

Of course, this could also be written using pipes to make the whole thing _slightly_ more readable:


```r
  Bank <- Bank %>% mutate(Gender.Dummy = if_else(Gender=="Female", 1, 0))
```

What this says (starting on the right of the assignment operator) is:

1.  Start with Bank
2.  Mutate it by adding Gender.Dummy.  The value of Gender.Dummy depends on the value of Gender.
3.  Assign the resulting tibble back to the Bank tibble.

### Recoding categorical data

Considering the following code, which is more illustrative than useful:


```r
Bank %>% mutate(Manager = recode(JobGrade, 
  "1" = "Non-mgmt",
  "2" = "Non-mgmt",
  "3" = "Non-mgmt",
  "4" = "Non-mgmt",
  .default = "Mgmt"))  %>% 
  select(Employee, JobGrade, Gender, Manager)
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
## # A tibble: 208 x 4
##    Employee JobGrade Gender Manager 
##       <dbl>    <dbl> <chr>  <chr>   
##  1        1        1 Male   Non-mgmt
##  2        2        1 Female Non-mgmt
##  3        3        1 Female Non-mgmt
##  4        4        1 Female Non-mgmt
##  5        5        1 Male   Non-mgmt
##  6        6        1 Female Non-mgmt
##  7        7        1 Female Non-mgmt
##  8        8        1 Male   Non-mgmt
##  9        9        1 Female Non-mgmt
## 10       10        1 Female Non-mgmt
## # ... with 198 more rows
```

The statement does the following:

1.  It starts with the Bank data frame (since Bank is piped into `mutate()`)
2.  It creates a new variable called "Manager" and sets its value based on the `recode` function.
3.  The first argument in the `recode` function is the source, JobGrade.  The other arguments are the mappings from old values to new values.  The special mapping ".default" means "everything else".  So here, JobGrade={1,2,3,4} are mapped to "Non-mgmt" and all other values of JobGrade are mapped to "Mgmt".
4.  The results of the `mutate` function are piped to the `select` function.  Select simply limits the columns so the new Manager column shows in the output.  Without it, the tibble is too wide to show in the console.
