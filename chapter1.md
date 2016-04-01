--- 
title       : Discovering googleVis
description : In this chapter you will be introduced to gooleVis for R. 

--- type:VideoExercise lang:r xp:50 skills:1
## Hans Rosling TED 2006

*** =video_link
//player.vimeo.com/video/97249290


--- type:NormalExercise lang:r xp:100 skills:1,4
##  Loading in your data

What you just saw, was one of the most famous ted talks on data and statistics. With the drama and urgency of a sportscaster, statistics guru [Hans Rosling](http://www.ted.com/speakers/hans_rosling) debunks myths about the so-called "developing world". But what sets Rosling apart isn't just his apt observations of broad social and economic trends, but the stunning way he presents them. Have you ever seen data presented like this? The data sings, trends come to life, and the big picture snaps into sharp focus.

In this demo chapter, you will learn how to DIY with the help of R. Step-by-step you will transform development statistics into moving bubbles and flowing curves that make global trends clear, intuitive and even playful.

First, you will need to load data on a country's evolution of life expectancy, GDP and population over the past years into R. We can get this data by using [DataMarket](https://datamarket.com/), a company that lets you freely search public data and load it into R with the [`rdatamarket`](http://www.rdocumentation.org/packages/rdatamarket/functions/rdatamarket-package) package. For the data you'll be using here, no account is needed. You will be using two R functions in this exercise:
- [`dminit()`](http://www.rdocumentation.org/packages/rdatamarket/functions/dminit) to initialize a DataMarket client with an API key. Access to public datasets does not require such key, so you can provide the value `NULL`.     
- [`dlist()`](http://www.rdocumentation.org/packages/rdatamarket/functions/dmlist) requires the ID of the dataset you need, and will return a `data.frame` object.  

*** =instructions
- Initialize a DataMarket client using the provided command. 
- The code to load in the yearly life expectancy data for countries is provided. Inspect the data frame `life_expectancy` with `head()` and/or `tail`.  
- Load in the yearly GDP (ID: 15c9!hd1) data frame for each country yourself. Assign the results to the variable `gdp`. 
- The yearly population for each country is loaded and assigned to the variable `population`. Inspect this data frame with `head()` and/or `tail`.    

*** =hint
Have a look at the code for `life_expectancy` on how to load `gdp` and `population`. Do not forget, no API key is required. 

*** =pre_exercise_code
```{r,eval=FALSE}
library("rdatamarket")
load(url("http://s3.amazonaws.com/assets.datacamp.com/course/googlevis/googlevis_ex1.RData"))

# correct_life_expectancy = dmlist("15r2!hrp")
# correct_gdp = dmlist("15c9!hd1")
# subset van population = dmlist("1cfl!r3d")
```

*** =sample_code
```{r}
# Load the `rdatamarket` package
library("rdatamarket")

# Initialize a DataMarket client
dminit(NULL)

# Inspect the variable `life_expectancy`
life_expectancy <- dmlist("15r2!hrp")

# Load in the yearly GDP data frame for each country
gdp <-

# Inspect the variable `population`.
population

```

*** =solution
```{r}
# Load the rdatamarket package
library("rdatamarket")

# Initialize a DataMarket client
dminit(NULL)

# Inspect the variable life_expectancy
life_expectancy <- dmlist("15r2!hrp")
head(life_expectancy)
tail(life_expectancy)

# Load in the yearly GDP data frame for each country
gdp <- dmlist("15c9!hd1")

# Inspect the variable population.
head(population)
tail(population)
```


*** =sct
```{r}
if (!exists("life_expectancy")) {
  DM.result = list(FALSE,"Did you assign to `life_expectancy` the evolution of a country's life expectancy over the past years?")
} else if (! exists("gdp")) {
  DM.result = list(FALSE,"Did you assign to `gdp` the evolution of a country's GDP over the past years? Use `dmlist()` to do this.")   
} else if (! (student_typed("dminit(NULL)"))) {
  DM.result = list(FALSE,"Make sure to initialize the DataMarket client with the help of `dminit()`. Remember, API key is not required.")
} else if (! identical(life_expectancy, correct_life_expectancy)){
  DM.result = list(FALSE, "Did you change the way to calculate the variable `life_expectancy`?")
} else if ( ! identical(gdp, correct_gdp) ){
  DM.result = list(FALSE, "Hava a look at how the variable `life_expectancy` is created. You need to do something similar for gdp.")
} else if ((! output_contains("head(life_expectancy)")) && (! output_contains("tail(life_expectancy)"))) {
  DM.result = list(FALSE,"Did you print the start and/or the end of the life expectancies to the console?")
} else if ((! output_contains("head(population)")) && (! output_contains("tail(population)"))) {
  DM.result = list(FALSE,"Did you print the start and/or the end of the country's population to the console?")
} else {
  DM.result = list(TRUE,"Good job! Now that you've imported the data, continue to the next exercise to start the real work.")
}
```

--- type:NormalExercise lang:r xp:100 skills:1,4
##  Preparing the data

Now you have the following three different datasets at your disposal: `life_expectancy`, `gdp`, and `population`. As of now, these datasets will always be preloaded in all the exercises' workspace so you can access and use them at anytime.  

If you've applied the [`head()`](http://www.rdocumentation.org/packages/multivator/functions/head) and/or [`tail()`](http://www.rdocumentation.org/packages/multivator/functions/head) function to these variables (if not, you can still do it via the console), you've probably noticed that:  
- Not all column names are named properly: the string "Value" is used to name the GDP value, the life expectancy value, and the population value. It would be better if you could make these more descriptive and unique for each. (Tip: use `names()` to see the column names of a dataset.) 
- Our data is only complete until 2008. 

This is something that should be fixed before you can start creating your graph. In addition, if you want to map all three development statistics into one interactive graph (and since this is extremely cool you definitely want to do this), it is necessary to merge our three data frames into one: `development`. One standard way to do this is as follows:
`development = merge(gdp,life_expectancy, by = c("Country","Year"))`
`development = merge(development,population, by = c("Country","Year"))`

However, in this exercise you will be required to do it with the [`join()`](http://www.rdocumentation.org/packages/adehabitatMA/html/join.html) function from the [`plyr()`](http://www.rdocumentation.org/packages/plyr/functions/plyr) package. You can read more on how to use the `join` function by typing `?join` in your console.      

*** =instructions
- The code to rename the column name "Value" to "GDP" for the `gdp` dataset is provided. Now rename the column name "Value" in the other two datasets to respectively "Population" and "Life Expectancy".
- Use `plyr()` and its `join()` function to merge our three data frames into one data frame `development`. In this case, you want to join on all common variables: "Country" and "Year". Joining on all common variables means the `by` argument is not required.   
- Take a subset from the `development` dataset that does not include the values after 2008. Name this new dataset `development_final`

*** =hint
Joining our three data frames into one is very easy in this case. If you have 3 datasets to join, `data_one`, `data_two` and `data_three`, on all common variables, you go in steps: `data_total = join(data_one,data_two)` and then `data_total = join(data_total,data_three)`. So every `join()` only needs two arguments here.    

We only need to provide 2 arguments to the `join()` function: 


*** =pre_exercise_code

```{r,eval=FALSE}
library("rdatamarket")
library("plyr")
load(url("http://s3.amazonaws.com/assets.datacamp.com/course/googlevis/googlevis_ex2.RData"))
# life_expectancy = dmlist("15r2!hrp")
# gdp = dmlist("15c9!hd1")
# population = dmlist("1cfl!r3d")
# names(gdp)[3] = "GDP"
# names(life_expectancy)[3] = "Life Expectancy"
# names(population)[3] = "Population"
# correct_development = join(gdp,life_expectancy)
# correct_development= join(correct_development,population)
```

*** =sample_code
```{r}
# load in the `plyr` package
library("plyr")

# Make the column name "Values" for each dataset more descriptive and unique
names(gdp)[3] <- "GDP"


# Use plyr to join your three data frames into one: development 
development <- join(___, ___)
development <- join(___, ___)

#Make sure no data beyond 2008 is included
development_final = 
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

#Make sure no data beyond 2008 is included
development_final <- development[development$Year <= 2008,]
```


*** =sct
```{r}
if (! (names(gdp)[3] == "GDP")){
  DM.result = list(FALSE,"Seems like you changed the code to rename the column values of `gdp`.")
} else if (! (names(life_expectancy)[3] == "Life Expectancy")){
  DM.result = list(FALSE,"Have a look at how the column values of `gdp` were renamed. Do the same for `life_expectancy`.")
} else if (! (names(population)[3] == "Population")){
  DM.result = list(FALSE,"Have a look at how the column values of `gdp` were renamed. Do the same for `population`.")
} else if(! "join" %in% called_functions()) {
  DM.result = list(FALSE,"Looks like you did not use the join() formula to merge your data frames.")
} else if (! isTRUE(try(all.equal(development,correct_development)))){
    DM.result = list(FALSE,"Seems like you made a mistake when merging the data frames. Have a look at the hint if you need any help.") 
} else if (! exists("development_final")) {
  DM.result = list(FALSE,"Did you assign `development_final`?")   
} else if (! isTRUE(try(all.equal(development_final,development[development$Year <= 2008,])))){
    DM.result = list(FALSE,"Do not forget to take a subset from `development`, excluding all data beyond 2008. Type `?subset` in the console for more help.") 
} else {
  DM.result = list(TRUE,"Looks like your data is ready to rumble! Time to make Hans Rosling proud.")
}
```

--- type:NormalExercise lang:r xp:100 skills:1,4
##  googleVis - the prelude

Time to start the magic! In the next exercises you will be introduced to the [`googleVis`](http://www.rdocumentation.org/packages/googleVis) package. This package provides an interface between R and the [Google Chart Tools](https://developers.google.com/chart/). As every package it contains functions, and here they will allow you to visualize data with the Google Chart Tools without uploading their data to Google. 

For this demo chapter you will need to create your first motion chart. A motion chart is a dynamic chart that will allow you to explore several indicators over time. To create a motion chart with the [`googleVis`](http://www.rdocumentation.org/packages/googleVis) package, you will need to use the [`gvisMotionChart()`](http://www.rdocumentation.org/packages/googleVis) function (what's in a name?). The beauty of this function is in its simplicity to use, and the huge range of tweaks you can do to prettify your graph.  

The `gvisMotionChart()` function in its simplest form takes 3 arguments. The first argument you need to provide is your data frame `development_final`. Next, you assign the subject to be analyzed to the `idvar` argument, and to the `timevar` argument the column name that contains the time dimension data. Et voila, your first motion chart with `googleVis` is ready!  

*** =instructions
- To make sure the graph is not too busy, you will work with a subset of only a few countries. These countries are stored in the variable `selection`. Take a subset from `dataframe_final` with only the countries from `selection` and name this dataset `development_motion`  
- Use the `gvisMotionChart()` function to create an interactive motion chart with R. Assign the output to `my_motion_graph`.
- What are the column names you need to provide to the `idvar` and `timevar` if you know that the Hans Rosling moving bubbles represent countries, and their movement is based on the date?
- Use the [`plot()`](http://www.rdocumentation.org/packages/Rssa/functions/plot) function to visualize your first motion graph. 

*** =hint
The `idvar` argument should be equal to "Country" and the `timevar` argument to "Year". Your function will now look something like `gvisMotionChart(data,idvar = "Country",timevar = "Year"). 

*** =pre_exercise_code
```{r}
library("rdatamarket")
library("plyr")
library("googleVis")
options(gvis.plot.tag = 'chart')
load(url("http://s3.amazonaws.com/assets.datacamp.com/course/googlevis/googlevis_ex3.RData"))
#life_expectancy = dmlist("15r2!hrp")
#gdp = dmlist("15c9!hd1")
#population = dmlist("1cfl!r3d")
#names(gdp)[3] = "GDP"
#names(life_expectancy)[3] = "Life Expectancy"
#names(population)[3] = "Population"
#development = join(gdp,life_expectancy)
#development= join(development,population)
#development_final = development[development$Year <= 2008,]
#development_final$Country <-as.character(development_final$Country)
#selection <- c("Afghanistan","Australia","Austria","Belgium","Bolivia","Brazil","Cambodia","Azerbaijan", "Chile","China","Denmark","Estonia","Ethiopia","Finland","France" ,"Georgia","Germany","Ghana","Greece","India","Indonesia","Iraq","Italy","Japan","Lithuania","Luxembourg","Mexico","New Zealand", "Niger", "Norway", "Poland", "Portugal","Rwanda", "Somalia", "South Africa", "Spain", "Sweden", "Switzerland", "Turkey","Uganda", "Ukraine", "United Kingdom", "United States", "Vietnam")
```


*** =sample_code
```{r}
# Include only the countries from `selection` in the interactive motion chart
selection
development_motion = 
  
# Create the interactive motion chart with R and `gvisMotionChart`
my_motion_graph = gvisMotionChart(___,___,___)

# Plot you motion graph with the help of `plot()`

```

*** =solution
```{r}
# Include only the countries from `selection` in the interactive motion chart
selection
development_motion = subset(development_final, Country %in% selection)
  
# Create the interactive motion chart with R and `gvisMotionChart)`
my_motion_graph = gvisMotionChart(development_motion,
                idvar = "Country",
                timevar = "Year"
)

# Plot you motion graph with the help of `plot()`
plot(my_motion_graph)
```

*** =sct
```{r}
gvisMotionChart_args = c("data","idvar","timevar")
student_gvisMotionChart_args =try(names(get_arguments("gvisMotionChart")[[1]]))
standard_feedback_message = "Looks like you forgot to set in `gvisMotionChart()` the arguments"
gvisMotionChart_values = c("development_motion","Country","Year")
student_gvisMotionChart_values = try(get_arguments("gvisMotionChart")[[1]]) 
standard_feedback_message2 = "Looks like you set the wrong argument values in `gvisMotionChart()`:"

if (! exists("development_motion")) {
  DM.result = list(FALSE,"Make sure you have taken a subset from `development_final` including only the countries in `selection`.")  
} else if (! identical(development_motion, subset(development_final, Country %in% selection) ) ){
  DM.result = list(FALSE, "Take a subset from `development_final` with the help of the `subset()` function. ")
} else if(! "gvisMotionChart" %in% called_functions()) {
  DM.result = list(FALSE,"You should use the `gvisMotionChart()` function in this exercise.")
} else if (!try(all(gvisMotionChart_args %in% student_gvisMotionChart_args))) { 
  missing_arguments =  gvisMotionChart_args[!(gvisMotionChart_args %in% student_gvisMotionChart_args)]
  DM.result = list(FALSE,paste(c(standard_feedback_message,missing_arguments),collapse=" "))
} else if (!try(all(gvisMotionChart_values %in% student_gvisMotionChart_values))) { 
  missing_values = student_gvisMotionChart_values[! (student_gvisMotionChart_values %in% gvisMotionChart_values)]
  DM.result = list(FALSE,paste(c(standard_feedback_message2,missing_values),collapse=" "))
} else if (! exists("my_motion_graph")) {
  DM.result = list(FALSE,"Mmmm. There is something with your `gvisMotionChart()` function. Check the hint if you need help.") 
} else if (! isTRUE(try(function_has_arguments("plot",c("x"),c("my_motion_graph")))>=1)) {
  DM.result = list(FALSE,"Do not forget to plot your motion graph!")
} else {
  DM.result = list(TRUE,"Bellissimo! Take some time to enjoy the fruits of your labor and play around with the motion chart! In the next exercise you will place the icing on the cake.")
} 
rm(gvisMotionChart_args,student_gvisMotionChart_args,gvisMotionChart_values,student_gvisMotionChart_values,standard_feedback_message,standard_feedback_message2) 
```

--- type:NormalExercise lang:r xp:100 skills:1,4
##  googleVis - the interlude

When working with a simple dataset to visualize, a single color and size for each observation is sufficient. But what if you like to know more? For example, how can you see at a glance which dot respresents which country in your motion chart? Or how these different dots are proportionate to each other? 

To make the motion chart even more understandable you can play with the size and color of each bubble. This way you can present more information into one motion chart. Doing this with googleVis is not that hard, again, you only have to play a little bit with the arguments. 

Let's say you want to make a motion chart that displays the GDP of a country on the x-as and the life expectancy on the y-as. Furthermore, you think it would be nice if each country has a unique color, and the size of each bubble represents the size of the population of that country. Doing this via the `gvisMotionChart()` function, will require adding 4 additional arguments to our existing function:
- xvar = Here you place the column name of the variable to be plotted on the x-axis
- yvar = Here you place the column name of the variable to be plotted on the y-axis 
- colorvar = Here you provide the column name that will make the bubbles change color.  
- sizevar = Here you provide the column name that will make the bubbles change size. 

Time to set this into action...

*** =instructions
- The code you wrote for `my_motion_graph` in the previous exercise is provided. Add the required additional arguments. 
- Make sure the x-as of your motion chart represents the GDP of a country, and the y-as the life expectancy.  
- The bubble color should depend on the country it represents.
- The bubble size should increase if a country has a larger number of citizens.  
- Do not forget to plot your new motion chart.

*** =hint
The four arguments you should add are xvar = "GDP", yvar = "Life Expectancy", colorvar = "Country",and sizevar = "Population".

*** =pre_exercise_code
```{r}
library("rdatamarket")
library("plyr")
library("googleVis")
options(gvis.plot.tag = 'chart')
load(url("http://s3.amazonaws.com/assets.datacamp.com/course/googlevis/googlevis_ex4.RData"))
#life_expectancy = dmlist("15r2!hrp")
#gdp = dmlist("15c9!hd1")
#population = dmlist("1cfl!r3d")
#names(gdp)[3] = "GDP"
#names(life_expectancy)[3] = "Life Expectancy"
#names(population)[3] = "Population"
#development = join(gdp,life_expectancy)
#development= join(development,population)
#development_final = development[development$Year <= 2008,]
#development_final$Country <-as.character(development_final$Country)
#selection <- c("Afghanistan","Australia","Austria","Belgium","Bolivia","Brazil","Cambodia","Azerbaijan", "Chile","China","Denmark","Estonia","Ethiopia","Finlan"""France","Georgia","Germany","Ghana","Greece","India","Indonesia","Iraq","Italy", "Japan","Lithuania","Luxembourg","Mexico","New Zealand", "Niger", "Norway", "Poland", "Portugal","Rwanda", "Somalia", "South Africa", "Spain", "Sweden", "Switzerland", "Turkey", "Uganda", "Ukraine", "United Kingdom", "United States", "Vietnam")
```


*** =sample_code
```{r}

# Include only the countries from `selection` in the interactive motion chart
selection
development_motion = subset(development_final, Country %in% selection)
  
# Create the interactive motion chart with R and `gvisMotionChart)`
my_motion_graph = gvisMotionChart(development_motion,
                    idvar = "Country",
                    timevar = "Year",
                    ___,
                    ___,
                    ___,
                    ___
)

# Plot you new motion graph with the help of `plot()`
```

*** =solution
```{r}
# Include only the countries from `selection` in the interactive motion chart
selection
development_motion = subset(development_final, Country %in% selection)
  
# Create the interactive motion chart with R and `gvisMotionChart())`
my_motion_graph = gvisMotionChart(development_motion,
                    idvar = "Country",
                    timevar = "Year",
                    xvar = "GDP",
                    yvar = "Life Expectancy",
                    colorvar = "Country",
                    sizevar = "Population"
)

# Plot you motion graph with the help of `plot()`
plot(my_motion_graph)
```

*** =sct
```{r}
gvisMotionChart_args = c("data","idvar","timevar", "xvar","yvar","colorvar","sizevar")
student_gvisMotionChart_args =try(names(get_arguments("gvisMotionChart")[[1]]))
standard_feedback_message = "Looks like you forgot to set in `gvisMotionChart()` the arguments"
gvisMotionChart_values = c("Country","Year", "GDP", "Life Expectancy", "Country", "Population")
student_gvisMotionChart_values = try(get_arguments("gvisMotionChart")[[1]]) 
standard_feedback_message2 = "Looks like you set the wrong argument values in `gvisMotionChart()`:"

if (! exists("development_motion")) {
  DM.result = list(FALSE,"Make sure you have taken a subset from `development_final` including only the countries in `selection`.")  
} else if (! identical(development_motion, subset(development_final, Country %in% selection) ) ){
  DM.result = list(FALSE, "You still need to work with a subset from `development_final`. Use the `subset()` function and the `selection` variable. ")
} else if(! "gvisMotionChart" %in% called_functions()) {
  DM.result = list(FALSE,"You should use the `gvisMotionChart()` function in this exercise.")
} else if (! exists("my_motion_graph")) {
  DM.result = list(FALSE,"Mmmm. There is something fishy with your `gvisMotionChart()` function. It doesn't return a result. Check the hint if you need help.") 
} else if (!try(all(gvisMotionChart_args %in% student_gvisMotionChart_args))) { 
  missing_arguments =  gvisMotionChart_args[!(gvisMotionChart_args %in% student_gvisMotionChart_args)]
  DM.result = list(FALSE,paste(c(standard_feedback_message,missing_arguments),collapse=" "))
} else if (!try(all(gvisMotionChart_values %in% student_gvisMotionChart_values))) { 
  missing_values = student_gvisMotionChart_values[! (student_gvisMotionChart_values %in% gvisMotionChart_values)]
  DM.result = list(FALSE,paste(c(standard_feedback_message2,missing_values),collapse=" "))
} else if (! isTRUE(try(function_has_arguments("plot",c("x"),c("my_motion_graph")))>=1)) {
  DM.result = list(FALSE,"Do not forget to plot your new motion graph!")
} else {
  DM.result = list(TRUE,"Isnt't that beautiful! Probably the best stats you've ever seen. Again, play around with the graph and get a good understanding of what it represents. Then head to the final question...")
} 
rm(gvisMotionChart_args,student_gvisMotionChart_args,gvisMotionChart_values,student_gvisMotionChart_values,standard_feedback_message,standard_feedback_message2) 
```


--- type:MultipleChoiceExercise lang:r xp:50 skills:1,4
##  googleVis - the recessional

As mentioned in the beginning, the goal of these charts is (not only) to impress your audience, but also to visualize trends and to provide a more clear view on the data and the corresponding insights. 

To test your understanding of the data, you end this demo chapter by solving the following multiple choice question: "What was de GDP of Germany in 1980, and what was the population size at that time?"   

<i> All the datasets and variables you constructed in the previous exercises are preloaded in your workspace.  Type 'ls()' in the console to see them.</i>

*** =instructions
- 81,653,702 Germans producing a GDP of 11,746
- 78,297,904 Germans producing a GDP of 11,746
- 78,297,904 Germans producing a GDP of 72,2
- 81,653,702 Germans producing a GDP of 30,888

*** =hint
You can still have a look at your interactive motion chart by typing `plot(my_motion_graph_avanced)` into the console. 

*** =pre_exercise_code
```{r}
library("rdatamarket")
library("plyr")
library("googleVis")
options(gvis.plot.tag = 'chart')
load(url("http://s3.amazonaws.com/assets.datacamp.com/course/googlevis/googlevis_ex5.RData"))
#life_expectancy = dmlist("15r2!hrp")
#gdp = dmlist("15c9!hd1")
#population = dmlist("1cfl!r3d")
#names(gdp)[3] = "GDP"
#names(life_expectancy)[3] = "Life Expectancy"
#names(population)[3] = "Population"
#development = join(gdp,life_expectancy)
#development= join(development,population)
#development_final = development[development$Year <= 2008,]
#development_final$Country <-as.character(development_final$Country)
#selection <- c("Afghanistan","Australia","Austria","Belgium","Bolivia","Brazil","Cambodia","Azerbaijan", "Chile","China","Denmark","Estonia","Ethiopia","Finland","France","Georgia","Germany","Ghana","Greece","India","Indonesia","Iraq","Italy","Japan","Lithuania","Luxembourg","Mexico","New Zealand", "Niger", "Norway", "Poland", "Portugal","Rwanda", "Somalia", "South Africa", "Spain", "Sweden", "Switzerland", "Turkey","Uganda", "Ukraine", "United Kingdom", "United States", "Vietnam")
#development_motion = subset(development_final, Country %in% selection)
my_motion_graph = gvisMotionChart(development_motion,idvar = "Country",timevar = "Year",xvar = "GDP",yvar = "Life Expectancy",colorvar = "Country",sizevar = "Population")
```

*** =sct
```{r,eval=FALSE}
msg <- "Have another look at your graph. Use the hint if you need extra help."
okmsg <- "Wunderbar! We hope you enjoyed doing this demo on DataCamp. In the future we plan to do more courses on using interactive visualizations to analyze and present your data via R and googleVis. In the mean time, keep practicing, and maybe you can, sometime, share the stage with Hans Rosling."
test_mc(2, c(msg, okmsg, msg, msg))
```


