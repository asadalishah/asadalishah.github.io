---
title: "Adventures in reading PDF files in R"
layout: post
date: 2019-11-20 10:00
tag:
- Learning R
- Beginner
- Importing Data
- Analytics
- PDF files
- Importing Data from pdf files
blog: true
star: false
---

One of things that got me interested in learning R was its promise to read PDF files. While I have been reading some internal bills data and such, I was in search of publically available data that peeked my interest and converting which would also generate some public good.

Finally, I found a dataset that met both of the above criteria. In this post, I will explain three things: the dataset in pdf format which I decided to import in R, the high level process I followed and the output and the next steps.

## The dataset
I am from Pakistan and I have always been interested to know how much tax elected and nominated public representatives pay. Fortunately, it is mandatory for public representatives to submit their tax returns and the Federal Bureau of Revenue (FBR), the tax collector publishes the tax data of all the public representatives for public consumption. As one would imagine, this data is published in pdf format on their website. The data is quite basic: essentially the name of the public representative, the public body they are a member of, the tax they paid for a fiscal year and if they paid any corporate taxes. I set myself to read this data in R, convert it into analyzable and searchable form, and conduct some very basic analysis. 


## Reading the data in R
To read tabular data from pdf, the package you will need is `tabulizer`. Here is the basic code to read the data using `tabulizer` in R:
```
install.packages("tabulizer")
library(tabulizer)
location <- 'http://download1.fbr.gov.pk/Docs/201922213121521755Parliamentarians.pdf'
out <- extract_tables(location) # a function of tabulizer
class(out) # gives a list of 20 items

```
**out** is a list where element is one tabular page pdf file. We apply `do.call`, another function of the `tabulizer` package which will do whatever we ask it to do to all the elements of the list, in this case `rbind` them i.e., bind the rows.

```
final_data <- do.call(rbind, out[])
```

**final_data** is the combined rows from all the list items from the object **out**. 

A quick examination of this data shows that there are duplicates in the first column with no data under other columns. These duplicates are created by repeat column headers or merged columns containing separators in the middle of a table to signify the start of a new **house** (to be defined shortly) of public representatives. To get rid of these quirks, a bit a basic manipulation to convert it into a seven variable, tidy dataframe. 

## Basic data transformation
The tax data is very basic in nature. It includes the house, the name of the elected or nominated representative, their national tax number, tax paid during that financial year and corporate tax paid, if any.

To do some analysis, I did some data transformation using `dplyr`. Given below is the variables given in the final data set:

1. *seq_all*: a serial number
2. *house*: house or the assembly; senate, national assembly (NAP), KPK assembly (PK), Baluchistan assembly (PB), Punjab assembly (PS) and Sindh assembly (PS).
3. *parli_names*: name of the member
4. *gender*: an attempt at creating a gender variable bases on known females names. Consider this a work in progress; there are many females which would be coded as males as the names ‘library’ is not exhaustive.
5. *fraction_head_count*: each member divided by the total number of members. Total of this would be 100.
6. *cum_head_count*: cumulative total of #5. First member would be 1, second member would be 2 and so on.
7. *tax_paid*: actual tax paid during that year
8. *cum_tax_paid*: sorting the data from least tax paid to most tax paid and adding tax for each member. Suppose there are 4 members and each pays a tax of 0, 0, 5 and 10. So the cumulative total for the first member would be 0, for the second member would again be 0, for the third member would be 5, and for the fourth member would be 15. 
9. fraction_total_tax: tax paid by every member divided by the sum of tax paid by all members
10. cum_tax_total: the concept describe in #8 applied to #9
11. tax_corp: any corporate tax paid by the member during the year


Before getting into some basic statistics, here is a code snippet which would help you search the tax paid by name.

```
test %>% 
  filter(grepl('IMRAN|SHARIF|ASAD', parli_names)) %>%
  select(house, parli_names, tax_paid, fraction_tax_total, tax_corp)
```
The above code will show you the tax paid by anyone whose name includes IMRAN, SHARIF or ASAD.

## Basic statistics

Overall tax paid by the parliamentarians – total, mean and median

```
test %>% group_by(house) %>% summarise(total = sum(tax_paid)/1000000, 
                                   mean = mean(tax_paid)/1000000,
                                   median = median(tax_paid)/1000000)
```

Overall tax paid by house

```
test %>% group_by(house) %>% summarise(total = sum(tax_paid)/1000000, 
                                   mean = mean(tax_paid)/1000000,
                                   median = median(tax_paid)/1000000)
```

All those who paid zero tax during the financial year of 2016-2017.

```
test %>% 
  filter(tax_paid == 0) %>% 
  select(house, parli_names, tax_paid, fraction_tax_total, tax_corp) %>%
  group_by(house) %>%
  select(house, parli_names)
```

## Basic plots
Something which almost always produces interesting results is a contribution plot. By that, I mean was is the percentage contribution of different tax payers in the total tax paid.

This plot didn't disappoint this time either and revealed some shocking statistics.For example, only four parlimentarians pay 25% of the total tax paid by all parlimentarians. 

Given below are two annotated charts. This first chart plots cumulative percentage contribution of tax on y-axis against the percentage contribution of each body count (if you will) on x-axis.

![PDF](<blockquote class="imgur-embed-pub" lang="en" data-id="a/cCN4MnB"><a href="//imgur.com/a/cCN4MnB"></a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>)


The second plots the actual number of parlimentarians on x-axis against the percentage contribution of tax by each one of them on y-axis.

![Gherkin](https://github.com/asadalishah/asadalishah.github.io/blob/master/assets/images/pricesen.png)


## What’s next?
Here is an initial list of what I would like to do next on this project. 

1. First of all, I would like to add previous year’s tax data; this data is for year 2016-2017 published in February 2019. I would like to add the data for previous four years that is available on FBRs website.
2. Secondly, I would like to enhance variables such as gender, province, especially for national assembly and Senate and add city.
3. Thirdly, I would like to make the code dynamic so that the same chart can be calculated for houses dynamically.

Ultimately, I would like to develop a tax data product which is searchable by parliamentarians name, constituency, city, or any other property of interest. 


For those of you, who want to use this code or the data, it is available here. Give me a shout if you would like to work on it together.