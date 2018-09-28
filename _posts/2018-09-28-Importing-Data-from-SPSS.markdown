---
title: "Importing Data from SPSS"
layout: post
date: 2018-09-28 10:00
tag:
- Learning R
- Beginner
- Importing Data
- Analytics
- SPSS file
- Importing SPSS data in R
blog: true
star: false
---

This is my first real blog entry recording my real R learning experiences. The [first blog entry]( https://asadalishah.github.io/Learning-R/) was about making my intentions about recording my experiences public. Just _24_ days since I made my intentions public… So much for the theory purporting public proclamations increase motivation . 

I digress; let’s learn how to import data from SPSS files.

## First thing to remember
SPSS data files are unique; they are layered. That means that the data and labels are saved separately. You may ask what is data and what are labels? 

Let me explain this with an example. Suppose we have an SPSS file that contains only one columns (a variable); employee ID. Each employee ID is linked to a specific employee, obviously. Now what SPSS allows you to do is to go in the file and assign a name against each employee ID. When that is done, you can have two views in SPSS; data view which will show the numeric employee ID and the labels view which will display the employees’ names. And that is layered in a nutshell; you have data as well as labels saved in the file. Now it is possible that this assignment is not done. In that case, SPSS will display numeric employee ID in both data and labels views. Let’s move on.

We don’t know whether assignment is done or not. We have to assume that it is and have to read data and labels separately. 

Therefore, making an SPSS file ready for analysis involves the following steps: 
1. Reading the SPSS file for data
2. Reading the same SPSS file for labels
3. Combining the data and labels file and making it ready for analysis

The file we have is the case-level homicide data. If you are interested, you can read about this data [here]( http://www.murderdata.org/p/data-docs.html). 

I have downloaded the file on my computer and will read from my working directory. 

## Setup and extraction

To read the SPSS file, you will need the `foreign` package. 
```
install.packages("foreign")
library(foreign)
```
Once this step is done, we are ready to extract the data from the file. Let’s start with extracting the labels and data in two data frames.

```
data_labels <- as.tibble(read.spss("SHR76_16.sav", to.data.frame = TRUE))
data_only <- as.tibble(read.spss("SHR76_16.sav", to.data.frame = TRUE, 
                                 use.value.labels = FALSE))
```

We have created two dataframes; **data_label** for labels and **data_only** for data. You noticed that when we use data, we use `use.value.label=FALSE`. This argument is not used in case of labels which means by default `use.value.labels` is TRUE.

Let’s have a quick look at the two datasets we have created. I used the `as.tibble` and just wanted to make sure that whether the data has been imported and converted into a tibble.  

```
View(data_labels)
is.tibble(data_labels)
glimpse(data_only)
nrow(data_labels)
```

## Joining data using dplyr

Let’s have a look at the column names. We can do this using the `names()` function or the `colnames()` function. Both produce the same result.

```
colnames(data_labels)
colnames(data_only)
```
Did you notice that the column names for both files are exactly the same? That shouldn’t be surprising at all. After all, it was ONE layered file which we read twice. 

For joining these two dataframes, we will take the following specific steps: 

1. Identifying which columns have the same data so that we only take them once. 
2. Re-naming the columns to identify them as values or labels.
3. Removing similar columns from one dataset to avoid duplication.
4. Joining the two datasets into a new dataframe
5. Re-ordering the columns to make them useful for visual inspection

The first point above is important as in our case the variable names **ID** in addition to other variables will be repeated in both data and label files having exactly the same data. We need it only once. 

There are two ways to go about the first step: 1) good old visual inspection and 2) the complicated way involving `intersec`, `setdiff`, `sapply` and `merge`, all from the Base R. I will leave the complicated way for the next post and assume that we have the duplicate columns using visual inspection.  

We are ready to execute step 2 and 3 with one neat command. 

```
new_labels <- select(data_labels,
                     ID, CNTYFIPS, Ori, State, Agency, AGENCY_A,
                     Agentype_label=Agentype,
                     Source_label=Source,
                     Solved_label=Solved,
                     Year,
                     Month_label=Month,
                     Incident, ActionType,
                     Homicide_label=Homicide,
                     Situation_label=Situation,
                     VicAge,
                     VicSex_label=VicSex,
                     VicRace_label=VicRace,
                     VicEthnic, OffAge,
                     OffSex_label=OffSex,
                     OffRace_label=OffRace,
                     OffEthnic,
                     Weapon_label=Weapon,
                     Relationship_label=Relationship,
                     Circumstance_label=Circumstance,
                     Subcircum, VicCount, OffCount, FileDate,
                     fstate_label=fstate,
                     MSA_label=MSA)
```

We kept the common identifier variables (e.g., ID, CNTYFIPS, Ori, State, Agency, AGENCY_A), renamed variables by adding a suffix **_label**, re-ordered them, and saved them in a new dataframe names **new_labels**. All of this using the `select`  function. Awesome!

Let’s do the same for the data_only dataframe.

```
new_data_only <- select(data_only,
                        Agentype_value=Agentype,
                        Source_value=Source,
                        Solved_value=Solved,
                        Month_value=Month,
                        Homicide_value=Homicide,
                        Situation_value=Situation,
                        VicSex_value=VicSex,
                        VicRace_value=VicRace,
                        OffSex_value=OffSex,
                        OffRace_value=OffRace,
                        Weapon_value=Weapon,
                        Relationship_value=Relationship,
                        Circumstance_value=Circumstance,
                        fstate_value=fstate,
                        MSA_value=MSA)
```

Now, these new dataframes ** new_labels ** and ** new_data_only ** are ready to be joined. It can be achieved using the `cbind` from the Base R[^1]. 

```
new_data <- as.tibble(cbind(new_labels, new_data_only))
```

Wala… we have a new dataframe named **new_data** and we are ready to reorder it. We will put similar labels and value columns next to each other. Believe it or not, this is also done using `select`.

```
new_data <- select(new_data,
                   ID, CNTYFIPS, Ori, State, Agency, AGENCY_A,
                   Agentype_label, Agentype_value,
                   Source_label, Source_value,
                   Solved_label, Solved_value,
                   Year,
                   Month_label, Month_value,
                   Incident, ActionType,
                   Homicide_label,Homicide_value,
                   Situation_label,Situation_value,
                   VicAge,
                   VicSex_label,VicSex_value,
                   VicRace_label,VicRace_value,
                   VicEthnic, OffAge,
                   OffSex_label,OffSex_value,
                   OffRace_label,OffRace_value,
                   OffEthnic,
                   Weapon_label,Weapon_value,
                   Relationship_label,Relationship_value,
                   Circumstance_label,Circumstance_value,
                   Subcircum, VicCount, OffCount, FileDate,
                   fstate_label,fstate_value,
                   MSA_label,MSA_value)

```

And here you go, the combined data set is ready for your inspection using the **View** function. Remember, it is view with a Capital V.

```
View(new_data)
```

My next post would be in a week’s time and would be about automating the first step of the process i.e., identifying which columns names are the same and do they contain the same data or not. 


***

#### Footnotes

[^1]: We used **cbind** as we knew that both the dataframes are the same i.e., same ID and same sequence. If we had two datasets which were of different dimensions, say one had fewer rows, but with one unique key to link, we would have used **base::merge** or **dplyr::join** to ensure that data is aligned with a KEY value (say ID.)