---
title: "Working with NSDUH Data in R"
author: "Collin Miller"
date: "October 25, 2017"
output: md_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE, comment = NA, fig.width=8.4)
```

## Packages Utilized in this Demo 
`ggplot2`
`descr`
`Rcpp`
`DT`
`stringr`
`dplyr`
`tidyr`


## Description of Data

Data for this demo comes from the 2015 National Survey on Drug Use and Health (NSDUH). For more information on NSDUH and these data visit the [Substance Abuse & Mental Health Data Archive](https://www.rstudio.com/wp-content/uploads/2016/03/rmarkdown-cheatsheet-2.0.pdf). All 2015 data were downloaded and subsetted to only include **women** with **select variables** outlined below. The purpose of this demo is to display some of the functionality of R and how to begin working with and inspecting data. 

|Variable Name| Definition
|-------------|-----------
|QUESTID2|Respondent ID
|preg| Pregnancy status
|depndalc| Alcohol Dependence - Past Year
|depndmrj| Marijuana Dependence - Past Year
|depndcoc| Cocaine Dependence - Past Year
|depndher| Heroin Dependence - Past Year
|depndpyhal| Hallucinogen Dependence - Past Year
|depndpyinh| Inhalant Dependence - Past Year
|depndpymth| Methamphetamine Dependence - Past Year
|depndpypnr| Pain Reliever Dependence - Past Year
|depndpytrq| Tranquilizer Dependence - Past Year
|depndpystm| Stimulant Dependence - Past Year
|depndpysed| Sedative Dependence - Past Year
|depndpypsy| Psychotherapeutic Dependence - Past Year
|depndpyill| Illicit Drug Dependence - Past Year
|depndpyiem| Illicit Drug Dependence Other Than Marijuana - Past Year
|abusealc| Alcohol Abuse - Past Year
|abusemrj| Marijuana Abuse - Past Year
|abusecoc| Cocaine Abuse - Past Year
|abuseher| Heroin Abuse - Past Year
|abusepyhal| Hallucinogen Abuse - Past Year
|abusepyinh| Inhalant Abuse - Past Year
|abusepymth| Methamphetamine Abuse - Past Year
|abusepypnr| Pain Reliever Abuse - Past Year
|abusepytrq| Tranquilizer Abuse - Past Year
|abusepystm| Stimulant Abuse - Past Year
|abusepysed| Sedative Abuse - Past Year
|abusepypsy| Psychotherapeutic Abuse - Past Year
|abusepyill| Illicit Drug Abuse - Past Year
|abusepyiem| Illicit Drug Abuse Other Than Marijuana - Past Year
|irmcdchp| Imputation Revised Medicaid/CHIP Status
|NEWRACE2| Race|
|eduhighcat| Education Level
|irwkstat| Imputation Revised Employment Status
|income| Income
|sexident| Sexual Identity
|amdeyr| Adult Major Depressive Episode - Past Year
|ymdeyr| Youth Major Depressive Episode - Past Year
|AGE2| Age Categorical
|CATAG6| Age Categorical - 6 levels

## Load Data

The first thing we want to do is load the R data into the Global Environment. We use the `load()` function to bring in the R file type .rda. 

```{r, warning=FALSE, message=FALSE}
load("nsduh_female_dat_2015.rda")
```

## Examine Structure and Variables 

Once we have the data file in the Global Environment, we want to begin examining the structure. That is, how many observations and variables are in our data? What types of variables are we working with? As is almost always the case in R, there are several ways to begin answering these questions. The `str` and `glimpse` functions allow you to examine your variables and take a quick look at the first few observations. The `dim` function provides the number of rows/observations and columns/variables. 


```{r, warning=FALSE, message=FALSE}
#Structure
str(fem.d, give.att = FALSE)

#Glimpse with dplyr
library(dplyr)
glimpse(fem.d)

#dim - dimensions (rows/observations and columns/variables)
dim(fem.d)
```


## Clean Variables

When we review our variables, it's often the case that we want redefine/recode the variables in which we will be working. For example, in our data set, all variables were brought in as integers which we will want to recode into categorical or factor variables in R. The following code provides a few techniques for accomplishing this task. 


```{r}

fem.d$irsex.f <- factor(fem.d$irsex,
                        labels = c("Female"))

#Loop for recoding abuse and dependence into factors

fact_list <- names(fem.d[4:31])

for (k in fact_list){
  fem.d[k][fem.d[k]== 0] <- "No"
  fem.d[k][fem.d[k]== 1] <- "Yes"
}

#Convert to factors
fem.d[,fact_list] <- lapply(fem.d[,fact_list], factor)

#Race
fem.d$NEWRACE2.f <- factor(fem.d$NEWRACE2,
                           labels = c("White", "Black", "Native Am/AK Native",
                                      "Native HI/Other Pac Isl", "Asian", "Multi-racial",
                                      "Hispanic"))

#Education

fem.d$eduhighcat.f <- factor(fem.d$eduhighcat,
                             labels = c("Less high school", "High school grad", "Some col/Assoc Dg", "College graduate", "12-17 years old"))

#Employment

fem.d$irwrkstat.f <- factor(fem.d$irwrkstat,
                            labels = c("Employed full time", "Employed part time" , "Unemployed", "Other - not in labor force", "12-14 years old"))


#Medicaid status

fem.d$irmcdchp.f <- factor(fem.d$irmcdchp,
                           labels = c("Yes, Medicaid/CHIP", "No, Medicaid/CHIP"))


#Major depressive disorder in past year - Adult


fem.d$amdeyr.f <- factor(fem.d$amdeyr,
                         labels = c("Yes", "No"))

#Major depressive disorder in past year - Youth

fem.d$ymdeyr.f <- factor(fem.d$ymdeyr,
                         labels = c("Yes", "No"))

#Age

fem.d$CATAG6.f <- factor(fem.d$CATAG6,
                         labels = c("12-17", "18-25", "26-34", "35-49", "50-64", "64 or Older"))

#Sexual Identity
fem.d$sexident.f <- cut(fem.d$sexident, 
                    breaks = c(0, 1, 2, 3, Inf),
                    labels = c("Heterosexual", "Lesbian or Gay", "Bisexual", "NA"))
#NA categorical as NA value
fem.d$sexident.f[fem.d$sexident.f == "NA"] <- NA
#Drop levels with 0 observations
fem.d$sexident.f <- droplevels(fem.d$sexident.f)
```

## Visually Inspect Data 

After recoding variables, I like to take a quick look at the data set. R in combination with Markdown and `knitr` have some options for visually inspecting these data. In this case, I've used the `DT` package to produce an interactive table for spot checking data. Obvisouly, when data sets are large, like this one, visually inspecting data is not a good strategy for detecting errors as there are too many observations to manually review. However, I like to just take a quick look to see if things appear as expected. 

```{r, warning=FALSE, message=FALSE}
#install.packages("DT", dependencies = TRUE)
#install.packages("Rcpp")
library(Rcpp)
library(DT)
datatable(fem.d[1:50, ])
```

## Descriptive Statistics

All of the variables we're using in this demonstraction are factor/categorical, so we will be using numbers and percentages for our descriptive statistics. The `descr` package offers a nice function to obtain a frequency table and bar plot of each variable. Below I use the `freq` function to look at a number of demographic variables. Because we're running this function on multiple variables, you might think about writing a loop to save some time. 


## Frequencies
```{r, warning=FALSE, message=FALSE}
#Age distribtution
library(descr)

freq(fem.d$CATAG6.f)
freq(fem.d$NEWRACE2.f)
freq(fem.d$eduhighcat.f)
freq(fem.d$irwrkstat.f)
freq(fem.d$irmcdchp.f)
freq(fem.d$amdeyr.f)
freq(fem.d$ymdeyr.f)
freq(fem.d$sexident.f)

```

## Crosstabs

In addition to looking at each variable independently, we probably also want to look at relationships among variables. With categorical variables, we can do this with a contingecy table or crosstab. Below are a couple of examples of contingency tables displaying relationships between race and alcohol dependence and sexual identity and alcohol dependence.   

```{r, echo=TRUE,results='asis', message=FALSE, warning=FALSE}
library(pander)
x <- xtabs(~NEWRACE2.f+depndalc, data = fem.d)
y <- xtabs(~sexident.f+depndalc, data = fem.d)
pander(x)
pander(y)
```

## Transform for Visualizing Drug Dependence and Drug Abuse Variables

Although looking at frequencies and crosstabs can be helpful in understanding the data set we're working with, often visualizing data is an easier and quicker way to get a good picture of these data. However, data sometimes needs to be transformed before we can build helpful figures. For example, a bar graph depicting the frequency of dependence by substance is a quick way to understand the distribution. However, before we can do that, we need to get our data into a structure that will allow us to build this kind of plot. To do this, we will use the `tidyr` and `ggplot2` packages. Below are the steps used to transform these data.

1. Select the variables we will use to make the plot (including and identifier)
2. Turn a range of variables into one variable with unique key value pairs using `gather`
3. Create a new varible called `sub` that will be the abbreviated substance label getting rid of the first 5 characters of the variable "substance" using `substring`
4. Include only respondents with a positive ("Yes") response
5. Group the data by the new `sub` variable 
6. Total the number of observations by the group specified above 
7. Sort the data from largest to smallest (descendingly)


### Dependence

```{r, warning=FALSE, message=FALSE}
library(tidyr)
library(ggplot2)

depnd.d <- fem.d %>%
  select(QUESTID2, starts_with("depnd")) %>%
  gather("substance","value", 2:15) %>%
  mutate(sub = substring(substance, 6)) %>%
  filter(value == "Yes") %>%
  group_by(sub) %>%
  tally() %>%
  arrange(desc(n))
  

depnd.fig <- ggplot(data = depnd.d, aes(x = reorder(sub, -n), y=n)) +
  geom_bar(stat="identity") +
  labs(
    title = "Frequency of Dependence by Substance for Females",
    subtitle = "NSDUH 2015",
    x = "Substance",
    y = "Number"
  )

depnd.fig
```


### Abuse

```{r, warning=FALSE, message=FALSE}
abuse.d <- fem.d %>%
  select(QUESTID2, starts_with("abuse")) %>%
  gather("substance","value", 2:15) %>%
  mutate(sub = substring(substance, 6)) %>%
  filter(value == "Yes") %>%
  group_by(sub) %>%
  tally() %>%
  arrange(desc(n))
  

abuse.fig <- ggplot(data = abuse.d, aes(x = reorder(sub, -n), y=n)) +
  geom_bar(stat="identity", fill="steelblue") +
  labs (
    title = "Frequency of Abuse by Substance for Females",
    subtitle = "NSDUH 2015",
    x = "Substance",
    y = "Number"
  )

abuse.fig
```