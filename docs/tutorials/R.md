---
layout: default
title: R
parent: Tutorials
nav_order: 13
---




# R


R is a programming language and free software environment for statistical computing and graphics. It is widely used in academia and industry for data analysis, data visualization, and statistical modeling.










## formatR

[formatR](https://yihui.org/formatr/) is an R package that provides a set of tools for formatting R code. It includes functions for reformatting code, checking code style, and reordering function arguments. It can be installed from CRAN using the following command:



```bash
tidy_source(source = "R/met_2bw.R", indent =4, brace.newline = TRUE, width.cutoff = 50, args.newline = TRUE, file = "./new.R")

```