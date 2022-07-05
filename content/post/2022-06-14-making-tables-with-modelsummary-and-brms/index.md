---
title: Making tables with {modelsummary} and {brms}
author: Michael E. Flynn
date: '2022-06-14'
slug: []
categories:
  - Academia
  - Statistics
  - Models
  - Publishing
  - Methods
  - Data  Science
  - brms
tags:
  - brms
  - stats
  - methods
  - data science
  - blogging
  - publishing
subtitle: ''
summary: ''
authors: []
lastmod: '2022-06-14T15:00:31-05:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

**tl;dr: Learn how to make some cool, customizable, tables for multinomial logit models using {brms} and {modelsummary}.**

# Background

The title is a little too general—this is really a post about making some nice, publication-quality tables specifically for multinomial models using {brms} and {modelsummary}. I'm coming off a couple of long projects where we were running a lot of these. Actually, started using {brms} several years ago partly because I had data where we had 1) lots of individuals making choices, 2) those individuals were all grouped in some pretty clear ways, and 3) we were also interested in modeling group-level characteristics that might relate to individuals' choices. This more or less marked my full transition from Stata to R and from frequentist stats to Bayesian stats. 

Early on the biggest problem I ran into was finding a way to generate tables for multinomial models run using {brms}. Initially most packages didn't support {brms} and/or developing tables for multilevel/hierarchical models required a lot of extra legwork. Building tables to accomodate choice models was another issue. Taken together, these problems meant that I had to write a lot of extensive code by hand to generate clean Latex tables for the models I was using. The process has, thankfully, become much simpler over the last couple of years.

First, for those who aren't familiar {brms} is an amazing package created by [Paul B&uuml;rkner](https://paul-buerkner.github.io/brms/). It stands for Bayesian Regression Modeling using Stan, and, as the name suggests, provides users with a convenient front-end for building regression models using Stan as a back end. This is particularly useful for building multilevel models, but also lots of other stuff.

Second, {modelsummary} is another amazing and ever-expanding package created by [Vincent Arel-Bundock](https://vincentarelbundock.github.io/modelsummary/). This packages does a few different things, but most importantly and prominently it helps users to create excellent tables for summarizing data and building tables for regression output. The package is incredibly flexible, supports dozens of different model types, and Vincent is constantly adding new features. 

Both packages are fantastic. If you're interested in Bayesian modeling, or just looking for a new and easy way to build tables for whatever modeling package you already use, you should check out one or both of these packages.


# Multilevel Multinomial Logit Models

Lots of regression models are going to be fairly simple to present in a table format, and there are some fairly easy ways to go about generating those tables. Typically you'll have a single column per model, and each row of your table will be for a single variable. You might also have some summary statistics for the model at the bottom (think $ N $, $ R^2 $, etc.). 

Multinomail models get a little more complicated because you'll typically have multiple outcomes. Specifically, you'll have $k-1 $ columns to present in the table, where $k $ is the number of choices respondents have. In our case we had lots of models where survey respondents offered their assessments of various actors and we condensed those assessments down into four general categories: Positive, Neutral, Negative, and Don't Know. This means we ultimately ended up with three columns per model, with the "Neutral" response serving as the baseline category against which the others were compared.

Multilevel models complicate things slightly because you may also have summary statistics for the groups in your data in addition to the general summary statistics for the model. 

Last, Bayesian models and {brms} specifically provide users with a ton of additional information they might want to present beyond the traditional stuff you'd find in frequentist models. Some of this information can take a *long* time to compile. There's often going to be an efficiency and transparency tradeoff here, and so you may want to customize what you present in your table. Even if you end up presenting lots of information in the end, having the ability to control what's in your table at the outset can be really useful as you run the code to make sure the basic output looks right. 

Anyway, the goal here is rather niche, but it's to talk through the process of building nice and readable tables when we're using these models and have lots of information to present. {modelsummary} lets us do this, but also requires a bit of additional effort to fully customize our output.


# Getting Started

Ok, first we're going to load our libraries. The relevance of some of these is immediately obvious given what I've already said, but I'll talk more about some of the additional packages we need below. 


```r
library(tidyverse)
library(modelsummary)
library(brms)
library(broom)
library(broom.mixed)
```
