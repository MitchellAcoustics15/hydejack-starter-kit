SSID Intraclass Correlation
================
Andrew Mitchell
15 April 2020

Intraclass Correlation
======================

This section is heavily taken from the book *Multilevel Modeling Using R* by Finch, Bolin, & Kelley.

This notebook will serve two purposes: one is practicing my math skills in R and understanding of some of the underlying maths in multi-level modeling by coding some analysis from scratch; the other is to test the importance of a multilevel model for a Soundscape Index by analysing the intraclass correlation for our dataset.

Intro
-----

Within our SSID dataset, we have what I consider to be an obvious case for a multi-level model. We have over a thousand individual responses taken from several discrete locations, which fit into broader architectural categories, from several different countries. It could be argued that each of these different groups will have some sort of influence on the perception of the acoustic environment (and, in fact, I have shown this elsewhere).

In order to test this, I will follow a section from Chapter 2 of *Multilevel Modeling Using R* to calculate a measure of the intraclass correlation to determine to what degree the dependent variable (the soundscape descriptor) depends on the class. The principles we will be investigating are summarised in the extracts below:

> In cases where individuals are clustered or nested within a higher-level unit (e.g. SessionIDs, LocationIDs, Sound Source Profiles (SSPs)), it is possible to estimate the correlation among individuals' scores within the cluster/nested structure using the intraclass correlation (denoted *ρ*<sub>*I*</sub> in the population). The *ρ*<sub>*I*</sub> is a measure of the proportion of variation in the outcome variable that occurs between groups versus the total variation present and ranges from 0 (no variance between clusters) to 1 (variance between clusters but not within cluster variance). *ρ*<sub>*I*</sub> can also be conceptualised as the correlation for the dependent measure for two individuals randomly selected from the same cluster. It can be expressed as:

$$ \\rho\_I = \\frac{\\tau^2}{\\tau^2 + \\sigma^2} $$

> **Higher values of *ρ*<sub>*I*</sub> indicate that a greater share of the total variation in the outcome measure is associated with cluster membership; i.e. there is a relatively strong relationship among the scores for two individuals from the same cluster. Another way to frame this issue is that individuals within the same cluster (e.g. SSP) are more alike on the measured variable than they are like those in other clusters.**

Loading our dataset
-------------------

As always, the first thing we need to do is load our dataset and cut it down to the factors we care about.

``` r
library(tidyverse)
library(here)
library(psychometric)
library(dplyr)
library(readxl)
library(kableExtra)

ssid.data <- read_excel(here("data", "2020-04-15", "AllLondon_mono_mean_200414.xlsx"))
# Make sure all of the relevant response columns are integers
ssid.data <- ssid.data %>% mutate_at(vars(Traffic, Other, Human, Natural,
                                          pleasant, chaotic, vibrant, uneventful,
                                          calm, annoying, eventful, monotonous,
                                          overall, appropriateness, loudness, sss04, sss05,
                                          who01, who02, who03, who04, who05, WHO_sum,
                                          Age, Gender, edu00, eth00,
                                          misc00, misc03, misc01,
                                          use00, uni00),
                                     funs(as.integer))
# Set GroupID, SessionID, Location as factor type
ssid.data <- ssid.data %>% mutate_at(vars(GroupID, SessionID, LocationID),
                                     funs(as.factor))
acoustic_vars = c('LZeq', 'LCeq', 'LAeq', 'LAmin', 'LAmax', 'LA_5', 'LA_10', 'LA_50', 'LA_90', 'N', 'Nmax', 'N5', 'N10', 'N50', 'N90', 'N95', 'R', 'Rmin', 'Rmax', 'R5', 'R10', 'R50', 'R90', 'R95', 'S', 'S.1', 'Smax', 'S5', 'S10', 'S50', 'S90', 'S95', 'T', 'Tmax', 'T5', 'T10', 'T50', 'T90', 'T95', 'FS', 'FSmin', 'FSmax', 'FS5', 'FS10', 'FS50', 'FS90', 'FS95', 'THD', 'THDmin', 'THDmin_freq', 'THDmax', 'THDmax_freq', 'THD5', 'THD10', 'THD50', 'THD90', 'THD95', 'I', 'Imin', 'Imax', 'I5', 'I10', 'I50', 'I90', 'I95', 'SIL.rms.', 'SIL.av.', 'SILmin', 'SILmax', 'SIL5', 'SIL10', 'SIL50', 'SIL90', 'SIL95', 'SII.rms.', 'SII.av.', 'SIImin', 'SIImax', 'SII5', 'SII10', 'SII50', 'SII90', 'SII95', 'LA10_LA90', 'LA5_LA95', 'LCeq_LAeq')

indep_vars = c('LAeq', 'LA_50', 'LA_5', 'LA_95', 'LCeq_LAeq', 'S5', 'S95', 'N5', 'N95')

#cut down the dataset
ssid.data <- na.omit(ssid.data[c("LocationID", "Pleasant", "Eventful", "loudness", "overall", indep_vars)])
```

We also need to add a new datapoint - the Sound Source Profile that each location belongs to. This has been defined elsewhere, so we won't discuss it in depth. The basic concept is that all urban public spaces can be sorted into 1 of 5 groups based upon the composition of sound sources present in the location. The five SSPs are:

1.  Natural Deficient -&gt; TorringtonSq, EustonTap, & CamdenTown
2.  Low Noise
3.  Human Dominant -&gt; TateModern, StPaulsCross & StPaulsRow
4.  Natural Dominant -&gt; RegentsParkFields, RegentsParkJapan, & RusellSq
5.  Mixed Sources -&gt; Pancras Lock & MarchmontGarden

Within London, we don't have any sites which fall into the 'Low Noise' SSP, so it is not included here.

``` r
ssid.data <- mutate(ssid.data, 
                   SSP = ifelse(LocationID == 'TorringtonSq' | LocationID == 'EustonTap' | 
                                LocationID == 'CamdenTown', 
                                'Nat.Def', 
                         ifelse(LocationID == 'TateModern' | LocationID == 'StPaulsCross' | 
                                LocationID == 'StPaulsRow', 
                                'Hum.Dom', 
                         ifelse(LocationID == 'RegentsParkFields' | LocationID == 'RussellSq' | 
                                LocationID == 'RegentsParkJapan', 
                                'Nat.Dom', 
                         ifelse(LocationID == 'PancrasLock' | LocationID == 'MarchmontGarden',
                                'Mixed', "NA")))))
                                    
#Make sure to redefine it as a factor variable
ssid.data$SSP <- as.factor(ssid.data$SSP)
```

Let's take a look at the data to make sure we know what we're working with. Here we can see we have around 1000 responses (one per row) and we can see what location they were taken in, the respondent's rating of the Pleasantness, Eventfulness, and loudness of the soundscape and how good they thought the overall soundscape was. We also have a series of acoustic metrics measured for approx. 30s while the respondent was fillout out the survey. For this analysis we will only need the LocationID and the survey responses, not the acoustic metrics.

``` r
kable(head(ssid.data))
```

<table>
<thead>
<tr>
<th style="text-align:left;">
LocationID
</th>
<th style="text-align:right;">
Pleasant
</th>
<th style="text-align:right;">
Eventful
</th>
<th style="text-align:right;">
loudness
</th>
<th style="text-align:right;">
overall
</th>
<th style="text-align:right;">
LAeq
</th>
<th style="text-align:right;">
LA\_50
</th>
<th style="text-align:right;">
LA\_5
</th>
<th style="text-align:right;">
LA\_95
</th>
<th style="text-align:right;">
LCeq\_LAeq
</th>
<th style="text-align:right;">
S5
</th>
<th style="text-align:right;">
S95
</th>
<th style="text-align:right;">
N5
</th>
<th style="text-align:right;">
N95
</th>
<th style="text-align:left;">
SSP
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
CamdenTown
</td>
<td style="text-align:right;">
-0.2196699
</td>
<td style="text-align:right;">
0.4267767
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
69.40
</td>
<td style="text-align:right;">
68.48
</td>
<td style="text-align:right;">
72.805
</td>
<td style="text-align:right;">
62.885
</td>
<td style="text-align:right;">
8.155
</td>
<td style="text-align:right;">
2.395
</td>
<td style="text-align:right;">
1.720
</td>
<td style="text-align:right;">
29.95
</td>
<td style="text-align:right;">
16.05
</td>
<td style="text-align:left;">
Nat.Def
</td>
</tr>
<tr>
<td style="text-align:left;">
CamdenTown
</td>
<td style="text-align:right;">
0.0000000
</td>
<td style="text-align:right;">
0.2500000
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
69.40
</td>
<td style="text-align:right;">
68.48
</td>
<td style="text-align:right;">
72.805
</td>
<td style="text-align:right;">
62.885
</td>
<td style="text-align:right;">
8.155
</td>
<td style="text-align:right;">
2.395
</td>
<td style="text-align:right;">
1.720
</td>
<td style="text-align:right;">
29.95
</td>
<td style="text-align:right;">
16.05
</td>
<td style="text-align:left;">
Nat.Def
</td>
</tr>
<tr>
<td style="text-align:left;">
CamdenTown
</td>
<td style="text-align:right;">
-0.4696699
</td>
<td style="text-align:right;">
0.1767767
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
69.40
</td>
<td style="text-align:right;">
68.48
</td>
<td style="text-align:right;">
72.805
</td>
<td style="text-align:right;">
62.885
</td>
<td style="text-align:right;">
8.155
</td>
<td style="text-align:right;">
2.395
</td>
<td style="text-align:right;">
1.720
</td>
<td style="text-align:right;">
29.95
</td>
<td style="text-align:right;">
16.05
</td>
<td style="text-align:left;">
Nat.Def
</td>
</tr>
<tr>
<td style="text-align:left;">
CamdenTown
</td>
<td style="text-align:right;">
0.1035534
</td>
<td style="text-align:right;">
-0.7500000
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
70.55
</td>
<td style="text-align:right;">
69.34
</td>
<td style="text-align:right;">
74.800
</td>
<td style="text-align:right;">
64.620
</td>
<td style="text-align:right;">
6.585
</td>
<td style="text-align:right;">
2.550
</td>
<td style="text-align:right;">
1.760
</td>
<td style="text-align:right;">
33.10
</td>
<td style="text-align:right;">
17.60
</td>
<td style="text-align:left;">
Nat.Def
</td>
</tr>
<tr>
<td style="text-align:left;">
CamdenTown
</td>
<td style="text-align:right;">
0.2500000
</td>
<td style="text-align:right;">
0.7500000
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
66.34
</td>
<td style="text-align:right;">
65.72
</td>
<td style="text-align:right;">
69.245
</td>
<td style="text-align:right;">
63.825
</td>
<td style="text-align:right;">
7.725
</td>
<td style="text-align:right;">
2.185
</td>
<td style="text-align:right;">
1.745
</td>
<td style="text-align:right;">
24.10
</td>
<td style="text-align:right;">
17.10
</td>
<td style="text-align:left;">
Nat.Def
</td>
</tr>
<tr>
<td style="text-align:left;">
CamdenTown
</td>
<td style="text-align:right;">
0.0732233
</td>
<td style="text-align:right;">
0.6767767
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:right;">
2
</td>
<td style="text-align:right;">
71.54
</td>
<td style="text-align:right;">
70.33
</td>
<td style="text-align:right;">
75.750
</td>
<td style="text-align:right;">
67.765
</td>
<td style="text-align:right;">
7.000
</td>
<td style="text-align:right;">
2.655
</td>
<td style="text-align:right;">
1.940
</td>
<td style="text-align:right;">
36.10
</td>
<td style="text-align:right;">
22.65
</td>
<td style="text-align:left;">
Nat.Def
</td>
</tr>
</tbody>
</table>
Calculate the required statistics from the dataset
--------------------------------------------------

We'll get to the exact formulae in a minute, but first I want to run through what basic statistics we'll need in order to make our calculations. For each SSP, we need its sample size, *N*; the mean of Pleasantness; and the variance for Pleasantness. We also need the overall mean Pleasantness

To do this, we will use the `dplyr` package, which we already loaded above.

``` r
stats.table <- ssid.data %>%                       # specify data frame
                    group_by(SSP) %>%               # specify group indicator
                    summarise(n=n(), 
                              mean=mean(Pleasant),
                              var=var(Pleasant))   # specify functions

kable(stats.table)
```

<table>
<thead>
<tr>
<th style="text-align:left;">
SSP
</th>
<th style="text-align:right;">
n
</th>
<th style="text-align:right;">
mean
</th>
<th style="text-align:right;">
var
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Hum.Dom
</td>
<td style="text-align:right;">
242
</td>
<td style="text-align:right;">
0.3076494
</td>
<td style="text-align:right;">
0.1010241
</td>
</tr>
<tr>
<td style="text-align:left;">
Mixed
</td>
<td style="text-align:right;">
163
</td>
<td style="text-align:right;">
0.2315103
</td>
<td style="text-align:right;">
0.1656291
</td>
</tr>
<tr>
<td style="text-align:left;">
Nat.Def
</td>
<td style="text-align:right;">
273
</td>
<td style="text-align:right;">
-0.0775404
</td>
<td style="text-align:right;">
0.1111350
</td>
</tr>
<tr>
<td style="text-align:left;">
Nat.Dom
</td>
<td style="text-align:right;">
326
</td>
<td style="text-align:right;">
0.5166178
</td>
<td style="text-align:right;">
0.1194867
</td>
</tr>
</tbody>
</table>
``` r
mean.Pleasant <- mean(ssid.data$Pleasant)
print(mean.Pleasant)
```

    ## [1] 0.2584026

Calculate *σ*<sup>2</sup> (variation within clusters)
-----------------------------------------------------

The sample estimate for variation within clusters is defined as:

$$ \\hat{\\sigma}^2 = \\frac{\\sum\_{j=1}^C (n\_j - 1)S\_j^2}{N - C} $$

where

*S*<sub>*j*</sub><sup>2</sup> = variance within cluster $j = \\frac{\\sum\_{i=1}^{n\_j} (y\_{ij} - \\bar{y}\_j)}{(n\_j - 1)}$

*n*<sub>*j*</sub> = sample size for cluster *j*

*N* = total sample size

*C* = total number of clusters

Using this and the `stats.table`, we can calculate $\\hat{\\sigma}^2$:

``` r
sigma.calc <- function(stats.table) {
    N = sum(stats.table$n)
    C = nrow(stats.table)
    
    top = 0
    for (i in 1:C) {
        top <- top + (stats.table$n[i] - 1) * stats.table$var[i]
    }
    sigma <- top / (N - C)
    sigma
}

sigma <- sigma.calc(stats.table)
print(sigma)
```

    ## [1] 0.1202406

Calculate *τ*<sup>2</sup> (variation between clusters)
------------------------------------------------------

Estimation of *τ*<sup>2</sup> involves a few more steps - in order to obtain the sample estimate for variation between clusters, $\\hat{\\tau}^2$, we must first calculate the weighted between-cluster variance, *S*<sub>*B*</sub><sup>2</sup>:

$$ \\hat{S}\_B^2 = \\frac{\\sum\_{j=1}^C n\_j(\\bar{y}\_j - \\bar{y})^2}{\\tilde{n}(C - 1)}    $$

where

$\\bar{y}\_j$ = mean on response variable for cluster *j*

$\\bar{y}$ = overall mean on response variable

$\\tilde{n} = \\frac{1}{C - 1} \[N - \\frac{\\sum\_{j=1}^C n\_j^2}{N}\]$

We cannot use *S*<sub>*B*</sub><sup>2</sup> as a direct estimate of *τ*<sup>2</sup> because it is impacted by the random variation among subjects within the same clusters. Therefore, in order to remove this random fluctuation, we will estimate the population between-cluster variance as:

$$ \\hat{\\tau}^2 = S\_B^2 - \\frac{\\hat{\\sigma}^2}{\\tilde{n}}    $$

Using this and the stats.table, we can calculate *τ*<sup>2</sup>:

``` r
n.tilde.calc <- function(stats.table) {
    C = nrow(stats.table)
    N = sum(stats.table$n)
    
    summed = 0
    for (i in 1:C) {
        summed = summed + stats.table$n[i]^2
    }
    
    n.tilde = (1 / (C - 1)) * (N - (summed / N))
    n.tilde
}

s.b.calc <- function(stats.table, overall.mean) {
    C = nrow(stats.table)
    n.tilde = n.tilde.calc(stats.table)
    
    top = 0
    for (i in 1:C) {
        temp = stats.table$n[i] * (stats.table$mean[i] - overall.mean)^2
        top = top + temp
    }
    
    s.b = top / (n.tilde * (C - 1))
    s.b
}

tau.calc <- function(stats.table, overall.mean) {
    s.b <- s.b.calc(stats.table, overall.mean)
    n.tilde <- n.tilde.calc(stats.table)
    sigma <- sigma.calc(stats.table)

    tau = s.b - (sigma / n.tilde)
    tau
}

tau = tau.calc(stats.table, mean.Pleasant)

print(tau)
```

    ## [1] 0.07155833

Calculate *ρ*<sub>*I*</sub>
---------------------------

We have now calculated all of the parts we need to estimate *ρ*<sub>*I*</sub> for the population:

$$ \\hat{\\rho}\_I = \\frac{ \\hat{\\tau}^2} {\\hat{\\tau}^2 + \\hat{\\sigma}^2} $$

``` r
rho.calc <- function(stats.table, overall.mean) {
    tau = tau.calc(stats.table, overall.mean)
    sigma = sigma.calc(stats.table)
    
    rho = tau / (tau + sigma)
    rho
}

rho = rho.calc(stats.table, mean.Pleasant)
print(rho)
```

    ## [1] 0.3730903

**Success!!**

I chose to do this calc by hand for practice, but we can check that we got it 'right' by using an existing function. Note, though, that we don't expect the answer to be exactly the same, since the *ρ*<sub>*I*</sub> technically assumes all of the groups are an equal size, and additional modifications are needed to account for this, which aren't covered in this section.

``` r
print(ICC1.lme(Pleasant, SSP, ssid.data))
```

    ## [1] 0.3340836

Seems close enough to me, given the limitations of our method!

*What this does confirm is that approximately 35% of the variation in Pleasantness rating is accounted for by the SSP.*

Repeat for some other variables
-------------------------------

Out of interest, let's take a look at the intraclass correlation (ICC) for Eventfulness, loudness, and overall ratings.

### Eventful

``` r
stats.table.E <- ssid.data %>%                       # specify data frame
                    group_by(SSP) %>%               # specify group indicator
                    summarise(n=n(), 
                              mean=mean(Eventful),
                              var=var(Eventful))   # specify functions

mean.E <- mean(ssid.data$Eventful)

icc.E <- rho.calc(stats.table.E, mean.E)
print(icc.E)
```

    ## [1] 0.1546242

``` r
print(ICC1.lme(Eventful, SSP, ssid.data))
```

    ## [1] 0.1488417

### loudness

``` r
stats.table.L <- ssid.data %>%                       # specify data frame
                    group_by(SSP) %>%               # specify group indicator
                    summarise(n=n(), 
                              mean=mean(loudness),
                              var=var(loudness))   # specify functions

mean.L <- mean(ssid.data$loudness)

icc.L <- rho.calc(stats.table.L, mean.L)
print(icc.L)
```

    ## [1] 0.1121996

``` r
print(ICC1.lme(loudness, SSP, ssid.data))
```

    ## [1] 0.1072711

### overall

``` r
stats.table.O <- ssid.data %>%                       # specify data frame
                    group_by(SSP) %>%               # specify group indicator
                    summarise(n=n(), 
                              mean=mean(overall),
                              var=var(overall))   # specify functions

mean.O <- mean(ssid.data$overall)

icc.O <- rho.calc(stats.table.O, mean.O)
print(icc.O)
```

    ## [1] 0.3300359

``` r
print(ICC1.lme(overall, SSP, ssid.data))
```

    ## [1] 0.2931177

ICC of LocationIDs, as opposed to SSPs
--------------------------------------

For a last look, let's see how the ICC changes when we consider each LocationID as its own group instead of separating it out by SSPs

``` r
stats.table.loc <- ssid.data %>%                       # specify data frame
                    group_by(LocationID) %>%               # specify group indicator
                    summarise(n=n(), 
                              mean=mean(Pleasant),
                              var=var(Pleasant))   # specify functions

mean.loc <- mean(ssid.data$Pleasant)

icc.loc <- rho.calc(stats.table.loc, mean.loc)
print(icc.loc)
```

    ## [1] 0.3643732

``` r
print(ICC1.lme(Pleasant, LocationID, ssid.data))
```

    ## [1] 0.35801

So, we get a very small bit more of the variance explained (according to the actual ICC calc), but not a significant amount, so the SSP method is capturing a good amount of the information!
