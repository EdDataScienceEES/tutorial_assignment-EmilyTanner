---
title: "Understanding and Implimenting R Overlay and Multilayer Visualization "
author: "Emily Tanner"
date: "29/11/2021"
output: 
  html_document: 
    toc: yes
    fig_caption: yes
    number_sections: yes
---

The wonderful thing about R is there are almost always multiple ways to achieve a desired result. This goes for many things, including data manipulation, synthesis, analysis and visualization and is largely due to the many packages available within R. When learning to code with R, sometimes choosing *HOW* you will get there can be overwhelming, given the number of options available to you. And to add even more complexity, not all of the options are equal, or suitable (depending on your data and the task at hand). The following tutorial covers 3 different approaches to overlaying plots (i.e. adding additional layers of data). It is aimed at beginner R users with some basic experience of how to interact with the interface. The goal is that you will better understand the parallels and differences between overlaying data in order to pick the most suitable *HOW* for your data and visualizations.

Tutorial sections:

 1. how we can visualize data using layered functions of ggplot2
 2. how we can split our plotting device to visualize relationships between variables 
 3. how we can overlay histograms with curve, lines and rug functions to visualize data

Key learning points:

1. the layered nature of R visualizations: how we can build upon on a preliminary plot with layers

2. how different packages and functions i.e. `par` & `facet_wrap` can create similar results

3. thinking critically about which option most effectively represents data and communicates your message


# Visualizing multiple layers of data within ggplot2
## Getting started
First thing's first, open `RStudio`, create a new script by clicking on `File/ New File/ R Script`. You can use the code below to start your script, making sure to fill in all your personal details so your script is well organized for future reference


```{r message=FALSE, warning=FALSE, include=TRUE}
# Purpose of the script
# Name:
# Date:
# Email:

# Set your working directory if not already set up to GitHub version control
# setwd("~/Downloads/overlay_tutorial")
```

If you haven't yet, you can set up GitHub version control [here](https://ourcodingclub.github.io/tutorials/git/).

```{r message=FALSE, warning=FALSE, include=TRUE}
# Load libraries ---- 
## if you haven't installed them before, run the code install.packages("package_name") and THEN call them to your library
library(tidyverse)  # used for multilayer plots, ggplot 2 and data manipulation
library(datasets)  # used to for NYC airquality data 

# Load data ----
View(airquality)  # let's take a look at the data - notice anything problematic?
```

Data: we will be using the `airquality` dataset provided by the R dataset library. The dataset provides daily air quality measurements in New York, from May to September 1973. More information on the data can be found by searching "airquality" within the help tab of R (see image below).

Data Structure:

The data has 6 variables: 

1.  Ozone	numeric	Ozone (ppb)
2. 	Solar.R	numeric	Solar R (lang)
3. 	Wind	numeric	Wind (mph)
4.	Temp	numeric	Temperature (degrees F)
5.	Month	numeric	Month (1--12)
6. 	Day	numeric	Day of month (1--31)

The built in datasets package of R is  a very handy tool for experimenting with R and examining an interesting collection of data. You can preview all of the datasets by entering library `(help = "datasets")` to your console.

## Data manipulation

```{r message=FALSE, warning=FALSE}
# Data manipulation ----
airquality1 <- as_tibble(airquality %>%  # creating a local copy of our data via a tibble object for good practice
  dplyr::mutate(Date = as.Date(paste("1973",Month,Day,sep="/"))))  # let's transform the current day / month structure into useful year-month-day 
## this will facilitate plotting because R can now recognize the continuity of our data over time

summary(airquality1) # let's take another look long data - is it how we want it? 
## Hint: Anything at the bottom of the Ozone and Solar.R sections that might cause trouble later?
```

```{r, echo = TRUE, results = 'hide', warning = FALSE}
airquality1 <- airquality1 %>%  # overwriting the airquality1 object via a pipe
  na.omit(airquality1)  # let's get rid of "NA" values in the dataset       
summary(airquality1)  # Check again - NA's are gone! 
str(airquality1)  # Let's see the structure of the data - what kind of variables is R assigning for our data? 

```


## GGplot! Let's break it down by layer.

GGplot2 is an incredibly useful package for creating elegant visuals. There are a plethora of tutorials that go into depth on beautiful visualizations and ggplot2 tools. This tutorial won't be going into depth with ggplot2 - the objective with using it is to highlight the parallels and differences between ggplot2 functions and other ways of overlaying data.

However, I highly recommend you explore the ggplot2 possibilities in your own time: 
[Coding Club data vis tutorial](https://ourcodingclub.github.io/tutorials/data-vis-2/)
[Coding Club data vis tutorial 2](https://ourcodingclub.github.io/tutorials/datavis/)
[Coding Club visualization beautification tutorial](https://ourcodingclub.github.io/tutorials/dataviz-beautification-synthesis/)

```{r}
# 1. multilayer plots in ggplot package ----

(prelim_plot <- ggplot(airquality1, aes(x = Date, y = Ozone, colour = Month)) +
   geom_point(colour = "#8B5A00") +  # calls for a scatter plot
     xlab("Date") +
     ylab("Ozone (ppb)") +
     ggtitle("Most Basic GGplot: New York Air Quality (1973)"))
```

...yikes. What do you think of this plot? Does it tell you much? Why don't we add some Solar Radation Data? 

## Using Colour to add Additional layer
```{r}
(ggplot(airquality1, aes(x = Date, y = Ozone, colour=Solar.R)) + 
  geom_point() +  # calls for a scatter plot
  xlab("Date") +
  ylab("Ozone (ppb)") +
  ggtitle("Less Basic GGplot: New York Air Quality (1973): Ozone and Solar Radiation"))
```

...Now our figure tells us something about the relationship between Solar Radiation and Ozone in respect to air quality in New York - cool! By just changing that colour parameter, our figure now communicates another element of information.

## Using `geom_` functions to add additional layers

`ggplot2` hosts a number of `geom_` functions which return a layer when used. You can find a list of the functions [here](https://raw.githubusercontent.com/rstudio/cheatsheets/main/data-visualization.pdf). Let's take a look at a few in the application of the New York Air Quality data..

```{r message=FALSE, warning=FALSE}
(ggplot(airquality1, aes(x = Date, y = Ozone, colour=Solar.R)) +
  geom_point() +  # calls for a scatter plot
  geom_smooth() +  # creates a smoothed conditional mean
  xlab("Date") +
  ylab("Ozone (ppb)") +
  ggtitle("GGplot: New York Air Quality Trends (1973): Ozone and Solar Radiation"))  
ggsave("prelim3_plot_tut.png", width = 6, height = 5)  # let's save our plot 

```


## Data manipulation: wide -> long transformation

I won't go into detail about wide -> long transformation (you can find more information [here](https://ourcodingclub.github.io/tutorials/data-manip-intro/) but ggplot2 tends to favour long data formats. We will use a long version of our dataset for the rest of the ggplot2 section.

```{r, echo = TRUE, results = 'hide', warning = FALSE}
# long transformation
air_long <- airquality1 %>%
  select(-Month, -Day) %>%              # Removing Month and Day columns
  gather(variable, value, -Date)
str(air_long)  # Let's see the structure of the data
View(air_long)  # you can use this to view the entirety of the data, opening it in a new tab

```

```{r}
ggplot(air_long, 
       aes(x = Date, y = value, 
           linetype = variable,
           color = variable, 
           group = variable)) +
      geom_point()  

ggplot(air_long,
       aes(x = Date, y = value, 
             colour=variable)) +
      geom_point() + 
      geom_line() +  # adding in lines to connect data points
      ggtitle("New York Air Quality Parameters")  # let's add in a title


```

..how does it look? is it an improvement from the our original plot of Ozone with solar radiation set as our colour?

# Thinking critically...what makes a good visualization?

Overlaying plots is awesome - your plot can go from the most basic visualization to something that effectively and succinctly communicates a relationship or a message to the reader. HOWEVER, too much of a good thing can be bad.. we have to think about how TOO many layers can complicate readers' understanding and detract from the key message. 

Often times, _less is more_. It is also important to think of the TYPE of visualization that is most effective in transmitting information. I found this infographic from the coding club data visualization tutorial quite helpful.

![](images/which_plot.png)

To access the full tutorial visit:
[data vis tutorial](https://ourcodingclub.github.io/tutorials/datavis/)
For more information on types of r plots:
[r graph gallery](https://www.r-graph-gallery.com/)


In this case, I am interested in visualizing the four main variables involved in the study to gain a basic understanding of how I might go about further analyzing this data. However, I don't want to layer them on top of each other (like the plot I have created above), but rather, I would like to see them side-by-side, for a more effective visualization. I can use the `facet wrap` or `par` function to do this - splitting the plot space is a great way to visualize multiple variables without overcrowding a single graphic.

# Splitting plot space 

## `facet wrap` function

```{r echo=TRUE, message=FALSE, warning=FALSE}
# Visualizing by variable
ggplot(air_long, 
       aes(x = Date, y = value)) +
  geom_point(size=1) +
  geom_smooth(se=F, colour="grey") +  # adding in a grey smoothed conditional mean, with no standard error confidence interval
  facet_wrap("variable") +  # facet_wrap based on variable
  ggtitle("Visualizing by Variable: New York Air Quality Data")

ggplot(air_long, 
       aes(x = Date, y = value, 
           linetype = variable,
           color = variable,  # adding in color by variable
           group = variable)) +
  geom_point() +  geom_smooth(se=F, colour="black") +   
  facet_wrap("variable") +
  ggtitle("Visualizing by Variable: New York Air Quality Data")


```

Now we have made several plots using both wide and long forms of the same data. Your application of these techniques will largely depend on your research question. Have you picked up any formatting that you would like to apply to future projects?

Let's take a look at an alternative approach to visualizing relationships between variables in a data set (without using ggplot)...

## `par` function to split plotting

```{r message=FALSE, warning=FALSE}

# par function is used to set graphical parameters for the plotting device 
par(mfrow=c(2,2))  # listing rows first and columns second --> e.g. 2 rows 1 column 
plot(airquality1$Date, airquality1$Ozone, type="l", xlab="Date",
     ylab="Ozone (ppb)")
plot(airquality1$Date, airquality1$Solar.R, type="l", xlab="Date",
     ylab="Solar Radiation (lang)" )
plot(airquality1$Date, airquality1$Temp, type="l", xlab="Date",
     ylab="Temperature (degrees F)" )
plot(airquality1$Date, airquality1$Wind, type="l", xlab="Date",
     ylab="Wind (mph)" )

dev.off()   # turning the device operator off, this an important step because otherwise our plotting device will continue plotting with parameters set above

```

     Notice in the plot function, I specify type "l" above. This stands for line and follows the syntax for specifying plot type as detailed below:

    "p" - points
    "l" - lines
    "b" - both points and lines
    "c" - empty points joined by lines
    "o" - overplotted points and lines
    "s" and "S" - stair steps
    "h" - histogram-like vertical lines
    "n" - does not produce any points or lines
    (Iquebal 2012)

Do you see how `par` and `facet_wrap` can be used to produced similar results? Do you see any benefits to using one over the other? Can you imagine how they might be used in different applications?


Let's take a look at how we can visualize distributions
# Overlaying base r plots with statistics

Like I mentioned before, there are so many ways to perform very similar functions within R. Let's take a look at some layers we can add on to base R graphing functions (again: not an exhaustive list)

### Make a base r histogram

```{r, echo = TRUE, results = 'hide', warning = FALSE}

hist(airquality1$Solar.R)  # creating a basic histogram of Solar Radiation in our airquality data

```

### Adding more information to our basic histogram

```{r message=FALSE, warning=FALSE}
hist(airquality1$Solar.R,
     breaks = 12,  # creating more breaks shows more detail (notice a greater number of bars) 
     freq   = FALSE,  # We can change our hist to a density plot by changing axis to show density, not frequency
     col    = "thistle4",  # specifying colour for histogram
     main   = ("Solar Radiation Density Plot"),
     xlab   = "Solar Radiation (lang)")
```

## Add a normal distribution layer
We can overlay a normal distribution curve to visualize how our data deviates from normality.

      `curve(dnorm(x, mean = mean(airquality1$Solar.R), sd = sd(airquality1$Solar.R)),`
      `col = "green",  # Color of curve`
      `lwd = 2,           # Line width of 2 pixels`
      `add = TRUE)        # Superimpose on previous graph`

```{r, echo = FALSE, results = 'hide', warning = FALSE}
hist(airquality1$Solar.R,
     breaks = 12,  # creating more breaks shows more detail (notice a greater number of bars) 
     freq   = FALSE,  # We can change our hist to a density plot by changing axis to show density, not frequency
     col    = "thistle4",  # specifying colour for histogram
     main   = ("Solar Radiation Density Plot"),
     xlab   = "Solar Radiation (lang)")
curve(dnorm(x, mean = mean(airquality1$Solar.R), sd = sd(airquality1$Solar.R)),
      col = "green",     # Colour of curve
      lwd = 2,           # Line width of 2 pixels
      add = TRUE)        # Superimpose on previous graph
```


## Add a kernel density estimator layer
Further, we can add a kernel density estimation (KDE). A KDE is a non-parametric estimate of the probability density function of a random variable. Application of the KDE provides an additional layer of information that allows us to better analyze the probability (Węglarczyk 2018).

You can find out more about kernel density estimation [here](https://cran.r-project.org/web/packages/kdensity/vignettes/tutorial.html)

      `lines(density(airquality1$Solar.R), col = "blue", lwd = 2)`

```{r, echo = FALSE, results = 'hide', warning = FALSE}
hist(airquality1$Solar.R,
     breaks = 12,  # creating more breaks shows more detail (notice a greater number of bars) 
     freq   = FALSE,  # We can change our hist to a density plot by changing axis to show density, not frequency
     col    = "thistle4",  # specifying colour for histogram
     main   = paste("Solar Radiation Density Plot"),
     xlab   = "Solar Radiation")
curve(dnorm(x, mean = mean(airquality1$Solar.R), sd = sd(airquality1$Solar.R)),
      col = "green",  # Color of curve
      lwd = 2,           # Line width of 2 pixels
      add = TRUE)        # Superimpose on previous graph
lines(density(airquality1$Solar.R), col = "blue", lwd = 2)

```

## Add a rug plot layer
We can also add a 'rug plot' which is a distribution plot that goes along the border of a plot (could be on the top, bottom or sides). The rug plot gives us more information on the distribution of values (Fox, 2003).
      `rug(airquality1$Solar.R, lwd = 2, col = "gray")`

```{r, echo = FALSE, results = 'hide', warning = FALSE}
hist(airquality1$Solar.R,
     breaks = 12,   # creating more breaks shows more detail (notice a greater number of bars) 
     freq   = FALSE,       # We can change our hist to a density plot by changing axis to show density, not frequency
     col    = "thistle4",   # specifying colour for histogram
     main   = "Solar Radiation Density Plot",
     xlab   = "Solar Radiation(lang)")
curve(dnorm(x, mean = mean(airquality1$Solar.R), sd = sd(airquality1$Solar.R)),
      col = "green",  # Color of curve
      lwd = 2,           # Line width of 2 pixels
      add = TRUE)        # Superimpose on previous graph
lines(density(airquality1$Solar.R), col = "blue", lwd = 2)
rug(airquality1$Solar.R, lwd = 2, col = "gray")

```


# The end

I hope this tutorial has been helpful in breaking down the 'layered' nature of R visualization. The tutorial aims to cover some basic functions and options when it comes to overlaying plots, but by no means is exhaustive. It is hoped that this tutorial has demonstrated how many ways there are to approach visualization within R. You can use different packages and commands to get very similar results. You may have also noticed that some packages really streamline processes and enable you to do more, depending on your data and desired outcome. I wish you the best of luck on your coding journey!


# References

* Fox, J., 2003. Effect displays in R for generalised linear models. Journal of statistical software, 8(15), pp.1-27.

* Iquebal, M.A., GRAPHS AND CHARTS IN R.

* Węglarczyk, S., 2018. Kernel density estimation and its application. In ITM Web of Conferences (Vol. 23). EDP Sciences.


