---
author:
  - Dainish Jabeen
bibliography:
  - refs.bib
date: 2023-10-01
title: R
---

# Data Types

Focus is on numerical types: Scalers, Vectors, Matrices, Data Frames and Lists.

```R
x <- 28 #Initalise var

vect <- c(1,10,100) #Initalise vector

vect < 0 # Auto does per component iteration

vect[(vect<0)] #Outputs all values < 0, () creates index

(cond && cond ..) #Create multi condition index
```
## Matrix

```R
matrix(data,nrow,ncol,byrow=T/F) #byrow if filled left to right

a <- matrix(1:10,byrow=TRUE,nrow=5) #Creates 5x2 matrix 

cbind(matrix,vector) #Combine matrix and vectors

m[1,] #all elements of first row
m[,1] #all elements of first col
```
## Data Frames

```R
data.frame(df,stringsAsFactors=T/F) # Converts strings to factors

df[row,col]
df[,c("col id 1","col id 2")]

df$newcol <- newcol #Add new col

subsetdf <- subset(df,subset=cond) #Create subset df

df <- df[order(df$col),] #Arrange by col order

df[,colnames(df) %in% list] #Output all col names in list

select(df,cols) #Choose desired cols
filter(df,cond)

table(df) #Cont numb of observations per level
```


**Tibble data frames**

-   Can have list in cols
-   Auto extends to match col rows
## Lists

Can hold any other data type in an array type.

# R Factor

Categorize and store data, in categorical variables.

$$factor(x=character(),levels,labels=levels,added=is.ordered(x))$$

Stores strings into categories to be used in ML tasks.

# Dplyr

```R
left_join() #keep data from original
right_join() #keep data from destination data

inner_join() #exclude unmatched cols
full_join() #keep everything

mutate(df,var=condition,..) #Create new var 
na.omit() #Remove all NA items

na.rm = True #Ignore missing vals

mutate(col_name=ifelse(condition,val,col)) 
#If condition met will replace col value with val
```

# Tidyr

```R
gather() #Convert from wide to long (change inner vals into a new col)
spread() #Convert from long to wide 

separate() #Split data in a col to multiple cols
unite() #Combine data from cols into a col

diff() #Return lagged and iterated difference
```
# Functions

```R
function.name <- function(arg)
        {
        }
        
rm{func} #removes func
```

## Useful Functions

```R
ls(environment()) #Print currently global vars and funcs    

apply(x,margin=1/2,func) #Apply func on all rows or cols (1:row,2:col)
lapply(x,func) #Apply func output list
sapply(x,func) #Output matrix
tapply(x,index,func) #

#Impute example

new_df <- data.frame(
	sapply(df,function(col) ifelse(is.na(col),mean(x,na.rm=True),col))
)

summarise() #Basic stats

group_by(col) #Group df by chosen id
nth() #Goto nth val

arrange() #Sort works with pipelines
```
# Loops

Python like.

```R
for item in list{
}
```

# Pipeline

Use the operator % \> % to allow operations on the same data through stages.

```R
New_df <- df %>%
step_1 %>%
step_2 ...
```
