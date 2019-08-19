---
permalink: /script/r/stats/
title: R scripts for basic stats
breadcrumb: Stats
---

return to [Where do I go from here?](../next/)

go to [Getting started with R and RStudio](../)

# R Scripts for Basic Stats

This page contains scripts for several basic statistical tests and one more complex test (ANOVA). This lesson assumes that you have RStudio installed on your computer and are familiar with the lessons [Navigating around in RStudio](../navigate/) and [Introduction to R data structures](../structures/).  If you haven't already studied them, you should look at them first.

In each test, the first line of the script as written retrieves a CSV file from the [DiSC Office's GitHub code repo](https://github.com/HeardLibrary/digital-scholarship) using a URL.  If you are going to use the script with your own data from your hard drive, you will need to substitute the following line for the first line of the script:

```
myDataFrame <- read.csv(file.choose())
```

You'll need to replace `myDataFrame` with whatever name was used for the data frame in the example.  See [Methods for reading CSV data into data frames](https://heardlibrary.github.io/digital-scholarship/script/r/structures/#methods-for-reading-csv-data-into-data-frames) for more information.  

## t-test of means

A t-test of means assesses the difference between two levels of a single factor.  In this example, the factor is gender and the levels are `men` and `women`.  The dependent variable (response) is named `height` and has numeric values.  In order for R to carry out the test, the data need to be arranged with one column for the grouping variable designating gender and a second column containing the height data, like this:  

![](../images/t-test-table-grouping-variable.png)

Data arranged in this manner is called ["tidy data"](https://heardlibrary.github.io/digital-scholarship/script/r/next/#tidy-data).  **Important note:** because of the difference between the way that R handles numeric and string data when it loads it into a data frame, it is important that any grouping variable that you use in the source CSV file contain non-numeric characters.  See [Data types in data frames and tibbles](https://heardlibrary.github.io/digital-scholarship/script/r/structures/#data-types-in-data-frames-and-tibbles) for details. 

Here's the script to perform the t-test of means on [these data](https://github.com/HeardLibrary/digital-scholarship/blob/master/data/r/t-test.csv):

```
heightsDframe = read.csv(file="https://raw.githubusercontent.com/HeardLibrary/digital-scholarship/master/data/r/t-test.csv")
t.test(height ~ grouping, data=heightsDframe, var.equal=TRUE, conf.level=0.95)
```

You can run the script by pasting it into the editor pane, highlighting (selecting) both lines, and clicking the **Run** button.  You could also enter each of the two lines one at a time in the Console pane, but that would be more work.  If you have forgotten how the RStudio GUI is laid out, please review [The RStudio GUI](https://heardlibrary.github.io/digital-scholarship/script/r/navigate/#the-rstudio-gui).

The results of the test show up in the Console pane.  The *P*-value of 0.01711 indicates that the heights of men and women are significantly different based on these data.  

In `height ~ grouping` of the second line of the script, `height` is the name of the column containing the data and `grouping` is the name of the column containing the grouping variable.  We can see that the columns in the data frame are laid out in this way by clicking on the name of the data frame (`heightsDframe`) in the workspace summary in the upper right pane of RStudio.  The data frame will be displayed in table form in the upper left pane where you can verify that `height` and `grouping` are actually the names used in the headers of the appropriate columns.  If the data you want to analyze have different column headers, you'll need to change the script to the appropriate column names.  

## paired t-test

A paired t-test differs from a t-test of means in that each particular observation in one of the two groups is paired in some way with a particular observation in the other group. 

In R, the pairing of observations is done by placing the observations of the two groups in two vectors.  Because the items in the vectors are ordered, R knows to associate the first item in the first vector with the first item in the second vector, etc.  The general form of the function for paired t-test is:

```
t.test(firstVector, secondVector, paired=TRUE)
```

If we want to read in the values from a CSV file, we'll want to lay out the file differently from the way we laid out the data for the t-test of means:

![](../images/paired-samples.png)

The data for each group are located in a separate column since we will want to be able to refer the data in a column by name.  This example has a third column with a letter identifying each pair, but it isn't necessary for the test and R will simply ignore it. Here's how we can perform the test on [these data](https://github.com/HeardLibrary/digital-scholarship/blob/master/data/r/paired-t.csv) comparing enzyme reaction rates in the presence and absence of malonate:

```
malonateDframe = read.csv(file="https://raw.githubusercontent.com/HeardLibrary/digital-scholarship/master/data/r/paired-t.csv")
t.test(malonateDframe$no_malonate, malonateDframe$malonate, paired=TRUE)
```

Notice that in this example, we referred to the data in a column using the `$` notation (e.g. `malonateDframe$no_malonate` for the data in the "no_malonate" column).  R treats the column data as it would a vector when it inputs it into the `t.test` function.

## chi-squared goodness of fit test

When running a chi-squared goodness of fit test using R, the actual frequencies (i.e. the observed frequencies) must be absolute (i.e. counts).  The expected frequencies must be relative (i.e. the probabilities or proportions expressed as decimal fractions).  This differs from the way the test is conducted in Excel where both the actual and expected frequencies must be absolute.  For example, an Excel example to calculate whether coin flips differ from the expected equal probability of heads and tails looks like this:

![](../images/goodness-of-fit-excel-screenshot.png)

while the setup in R would look like this:

![](../images/goodness-of-fit-r-screenshot.png)

To carry out the test, we need two vectors: one containing the observed values as counts and another containing the expected values as relative frequencies.  For the example above, here's how we would do the test:

```
observedFlips = c(46, 41)        # actual absolute: (heads, tails)
expectedProb = c(0.5, 0.5)      # expected relative: (heads, tails)
chisq.test(x = observedFlips, p = expectedProb)
```

Note that the order of the items in the observed vector have to correspond to the order in the expected vector.

## chi-squared contingency test

Here is [a contingency table for data](https://github.com/HeardLibrary/digital-scholarship/blob/master/data/r/contingency.csv) to test whether the sex of the second child in families is independent of the sex of the first child.  

![](../images/contingency-table.png)

In R, the data for a contingency test must be in the form of a data structure called a *matrix* Details of the matrix data structure are beyond the scope of this lesson - we will simply convert the data from a data frame into a matrix using a function.  Here is the script:

```
childrenDframe = read.csv(file="https://raw.githubusercontent.com/HeardLibrary/digital-scholarship/master/data/r/contingency.csv")
childrenMatrix <- as.matrix(childrenDframe[,-1]) # convert data frame to matrix, removing first column
chisq.test(childrenMatrix, correct=FALSE)      # don't do Yates' correction
```

The first column of the table is to help us keep the combinations straight, but is not needed by R.  So when we convert the data frame into a matrix in the second line of the script, the `-1` removes the first column before the conversion.  

In the third line of the script, `correct=FALSE` turns of Yates' correction which can be used when contingency tests have very values.  

## ANOVA

The setup of a data table to carry out an ANOVA in R uses grouping variables in the same manner as the t-test of means.  Please review the description of grouping variables and the warning about numeric values in groupin variables in the [section about running a t-test of means](#t-test-of-means).  

### Single factor ANOVA



----
Revised 2019-08-19