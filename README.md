# Factor-analysis
## Introduction
Factor analysis is a statistical technique used widely in psychology and the social sciences for the purpose of identifying unobservable constructs (factors) from a set of observables. There are two primary uses of factor analysis: 
1.	Exploring how many different factors are captured by the data, which measures are capturing which factors, and whether we can reduce the number of measures we use and still capture the same constructs. This is exploratory factor analysis, or EFA, which I focus on in this post.  
2.	Performing checks that a set of measures captures the constructs the researcher intended to capture. This is done using confirmatory factor analysis, or CFA. 

The capabilities of EFA are best explained through an example. Suppose you are a recruitment agency, performing tests on job applicants and providing information about applicant quality to employers. You capture all school qualification scores and conduct a long series of tests: a basic numeracy test, a basic verbal test, logic puzzles, an IQ test, etc. You provide all these results to potential employers. But employers are confused when presented with this much information. They don’t want to compare ~20 numbers across 100 people. In addition, administering all these tests is costly and time consuming for applicants. They may choose a different recruitment agency. EFA will:
1. Show you how the different tests are related to each other
2. Give you a set of underlying qualities (factors) measured by the tests
3. Enable you to reduce the number of tests you do, without losing the ability to capture these underlying qualities

Having performed EFA, you were able to identify three underlying candidate characteristics: creativity, verbal ability and numeracy. This made your candidate reports for employers much more accessible. In addition, you were able to identify redundant variables; once you had applicants’ school results, you only had to perform two additional tests to capture all three characteristics.

As this example illustrates, EFA can be a very useful tool. For researchers hoping to capture unobservable constructs, such as psychological or personality traits, EFA can help to legitimise the measures used and reduce costs by identifying redundant measures. So, how does it work?

## Explanation
It is very hard to establish what is driving correlations when presented with a correlation matrix of a large number of measures. EFA works by asking what might account for all of these correlations. In other words, EFA is a method for investigating whether a number of variables of interest Y_1, Y_2, …, Y_k, are linearly related to a smaller number of unobservable factors F_1, F_2, …, F_k. For example:

Y_1=β_10+β_11 F_1+β_12 F_2+e_1
Y_2=β_20+β_21 F_1+β_22 F_2+e_2
Y_3=β_30+β_31 F_1+β_32 F_2+e_3

Coefficients β_ij are referred to as factor loadings, of variable Y_i on factor F_j. As factors are unobservable, we cannot simply estimate the loadings by regression. Instead, we use factor analysis models. The simplest example is one which satisfies the following assumptions: 
1. The error terms e_i are independent of one another, and such that E(e_i)=0 and Var(e_i)=σ_i^2
2. The unobservable factors F_j are independent of one another and of the error terms, and are such that E(F_j)=0 and Var(F_j)=1
   
>[!NOTE]
>Note, more complex models allow for Assumption 2 to be relaxed; I will come back to this.

The assumptions imply that the Variance of Y_i consists of:

Var(Y_i)=β_i1^2+β_i2^2+σ_i^2

The β_i1^2+β_i2^2 component is called communality, or the part that is explained by the common factors F_1 and F_2; σ_i^2 is the specific variance, or the part of the variance that is not accounted for by the common factors. 

The final term to introduce is uniqueness, which is the percentage of the variance that is not explained by the common factors, or 1-communality. Under the assumptions of the model, uniqueness is σ_i^2. However, if the assumptions are violated and uniqueness is not pure measurement error, it is accounting for something particular to the variable. The greater the uniqueness, the more likely that it is more than just measurement error. Values more than 0.6 are usually considered high. If the uniqueness is high, then the variable is not well explained by the factors.

## Factor analysis in STATA

The model can be solved using several methods in Stata, using the command -factor-. The syntax is:

factor varlist [if] [in] [weight] [, method options]  

The command offers a choice of the following methods for estimating factor loadings based on the correlation matrix:

**Method options:**
- **pf**	principal factor (the default). The factor loadings are computed using the squared multiple correlations as estimates of the communality
- **pcf**	principal-component factor. Unlike pf, the communalities are assumed to be 1 - meaning there is no uniqueness. The model will not be appropriate if the proportion of the variation explained by unique variation is high 
- **ipf**	iterated principal factor. This method re-estimates the communalities iteratively 
- **ml**	maximum likelihood factor. This method assumes that the data are multivariate normal distributed

  ### Example 1
  Example 1 shows the output displayed when using the default method for estimating the factor structure of a set of measures designed to capture self-efficacy, or a person’s judgement of how well they can deal with prospective situations. The log file shows output for this example using the different estimation methods. 

The first column of the first table, Factor, will always equal the number of variables: in this case we tested 10 measures trying to capture self-efficacy. The eigenvalues in the second column show the total variance accounted for by each factor. The difference column shows the magnitude of the differences between one eigenvalue and the next. Proportion indicates the relative weight of each factor in the total variance: for example, Factor 1 accounts for: 2.72789/(2.72789+...-0.16821)=1.1351of the variance. Cumulative column simply shows the cumulative proportion of the variance accounted for, always ending in 1. The second table provides the estimates of factor loadings and uniqueness.  

There are several options within -factor- which are listed in the Stata manual; however, two options deserve particular mention at this stage. We can choose the number of factors from the first table, overriding the manual settings:
- factors(#) 	sets the maximum number of factors to be retained
- mineigen(#)	sets the minimum value of eigenvalues to be retained. The default for all methods except pcf is a number very close to zero, meaning that factors associated with negative eigenvalues will not be retained. The default for pcf is 1 and this value is more common in literature. In Example 1, specifying mineigen(1) would lead to only Factor 1 being retained.  

![factora1](https://github.com/csae-coders-corner/Factor-analysis/assets/148211163/5f21d70e-0b3c-4a99-8a63-0b09d8ead0e3)


## Rotation
The second table of Example 1 provides a set of solutions; however, there is no unique set of solutions as there are infinite sets of loadings which fit the same model. We seek a set of loadings that fit the observations equally well as the output of -factor- but are easier to interpret. This is achieved through a rotation and the command for this is -rotate-:

rotate [, options]  

There are infinite possible rotations; however, there are certain methods which help to specify the kind of solution we are seeking. We can adapt our assumptions about factors being independent of each other at this stage; in the options you can specify:
- oblique 	For an oblique rotation to be applied, instead of the default, which is the orthogonal rotation. The factors before rotation are usually orthogonal, whereas the oblique rotated factors can be correlated


Depending on the choice between an orthogonal and an oblique rotation, the following rotation methods can be specified (this is a subset of the most commonly used rotations; for full details, consult the -rotate- chapter of the Stata manual):
- varimax	This is the default for an orthogonal rotation. It seeks the rotated loadings that maximize the variance of the squared loadings for each factor. The idea is to make some loadings as large as possible, and the rest as small as possible in absolute value
- quartimax 	This is an alternative to varimax; it maximizes the variance of the squared loadings within the variables and tends to produce factors with high loadings for all variables
- oblimin(#)	This option is suitable for both orthogonal and oblique rotations. The default is oblimin(0), with values above 0 not recommended for oblique rotations. 
  
### Example 2: Final output
Based on the output from Example 1, the researcher decides to model the data using a two factor structure, performing an oblique rotation due to the likelihood of factors being correlated. The full output is displayed in the log file. After rotating, a further useful command is -sortl- which sorts the variables in order of factor loadings, making it easier to see which variables load in each factor. The final set of factor loadings is shown below. 
Based on the results, the researcher would conclude that items 4, 5, 9 and 10 capture one construct while items 1, 2, 3a, 6, 7 and 8 capture a different construct. To get a sense of what the constructs could be, the researcher would explore the original measures. Perhaps some of the measures were negatively-worded, while some were positively worded. Alternatively, the two sets could be capturing slightly different aspects of self-efficacy, for example believing in yourself and coping when faced with adversities. 

![factora2](https://github.com/csae-coders-corner/Factor-analysis/assets/148211163/6bf2e7e1-ce75-4e62-8a99-47978a326e7c)

Next, the researcher might consider whether some items are redundant, based on the factor loadings. Generally, loadings below 0.3 are considered low. Therefore, the researcher might choose to conduct this analysis dropping each of the lowest scoring items, 2 and 9. If the results remain similar, the researcher might be able to drop these in future studies, saving some valuable survey time. 
Finally, the high level of uniqueness within the variables should remain a concern for the researcher, as it indicates that factors do not account for a very large proportion of the variance. This may throw doubt on the researcher’s existing metrics and encourage them to seek alternative measures.  

## References and further reading
1.	Stata (2019). Multivariate Statistics Reference Manual. Stata Press.
- mvfactor available at: https://www.stata.com/manuals13/mvfactor.pdf and the references within
- mvrotate available at: https://www.stata.com/manuals13/mvrotate.pdf and the references within
2.	Paul Kline (1994). An easy guide to factor analysis. Routledge. This is a well written, light introduction to the key concepts
3.	Oscar Torres-Reyna. Getting Started in Factor Analysis (using Stata 10). A short set of useful slides available at: https://dss.princeton.edu/training/Factor.pdf
4.	Hoelzle & Meyer (2012). Exploratory factor analysis: Basics and beyond. Handbook of Psychology, Second Edition, 2. 
5.	Peter Tryfos, chapter on factor analysis available at: http://www.yorku.ca/ptryfos/methods.htm

**Marta Grabowska, Research Assistant in Development Economics, Blavatnik School of Government, 03 December 2019**
