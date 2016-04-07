--- 
title       : Discovering googleVis
description : In this chapter you will be introduced to gooleVis for R. 

--- type:VideoExercise lang:r xp:50 skills:1
## Hans Rosling TED 2006

*** =video_link
//player.vimeo.com/video/97249290


--- type:NormalExercise lang:r xp:100 skills:1,4
##  Loading in your data

What you just saw was one of the most famous ted talks on data and statistics. With the drama and urgency of a sportscaster, statistics guru [Hans Rosling](http://www.ted.com/speakers/hans_rosling) debunks myths about the so-called "developing world". But what sets Rosling apart isn't just his apt observations of broad social and economic trends, but the stunning way he presents them. Have you ever seen data presented like this? The data sings, trends come to life, and the big picture snaps into sharp focus.

In this demonstation, you will learn how to DIY with the help of R. Step-by-step you will transform development statistics into moving bubbles and flowing curves that make global trends clear, intuitive and even playful.

First, you will need to load data on a country's evolution of life expectancy, GDP and population over the past years into R. We can get this data by using [DataMarket](https://datamarket.com/), a company that lets you freely search public data and load it into R with the [`rdatamarket`](http://www.rdocumentation.org/packages/rdatamarket/functions/rdatamarket-package) package. For the data you'll be using here, no account is needed. You will be using two R functions in this exercise:

- [`dminit()`](http://www.rdocumentation.org/packages/rdatamarket/functions/dminit) to initialize a DataMarket client with an API key. Access to public datasets does not require such key, so you can provide the value `NULL`.     
- [`dmlist()`](http://www.rdocumentation.org/packages/rdatamarket/functions/dmlist) requires the ID of the dataset you need, and will return a `data.frame` object.  

*** =instructions
- Initialize a DataMarket client using the provided command. 
- The code to load in the yearly life expectancy data for countries is provided. Inspect the data frame `life_expectancy` with `head()` and/or `tail()`.  
- Load in the yearly GDP (ID: 15c9!hd1) data frame for each country yourself. Assign the results to the variable `gdp`. 
- The yearly population for each country is loaded and assigned to the variable `population`. Inspect this data frame with `head()` and/or `tail()`.    

*** =hint
Have a look at the code for `life_expectancy` on how to load `gdp` and `population`. Do not forget, no API key is required. 

*** =pre_exercise_code
```{r,eval=FALSE}
library("rdatamarket")
load(url("http://s3.amazonaws.com/assets.datacamp.com/course/googlevis/googlevis_ex1.RData"))
population <- dmlist("1cfl!r3d")
```

*** =sample_code
```{r}
# Load the `rdatamarket` package
library("rdatamarket")

# Initialize a DataMarket client
dminit(NULL)

# Load in the yearly life expectancy data frame for each country as 'life_expectancy'
life_expectancy <- dmlist("15r2!hrp")

# Inspect `life_expectancy` with head() and/or tail()


# Load in the yearly GDP data frame for each country as 'gdp'


# The variable 'population' has been pre-loaded. Inspect it with head() and/or tail()

```

*** =solution
```{r}
# Load the rdatamarket package
library("rdatamarket")

# Initialize a DataMarket client
dminit(NULL)

# Load in the yearly life_expectancy data frame for each country as 'life_expectancy'
life_expectancy <- dmlist("15r2!hrp")

# Inspect the variable life_expectancy with head() and/or tail()
head(life_expectancy)
tail(life_expectancy)

# Load in the yearly GDP data frame for each country as 'gdp'
gdp <- dmlist("15c9!hd1")

# The variable 'population' has been pre-loaded. Inspect it with head() and/or tail()
head(population)
tail(population)
```


*** =sct
```{r}

# Instruction 1 and 2
test_object("life_expectancy",
            undefined_msg = "Did you assign to `life_expectancy` to the evolution of a country's life expectancy over the past years?",
            incorrect_msg = "Did you change the way to calculate the variable `life_expectancy`?")
test_or(
        test_output_contains("head(life_expectancy)"), 
        test_output_contains("tail(life_expectancy)"),
                             incorrect_msg = "Did you print the start and/or the end of the life expectancies data frame to the console?")
   
# Instruction 3
test_function("dmlist",
              incorrect_msg = "Have a look at how the variable `life_expectancy` is created. Remember to use the key `15c9!hd1`")
msg = "Have a look at how the variable `life_expectancy` is created. You should do something similar for gdp using the code `15c9!hd1`"
test_object("gdp",
            undefined_msg = msg,
            incorrect_msg = msg)

# Instruction 4
test_or(
        test_output_contains("head(population)"), 
        test_output_contains("tail(population)"),
                             incorrect_msg = "Did you print the start and/or the end of the `population` data frame to the console?")

test_error(incorrect_msg = msg)
success_msg("Good job! Now that you've imported the data, continue to the next exercise to start the real work.")
```

--- type:NormalExercise lang:r xp:100 skills:1,4
##  Preparing the data

Now you have the following three different datasets at your disposal: `life_expectancy`, `gdp`, and `population`. As of now, these datasets will always be preloaded in all the exercises' workspace so you can access and use them at any time.  

If you've applied the [`head()`](http://www.rdocumentation.org/packages/multivator/functions/head) and/or [`tail()`](http://www.rdocumentation.org/packages/multivator/functions/head) function to these variables you've probably noticed that: 

- Not all column names are named properly: the string "Value" is used to name the GDP value, the life expectancy value, and the population value. It would be better if you could make these more descriptive and unique for each. (Tip: use `names()` to see the column names of a dataset.) 
- Our data is only complete until 2008. 

These issues should be fixed before you start creating your graph. In addition, if you want to map all three development statistics into one interactive graph (and you should because it is extremely cool), you will have to merge your three data frames into one called `development`. One standard way to do this is as follows:

`development = merge(gdp,life_expectancy, by = c("Country","Year"))`
`development = merge(development,population, by = c("Country","Year"))`

However, in this exercise you will be required to do it with the [`join()`](http://www.rdocumentation.org/packages/adehabitatMA/html/join.html) function from the [`plyr()`](http://www.rdocumentation.org/packages/plyr/functions/plyr) package. You can read more on how to use the `join` function by typing `?join` in your console.      

*** =instructions
- The code to rename the column name "Value" to `"GDP"` for the `gdp` dataset is provided. Now rename the column name "Value" in the other two datasets to respectively `"Population"` and `"Life Expectancy"`.
- Use `plyr()` and its `join()` function to merge our three data frames into one data frame `development`. In this case, you want to join on all common variables: "Country" and "Year". Joining on all common variables means the `by` argument is not required.

*** =hint
Joining our three data frames into one is very easy in this case. If you have 3 datasets to join, `data_one`, `data_two` and `data_three`, on all common variables, you go in steps: `data_total = join(data_one, data_two)` and then `data_total = join(data_total, data_three)`. So every `join()` only needs two arguments here.


*** =pre_exercise_code

```{r,eval=FALSE}
library("rdatamarket")
library("plyr")
load(url("http://s3.amazonaws.com/assets.datacamp.com/course/googlevis/googlevis_ex2.RData"))
```

*** =sample_code
```{r}
# load in the `plyr` package
library("plyr")

# Make the column name "Values" for each dataset more descriptive and unique
names(gdp)[3] <- "GDP"


# Use plyr to join your three data frames into one: development 
development <- join(gdp, ___)
development <- join(___, ___)
```

*** =solution
```{r}
# load in the `plyr` package
library("plyr")

# Make the column name "Values" for each dataset more descriptive and unique
names(gdp)[3] <- "GDP"
names(life_expectancy)[3] <- "Life Expectancy"
names(population)[3] <- "Population"

# Use plyr to join your three data frames into one: `development` 
development <- join(gdp,life_expectancy)
development <- join(development,population)
```


*** =sct
```{r}
test_error()

# Instruction 1
test_data_frame("gdp",
                undefined_msg = "It seems like you changed the code to rename the column values of `gdp`.",
                undefined_cols_msg = "It seems like you changed the code to rename the column values of `gdp`.")
test_data_frame("life_expectancy",
                undefined_msg = "Have a look at how the column values of `gdp` were renamed. Do the same for `life_expectancy`.",
                undefined_cols_msg = "Have a look at how the column values of `gdp` were renamed. Do the same for `life_expectancy`.")
test_data_frame("population",
                undefined_msg = "Have a look at how the column values of `gdp` were renamed. Do the same for `population`.",
                undefined_cols_msg = "Have a look at how the column values of `gdp` were renamed. Do the same for `population`.")

# Instruction 2
test_function("join",
              incorrect_msg = "Looks like you did not use the join() formula to correctly to merge your data frames.")

success_msg("Your data is almost ready! Only one more set of data preparations and you can start to make some sweet visualizations!")
```

--- type:NormalExercise lang:r xp:100 skills:1,4
##  Last data preps
Now that you have merged your data, it would make sense to trim the data set. You can do this in 2 ways:

- Take out data for years you know that have incomplete observations. In this case, the data is only complete up until 2008.
- Trim down the data set to include fewer countries. Your dataframe `development` is currently loaded and contains observations about 226 countries per year. That could be a bit messy to plot on one graph. 

One way to do this would be to make use of the `subset()` function. You can use this function to pull values from your data frame based on sets of conditions. For example, if you want to see only observations from 2005:

`dev_2005 <- subset(development, Year == 2005)`

Then if you want to see only countries that had a gdp of over 30,000 in 2005:

`dev_2005_big <- subset(dev_2005, GDP >= 30000)`

Feel free to type `?subset` in the console to read more on how to use the function.

*** =instructions
- Take a subset from the `development` dataset that does not include the values after 2008. Then output the final few rows of the data frame with `tail()`.
- To make sure the graph is not too busy, you will work with a subset of only a few countries. These countries are stored in the variable `selection`. Take a subset from `development` with only the countries in `selection` and name this dataset `development_motion`. The operator `%in%` will be helpful here.

*** =hint
Remember to use `subset()` to pull only years >= 2008 particular values from `development`. Then, make use of the `%in%` operator to only select values from the Country column that are in `selection`.

*** =pre_exercise_code

```{r,eval=FALSE}
library("rdatamarket")
library("plyr")
load(url("http://s3.amazonaws.com/assets.datacamp.com/course/googlevis/googlevis_ex2.RData"))
names(gdp)[3] <- "GDP"
names(life_expectancy)[3] <- "Life Expectancy"
names(population)[3] <- "Population"
development <- join(gdp,life_expectancy)
development <- join(development,population)
load(url("http://s3.amazonaws.com/assets.datacamp.com/course/googlevis/googlevis_ex3.RData"))
selection <- c("Afghanistan","Australia","Austria","Belgium","Bolivia","Brazil","Cambodia","Azerbaijan", "Chile","China","Denmark","Estonia","Ethiopia","Finland","France","Georgia","Germany","Ghana","Greece","India","Indonesia","Iraq","Italy", "Japan","Lithuania","Luxembourg","Mexico","New Zealand", "Niger", "Norway", "Poland", "Portugal","Rwanda", "Somalia", "South Africa", "Spain", "Sweden", "Switzerland", "Turkey", "Uganda", "Ukraine", "United Kingdom", "United States", "Vietnam")
```

*** =sample_code
```{r}
# Only include data on or before 2008 and inspect the tail of the data frame
development <- subset(___, ___)

# Include only the countries from `selection` in `development_motion`
selection
development_motion <- subset(___, ___)

```

*** =solution
```{r}
# Only include data on or before 2008 and inspect the tail of the data frame
development <- subset(development, Year <= 2008)
tail(development) 

# Include only the countries from `selection` in `development_motion`
selection
development_motion <- subset(development, Country %in% selection)
```

*** =sct
```{r}
# Instruction 1
test_output_contains("tail(development)", 
                     incorrect_msg = "Don't forget to inspect the last few rows of `development`. The output should not contain any observations after 2008 and 5 columns.")
                     
# Instruction 2
test_object("development_motion",
            undefined_msg = "Make sure you have taken a subset from `development` including only the countries in `selection`.",
            incorrect_msg = "You still need to work with a subset from `development`. Use the `subset()` function and the `selection` variable. ")

test_error()
success_msg("Looks like your data is ready to rumble! Time to make Hans Rosling proud.")
```

--- type:NormalExercise lang:r xp:100 skills:1,4
##  googleVis - the prelude

Time to start the magic! In the next exercises, you will be introduced to the [`googleVis`](http://www.rdocumentation.org/packages/googleVis) package. This package provides an interface between R and the [Google Chart Tools](https://developers.google.com/chart/). As with every package, `googleVis` give you access to various functions. Here they will allow you to visualize data with the Google Chart Tools without uploading your data to Google. 

For this exercise, you will need to create your first motion chart. A motion chart is a dynamic chart that will allow you to explore several indicators over time. To create a motion chart with the [`googleVis`](http://www.rdocumentation.org/packages/googleVis) package, you will need to use the [`gvisMotionChart()`](http://www.rdocumentation.org/packages/googleVis) function (what's in a name?). The beauty of this function is in its simplicity to use, and the huge range of tweaks you can do to prettify your graph.  

The `gvisMotionChart()` function in its simplest form takes 3 arguments. The first argument you need to provide is your data frame `development_motion` (which is pre-loaded). Next, you assign the subject to be analyzed to the `idvar` argument, and to the `timevar` argument the column name that contains the time dimension data. Et voila, your first motion chart with `googleVis` is ready! 

Note: the `options=list()` argument is necessary here to fit better in your viewing screen. 

*** =instructions
- Use the `gvisMotionChart()` function to create an interactive motion chart with R. Assign the output to `my_motion_graph`. What column names do you need to provide to the `idvar` and `timevar` if you know that the Hans Rosling moving bubbles represent countries, and their movement is based on the date?
- Use the [`plot()`](http://www.rdocumentation.org/packages/Rssa/functions/plot) function to visualize your first motion graph. 

*** =hint
The `idvar` argument should be equal to "Country" and the `timevar` argument to "Year". 

*** =pre_exercise_code
```{r,eval=FALSE}
library("ggvis")
library("googleVis")
options(gvis.plot.tag = 'chart')
load(url("http://s3.amazonaws.com/assets.datacamp.com/course/googlevis/googlevis_ex5.RData"))
```


*** =sample_code
```{r}
# Create the interactive motion chart with R and `gvisMotionChart`
my_motion_graph = gvisMotionChart(___,
                                  ___,
                                  ___,
                                  options=list(height='automatic', 
                                               width='automatic'))
                                  
# Plot you motion graph with the help of `plot()`

```

*** =solution
```{r}
# Create the interactive motion chart with R and `gvisMotionChart`
my_motion_graph = gvisMotionChart(development_motion,
                                  idvar = "Country",
                                  timevar = "Year",
                                  options=list(height='automatic', 
                                               width='automatic'))

# Plot you motion graph with the help of `plot()`
plot(my_motion_graph)
```

*** =sct
```{r}
# Instruction 1
test_function("gvisMotionChart", c("data", "idvar", "timevar"), eval = FALSE, 
              not_called_msg = "You should use the `gvisMotionChart()` function in this exercise.",
              incorrect_msg = "Make sure the `idvar` argument is set to 'Country' and the `timevar` argument is set to 'Year'.")

# Instruction 2
test_function("plot",
              not_called_msg = "Do not forget to plot your new motion graph!")
#test_output_contains("plot(my_motion_graph)",
#                     incorrect_msg = "Make sure to use `plot()` to display `my_motion_graph`!")

test_error()
success_msg("Bellissimo! Take some time to enjoy the fruits of your labor and play around with the motion chart! In the next exercise you will start to place the icing on the cake.")
```

--- type:NormalExercise lang:r xp:100 skills:1,4
##  googleVis - the interlude

When working with a simple dataset to visualize, a single color and size for each observation is sufficient. But what if you like to know more? For example, how can you see at a glance which dots respresents which countries in your motion chart? Or how these different dots are proportionate to each other? 

To make the motion chart even more understandable you can play with the size and color of each bubble. This way you can present more information into one motion chart. Doing this with googleVis is not that hard, again, you only have to play a little bit with the arguments. 

Let's say you want to make a motion chart that displays the GDP of a country on the x-axis and the life expectancy on the y-axis. Furthermore, you think it would be nice if each country has a unique color, and the size of each bubble represents the size of the population of that country. Doing this via the `gvisMotionChart()` function will require adding 3 additional arguments to our existing function:

- xvar = Here you place the column name of the variable to be plotted on the x-axis
- yvar = Here you place the column name of the variable to be plotted on the y-axis 
- sizevar = Here you provide the column name that will make the bubbles change size. 

Time to set this into action...

*** =instructions
- The code you wrote for `my_motion_graph` in the previous exercise is provided. Add the required additional arguments. 
- Make sure the x-axis of your motion chart represents the GDP of a country, and the y-axis is life expectancy.  
- The bubble size should increase if a country has a larger number of citizens.  
- Do not forget to plot your new motion chart.

*** =hint
You need to set three additional arguments in `gvisMotionChart()`: `xvar`, `yvar`, and `sizevar`. Check out the arguments you have already set.

*** =pre_exercise_code
```{r}
library("googleVis")
options(gvis.plot.tag = 'chart')
load(url("http://s3.amazonaws.com/assets.datacamp.com/course/googlevis/googlevis_ex5.RData"))
```


*** =sample_code
```{r}
# Create the interactive motion chart with R and `gvisMotionChart())`
my_motion_graph = gvisMotionChart(development_motion,
                                  idvar = "Country",
                                  timevar = "Year",
                                  xvar = ___,
                                  ___,
                                  ___,
                                  options=list(height='automatic', 
                                               width='automatic'))

# Plot your new motion graph with the help of `plot()`
plot(my_motion_graph)
```

*** =solution
```{r}
# Create the interactive motion chart with R and `gvisMotionChart())`
my_motion_graph = gvisMotionChart(development_motion,
                                  idvar = "Country",
                                  timevar = "Year",
                                  xvar = "GDP",
                                  yvar = "Life Expectancy",
                                  sizevar = "Population",
                                  options=list(height='automatic', 
                                               width='automatic'))

# Plot your new motion graph with the help of `plot()`
plot(my_motion_graph)
```

*** =sct
```{r}
# Instructions 1
test_function("gvisMotionChart", c("data", "idvar", "timevar", "xvar", "yvar", "sizevar") , eval = FALSE, 
              not_called_msg = "You should use the `gvisMotionChart()` function in this exercise.",
              incorrect_msg = "Make sure the `xvar` argument is set to 'GDP' and the `yvar` argument is set to 'Life Expectancy'.")

# Instruction 1
test_function("plot",
              not_called_msg = "Do not forget to plot your new motion graph!")

test_error()
success_msg("Fantastic! The plot looks great, but you could make one more adjustment to make the plot a little easier to see.")
```

--- type:NormalExercise lang:r xp:100 skills:1,4
##  googleVis - final output

That last plot looked awesome! There is clearly a relationship between a country's GDP per capita and life expectancy at birth. However, it looks like the relationship is non-linear. You should make a transformation to the data to see if you can make the plot easier to read. 


*** =instructions
- Create a new column in `development_motion` that corresponds to the log of the GDP column called "logGDP".
- The code you wrote for `my_motion_graph_log` in the previous exercise is provided. Change the x-axis argument to our newly created logGDP column. 
- Do not forget to plot your new motion chart.

*** =hint
The argument you need to set the argument `xvar` to the log of GDP in the `gvisMotionChart()` function

*** =pre_exercise_code
```{r}
library("rdatamarket")
library("plyr")
library("googleVis")
options(gvis.plot.tag = 'chart')
load(url("http://s3.amazonaws.com/assets.datacamp.com/course/googlevis/googlevis_ex5.RData"))
#load(url("http://s3.amazonaws.com/assets.datacamp.com/course/googlevis/Country_Mapping.RData"))
```

*** =sample_code
```{r}
# Create a new column that corresponds to the log of the GDP column


# Create the interactive motion chart with R and `gvisMotionChart())`
my_motion_graph_log = gvisMotionChart(development_motion,
                                      idvar = "Country",
                                      timevar = "Year",
                                      xvar = ___,
                                      yvar = "Life Expectancy",
                                      sizevar = "Population",
                                      options=list(height='automatic', 
                                                   width='automatic'))

# Plot your new motion graph with the help of `plot()`
```

*** =solution
```{r}
# Create a new column that corresponds to the log of the GDP column
development_motion$logGDP = log(development_motion$GDP)

# Create the interactive motion chart with R and `gvisMotionChart())`
my_motion_graph_log = gvisMotionChart(development_motion,
                                      idvar = "Country",
                                      timevar = "Year",
                                      xvar = "logGDP",
                                      yvar = "Life Expectancy",
                                      sizevar = "Population",
                                      options=list(height='automatic', 
                                                   width='automatic'))

# Plot your new motion graph with the help of `plot()`
plot(my_motion_graph_log)
```

*** =sct
```{r}
# Instruction 1 from previous exercise
msg = "Make sure you have created a new column `logGDP` in `development_motion"
test_object("development_motion",
            undefined_msg = msg,
            incorrect_msg = msg)

# Instructions 2 and 3
test_function("gvisMotionChart", c("data", "idvar", "timevar", "xvar", "yvar", "sizevar") , eval = FALSE, 
              not_called_msg = "You should use the `gvisMotionChart()` function provided in this exercise.",
              incorrect_msg = "Make sure the `xvar` argument is set to the log of the GDP column.")

# Instruction 4
test_function("plot",
              not_called_msg = "Do not forget to plot your new motion graph!")

test_error()
success_msg("Isn't that beautiful! Probably the best stats you've ever seen. Again, play around with the graph and get a good understanding of what it represents. Then head to the final question...")
```

--- type:MultipleChoiceExercise lang:r xp:50 skills:1,4
##  googleVis - the recessional

As mentioned in the beginning, the goal of these charts is (not only) to impress your audience, but also to visualize trends and to provide a more clear view on the data and the corresponding insights. 

To test your understanding of the data, you end this demo chapter by solving the following multiple choice question: "What was the GDP of Germany in 1980, and what was the population size at that time?"   

<i> All of the datasets and variables you constructed in the previous exercises are preloaded in your workspace.  Type 'ls()' in the console to see them.</i>

*** =instructions
- 81,653,702 Germans producing a GDP of 12,092
- 78,297,904 Germans producing a GDP of 11,746
- 78,297,904 Germans producing a GDP of 72.7
- 81,653,702 Germans producing a GDP of 30.89

*** =hint
You can still have a look at your interactive motion chart by typing `plot(my_motion_graph)` or `plot(my_motion_graph_log)` into the console. 

*** =pre_exercise_code
```{r}
library("rdatamarket")
library("plyr")
library("googleVis")
options(gvis.plot.tag = 'chart')
load(url("http://s3.amazonaws.com/assets.datacamp.com/course/googlevis/googlevis_ex5.RData"))
development_motion$logGDP = log(development_motion$GDP)
my_motion_graph = gvisMotionChart(development_motion,idvar = "Country",timevar = "Year",xvar = "GDP",yvar = "Life Expectancy",sizevar = "Population", options = list(width = 'automatic', height = 'automatic'))
my_motion_graph_log = gvisMotionChart(development_motion,idvar = "Country",timevar = "Year",xvar = "logGDP",yvar = "Life Expectancy",sizevar = "Population",options=list(height='automatic', width='automatic'))
```

*** =sct
```{r,eval=FALSE}
msg <- "Have another look at your graph. Use the hint if you need extra help."
okmsg <- "Wunderbar! We hope you enjoyed doing this demo on DataCamp. In the future, we plan to do more courses on using interactive visualizations to analyze and present your data via R and googleVis. In the meantime, keep practicing, and maybe you can, sometime, share the stage with Hans Rosling."
test_mc(2, c(msg, okmsg, msg, msg))
success_msg("Congrats on making some seriously awesome graphs! You can learn tons of other ways to visualize data using R [here on our courses page](https://www.datacamp.com/courses?learn=data_visualization) !")
```
