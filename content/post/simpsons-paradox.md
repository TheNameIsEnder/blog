+++

author = "Jacob Hell"
title = "Introduction to Simpson's Paradox"
date = "2021-01-26"
description = "Introduction to Simpson's Paradox"
tags = [ "statistics" ]

+++

Introduction and how to avoid it.

<!--more-->

## Definition

Simpson's Paradox can be defined like so:

```
a/b < A/B
c/d < C/D, and
(a+c) / (b/d) > (A+C) / (B+D)
```

Applying this to an example based loosely on real life:

```
Suppose that a University is trying to discriminate in favour of women when hiring staff. It advertises positions in the Department of History and in the Department of Geography, and only those departments. Five men apply for the positions in History and one is hired, and eight women apply and two are hired. The success rate for men is twenty percent, and the success rate for women is twenty-five percent. The History Department has favoured women over men. In the Geography Department eight men apply and six are hired, and five women apply and four are hired. The success rate for men is seventy-five percent and for women it is eighty percent. The Geography Department has favoured women over men. Yet across the University as a whole 13 men and 13 women applied for jobs, and 7 men and 6 women were hired. The success rate for male applicants is greater than the success rate for female applicants.
```

This can be simplified as:

```
History Major:
Men: 1/5	<	Women: 2/8

Geography Major:
Men: 6/8	<	Women: 4/5

University Total:
Men: 7/13	>	Women: 6/13

```

Above shows the paradox. How can women be favored in individual majors, but men are favored overall? The answer to this puzzle is that more women are applying for a competitive major, but more men are applying for a major with less competition. In other cases of Simpson's Paradox, different confounding variables are the cause.

## Avoiding Simpson's Paradox

Assuming you can't change the study, make sure that the results hold up to the subsets of the data. If there is a large discrepancy with the samples, weight the data accordingly. 

Lastly, check for confounding variables, and check their interaction with your results. Here are some suggestions:

**Restriction**: restrict the samples b only including certain subjects that have the same values of potential confounding variables.

**Matching**: Match the subjects of treatment group with a counterpart in comparison group. The matched subjects must have the same values on the confounding variables.

**Statistical Control**: Include potential confounders as variables in regression.

**Randomization**: Assign treatment to a study randomly in a large number of subjects.