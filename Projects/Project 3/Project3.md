Ryan Tillis - R Programming - Hospital Quality - Coursera
================
<a href="http://www.ryantillis.com"> Ryan Tillis </a>
7/1/2016

Plot the 30-day mortality rates for heart attack
------------------------------------------------

Read the outcome data into R via the read.csv function and look at the first few rows.

``` r
rates <- read.csv("outcome-of-care-measures.csv", colClasses = "character")
```

There are many columns in this dataset. You can see how many by typing ncol(outcome) (you can see the number of rows with the nrow function). In addition, you can see the names of each column by typing names(outcome) (the names are also in the PDF document. To make a simple histogram of the 30-day death rates from heart attack (column 11 in the outcome dataset), run

``` r
rates[, 11] <- as.numeric(rates[, 11])
```

    ## Warning: NAs introduced by coercion

``` r
## You may get a warning about NAs being introduced; that is okay
hist(rates[, 11])
```

![](Project3_files/figure-markdown_github/hist-1.png)

Because we originally read the data in as character (by specifying colClasses = "character" we need to coerce the column to be numeric. You may get a warning about NAs being introduced but that is okay.

#### Submission

``` r
url = "https://d396qusza40orc.cloudfront.net/rprog%2Fdata%2FProgAssignment3-data.zip"
download.file(url, destfile = "temp")
unzip("temp")
unlink("temp")
```

Finding the best hospital in a state
------------------------------------

Write a function called best that take two arguments: the 2-character abbreviated name of a state and an outcome name. The function reads the outcome-of-care-measures.csv file and returns a character vector with the name of the hospital that has the best (i.e. lowest) 30-day mortality for the specified outcome in that state. The hospital name is the name provided in the Hospital.Name variable. The outcomes can be one of “heart attack”, “heart failure”, or “pneumonia”. Hospitals that do not have data on a particular outcome should be excluded from the set of hospitals when deciding the rankings.

Handling ties. If there is a tie for the best hospital for a given outcome, then the hospital names should be sorted in alphabetical order and the first hospital in that set should be chosen (i.e. if hospitals “b”, “c”, and “f” are tied for best, then hospital “b” should be returned). The function should use the following template.

``` r
best <- function(state, outcome) {
        ## Read outcome data

       ## Check that state and outcome are valid
       ## Return hospital name in that state with lowest 30-day death
       ## rate
}
```

The function should check the validity of its arguments. If an invalid state value is passed to best, the function should throw an error via the stop function with the exact message “invalid state”. If an invalid outcome value is passed to best, the function should throw an error via the stop function with the exact message “invalid outcome”. Here is some sample output from the function.

#### Submission

``` r
require(plyr)
```

    ## Loading required package: plyr

``` r
hosp_sort <- function(state,outcome){
       #setting NA to be 0
       rates[rates == "Not Available"] <- 0
       index <- c(grep("^Hospital.*Death*", names(rates)))
       mortality_rates <- rates[,c(2,7,index)]
       names(mortality_rates)[3:5] <- c("heart attack", "heart failure", "pneumonia")
       
       #Making rates numeric
       mortality_rates[,3:5] <- apply(mortality_rates[,3:5],2,as.numeric)
       
       mortality_rates[mortality_rates == 0] <- NA
       selected_state <- mortality_rates[mortality_rates$State == state,]
       
       order_selected <- arrange(selected_state, selected_state[,outcome], Hospital.Name, na.last=TRUE)
       order_selected <- order_selected[complete.cases(order_selected[,outcome]),]
       return(order_selected)
        #na.omit(selected_state[order(c(selected_state[,outcome]), na.last = TRUE),])
}

best <- function(state, outcome, st = "a") {
       if (!state %in% unique(rates$State))
              return("Invalid State")
       if (!outcome %in% c("heart attack", "heart failure", "pneumonia"))
              return("Invalid Outcome")
       
       order_selected <- hosp_sort(state,outcome)
       return(order_selected[1,1])

}
```

#### Testing

``` r
best("TX", "heart attack")
```

    ## [1] "CYPRESS FAIRBANKS MEDICAL CENTER"

``` r
best("TX", "heart failure")
```

    ## [1] "FORT DUNCAN MEDICAL CENTER"

``` r
best("MD", "heart attack")
```

    ## [1] "JOHNS HOPKINS HOSPITAL, THE"

``` r
best("MD", "pneumonia")
```

    ## [1] "GREATER BALTIMORE MEDICAL CENTER"

``` r
best("BB", "heart attack")
```

    ## [1] "Invalid State"

``` r
best("NY", "hert attack")
```

    ## [1] "Invalid Outcome"

Ranking hospitals by outcome in a state
---------------------------------------

Write a function called rankhospital that takes three arguments: the 2-character abbreviated name of a state (state), an outcome (outcome), and the ranking of a hospital in that state for that outcome (num). The function reads the outcome-of-care-measures.csv file and returns a character vector with the name of the hospital that has the ranking specified by the num argument. For example, the call

``` r
rankhospital("MD", "heart failure", 5)
```

would return a character vector containing the name of the hospital with the 5th lowest 30-day death rate for heart failure. The num argument can take values “best”, “worst”, or an integer indicating the ranking (smaller numbers are better). If the number given by num is larger than the number of hospitals in that state, then the function should return NA. Hospitals that do not have data on a particular outcome should be excluded from the set of hospitals when deciding the rankings.

Handling ties. It may occur that multiple hospitals have the same 30-day mortality rate for a given cause of death. In those cases ties should be broken by using the hospital name. For example, in Texas (“TX”), the hospitals with lowest 30-day mortality rate for heart failure are shown here.

``` r
head(texas)
```

Note that Cypress Fairbanks Medical Center and Detar Hospital Navarro both have the same 30-day rate (8.7). However, because Cypress comes before Detar alphabetically, Cypress is ranked number 3 in this scheme and Detar is ranked number 4. One can use the order function to sort multiple vectors in this manner (i.e. where one vector is used to break ties in another vector). The function should use the following template.

``` r
rankhospital <- function(state, outcome, num = "best") {
        ## Read outcome data
       ## Check that state and outcome are valid
       ## Return hospital name in that state with the given rank
       ## 30-day death rate
       
}
```

The function should check the validity of its arguments. If an invalid state value is passed to best, the function should throw an error via the stop function with the exact message “invalid state”. If an invalid outcome value is passed to best, the function should throw an error via the stop function with the exact message “invalid outcome”.

#### Submission

``` r
worst <- function(state, outcome, st = "a") {
       if (!state %in% unique(rates$State))
              return("Invalid State")
       if (!outcome %in% c("heart attack", "heart failure", "pneumonia"))
              return("Invalid Outcome")
       
       order_selected <- hosp_sort(state,outcome)
       
       return(order_selected[nrow(order_selected),1])
}

rankhospital <- function(state, outcome, num = "best", st = "a"){
       if (num == "best")
              return(best(state,outcome))
       if (num == "worst")
              return(worst(state,outcome))       
       else {order_selected <- hosp_sort(state,outcome) 
              return(order_selected[num,1])}
}
```

#### Testing

Here is some sample output from the function.

Ranking hospitals in all states
-------------------------------

Write a function called rankall that takes two arguments: an outcome name (outcome) and a hospital rank- ing (num). The function reads the outcome-of-care-measures.csv file and returns a 2-column data frame containing the hospital in each state that has the ranking specified in num. For example the function call rankall("heart attack", "best") would return a data frame containing the names of the hospitals that are the best in their respective states for 30-day heart attack death rates. The function should return a value for every state (some may be NA). The first column in the data frame is named hospital, which contains the hospital name, and the second column is named state, which contains the 2-character abbreviation for the state name. Hospitals that do not have data on a particular outcome should be excluded from the set of hospitals when deciding the rankings. Handling ties. The rankall function should handle ties in the 30-day mortality rates in the same way that the rankhospital function handles ties. The function should use the following template.

``` r
rankall <- function(outcome, num = "best") {
        ## Read outcome data
       ## Check that state and outcome are valid
       ## For each state, find the hospital of the given rank
       ## Return a data frame with the hospital names and the
       ## (abbreviated) state name
}
```

NOTE: For the purpose of this part of the assignment (and for efficiency), your function should NOT call the rankhospital function from the previous section.

The function should check the validity of its arguments. If an invalid outcome value is passed to rankall, the function should throw an error via the stop function with the exact message “invalid outcome”. The num variable can take values “best”, “worst”, or an integer indicating the ranking (smaller numbers are better). If the number given by num is larger than the number of hospitals in that state, then the function should return NA.

#### Submission

``` r
hosp_sort <- function(state,outcome){
       #setting NA to be 0
       rates[rates == "Not Available"] <- 0
       index <- c(grep("^Hospital.*Death*", names(rates)))
       mortality_rates <- rates[,c(2,7,index)]
       names(mortality_rates)[3:5] <- c("heart attack", "heart failure", "pneumonia")
       
       #Making rates numeric
       mortality_rates[,3:5] <- apply(mortality_rates[,3:5],2,as.numeric)
       
       mortality_rates[mortality_rates == 0] <- NA
       selected_state <- mortality_rates[mortality_rates$State == state,]
       
       order_selected <- arrange(selected_state, selected_state[,outcome], Hospital.Name, na.last=TRUE)
       order_selected <- order_selected[complete.cases(order_selected[,outcome]),]
       return(order_selected)
        #na.omit(selected_state[order(c(selected_state[,outcome]), na.last = TRUE),])
}

best <- function(state, outcome, st = "a") {
       if (!state %in% unique(rates$State))
              return("Invalid State")
       if (!outcome %in% c("heart attack", "heart failure", "pneumonia"))
              return("Invalid Outcome")
       
       order_selected <- hosp_sort(state,outcome)
       return(order_selected[1,c(1,2)])

}

worst <- function(state, outcome, st = "a") {
       if (!state %in% unique(rates$State))
              return("Invalid State")
       if (!outcome %in% c("heart attack", "heart failure", "pneumonia"))
              return("Invalid Outcome")
       
       order_selected <- hosp_sort(state,outcome)
       
       return(order_selected[nrow(order_selected),c(1,2)])
}

rankhospital <- function(state, outcome, num = "best", st = "a"){
       if (num == "best")
              return(best(state,outcome))
       if (num == "worst")
              return(worst(state,outcome))       
       else {order_selected <- hosp_sort(state,outcome) 
              return(order_selected[num,c(1,2)])}
}

rankall <- function(outcome, num = "best") {
       #print(lapply(unique(rates$State),hosp_sort, outcome))
       results <- unlist(lapply(sort(unique(rates$State)), rankhospall, outcome, num),use.names=FALSE)
       hosp <- results[c(TRUE,FALSE)]
       state <- results[c(FALSE,TRUE)]
       all <- data.frame(hosp,state)
       names(all) <- c("Hospital.Name", "state")
       all
       
}
```

#### Testing

<hr>
See more at <a href="http://www.ryantillis.com"> my website. </a>

<hr>
