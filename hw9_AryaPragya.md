Homework 9
================
Pragya Arya
4/18/2022

-   [Research question](#research-question)
-   [Variables](#variables)
-   [Variable Summary](#variable-summary)
-   [Model](#model)
    -   [Priors](#priors)
-   [Results](#results)
-   [Convergence Check](#convergence-check)
-   [Posterior distribution of key
    parameters](#posterior-distribution-of-key-parameters)
-   [Interpretation](#interpretation)

## Research question

Does the influence of social rewards (i.e., likes on Twitter) on
tweeting frenquency vary as a function of habit strength?

## Variables

-   `tdiff_pmcs`: Tweet frequency- Time difference between a user’s
    tweet and their immediately preceding tweet (person-mean centered
    and scaled)
-   `likes_24hours_pmcs`: Social reward- Number of likes a user received
    in the past 24 hours (person-mean centered and scaled)
-   `likes_24hours_pms`: Person-mean number of likes received in the
    past 24 hours (scaled)
-   `avg_day_cs`: Habit strength - Average number of a user’s tweets per
    day (centered and scaled)

## Variable Summary

``` r
datasummary_skim(x %>% select(tdiff_pmcs, likes_24hours_pmcs, likes_24hours_pms, avg_day_cs))
```

    ## Warning in datasummary_skim_numeric(data, output = output, fmt = fmt, histogram
    ## = histogram, : The histogram argument is only supported for (a) output types
    ## "default", "html", or "kableExtra"; (b) writing to file paths with extensions
    ## ".html", ".jpg", or ".png"; and (c) Rmarkdown or knitr documents compiled to PDF
    ## or HTML. Use `histogram=FALSE` to silence this warning.

|                      | Unique (\#) | Missing (%) | Mean |  SD |  Min | Median |  Max |
|:---------------------|------------:|------------:|-----:|----:|-----:|-------:|-----:|
| tdiff\_pmcs          |       13173 |           0 |  0.0 | 1.0 | -4.4 |   -0.1 |  3.7 |
| likes\_24hours\_pmcs |        1579 |           2 |  0.0 | 1.0 | -2.7 |   -0.3 | 19.0 |
| likes\_24hours\_pms  |         181 |           0 |  0.2 | 1.0 |  0.0 |    0.0 |  7.4 |
| avg\_day\_cs         |         209 |           0 |  0.0 | 1.0 | -1.6 |   -0.2 |  2.0 |

## Model

Let *Y* = tdiff\_pmcs  
*l**i**k**e**s* = likes\_24hours\_pmcs  
*a**v**g*\_*l**i**k**e**s* = likes\_24hours\_pms  
*h**a**b**i**t* = avg\_day\_cs

*y*<sub>*i**j*</sub> = *β*<sub>0*j*</sub> + *β*<sub>1*j*</sub>*l**i**k**e**s*<sub>*i**j*</sub> + *e*<sub>*i**j*</sub>

*β*<sub>0*j*</sub> = *γ*<sub>00</sub> + *γ*<sub>01</sub>*h**a**b**i**t*<sub>*j*</sub> + *γ*<sub>02</sub>*a**v**g*\_*l**i**k**e**s* + *μ*<sub>0*j*</sub>

*β*<sub>1*j*</sub> = *γ*<sub>10</sub> + *γ*<sub>11</sub>*h**a**b**i**t*<sub>*j*</sub> + *μ*<sub>1*j*</sub>

### Priors

*γ*<sub>00</sub> ∼ *N*(0, 1)

*γ*<sub>01</sub> ∼ *N*(0, 1)

*γ*<sub>02</sub> ∼ *N*(0, 1)

*γ*<sub>10</sub> ∼ *N*(0, 1)

*γ*<sub>11</sub> ∼ *N*(0, 1)

*e*<sub>*i**j*</sub> ∼ *t*<sub>4</sub><sup>+</sup>(0, 3)

*μ*<sub>0*j*</sub> ∼ *t*<sub>4</sub><sup>+</sup>(0, 3)

*μ*<sub>1*j*</sub> ∼ *t*<sub>4</sub><sup>+</sup>(0, 3)

## Results

``` r
m1 <- brm(tdiff_pmcs ~ likes_24hours_pmcs * avg_day_cs + likes_24hours_pms + (1 | subject),
          prior = c(
            prior(normal(0, 1), class = 'Intercept'),
            prior(normal(0, 1), class = 'b'),
            prior(student_t(4, 0, 3), class = 'sd'),
            prior(student_t(4, 0, 3), class = 'sigma')
          ),
          data = x, family = gaussian(link = "identity"),
          cores = numcor, seed = 1,
          file = 'Twitter Main Analysis.rds')
```

## Convergence Check

The trace plots and rank histograms below suggest satisfactory
convergence.

``` r
mcmc_trace(m1, pars = c('b_likes_24hours_pmcs', 'b_avg_day_cs', 'b_likes_24hours_pms',
                         'b_likes_24hours_pmcs:avg_day_cs'))
```

![](hw9_AryaPragya_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
mcmc_rank_hist(m1, pars = c('b_likes_24hours_pmcs', 'b_avg_day_cs', 'b_likes_24hours_pms',
                         'b_likes_24hours_pmcs:avg_day_cs'))
```

![](hw9_AryaPragya_files/figure-gfm/unnamed-chunk-6-2.png)<!-- -->

## Posterior distribution of key parameters

``` r
sum_m1 <- as_draws_df(m1) %>%
  summarize_draws() %>%
  filter(variable %in% c('b_intercept',
                         'b_likes_24hours_pmcs', 'b_avg_day_cs', 'b_likes_24hours_pms',
                         'b_likes_24hours_pmcs:avg_day_cs'))

sum_m1 %>% 
  knitr::kable(digits = 3)
```

<table>
<thead>
<tr>
<th style="text-align:left;">
variable
</th>
<th style="text-align:right;">
mean
</th>
<th style="text-align:right;">
median
</th>
<th style="text-align:right;">
sd
</th>
<th style="text-align:right;">
mad
</th>
<th style="text-align:right;">
q5
</th>
<th style="text-align:right;">
q95
</th>
<th style="text-align:right;">
rhat
</th>
<th style="text-align:right;">
ess\_bulk
</th>
<th style="text-align:right;">
ess\_tail
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
b\_likes\_24hours\_pmcs
</td>
<td style="text-align:right;">
0.054
</td>
<td style="text-align:right;">
0.054
</td>
<td style="text-align:right;">
0.008
</td>
<td style="text-align:right;">
0.009
</td>
<td style="text-align:right;">
0.040
</td>
<td style="text-align:right;">
0.067
</td>
<td style="text-align:right;">
1.002
</td>
<td style="text-align:right;">
8606.480
</td>
<td style="text-align:right;">
2781.351
</td>
</tr>
<tr>
<td style="text-align:left;">
b\_avg\_day\_cs
</td>
<td style="text-align:right;">
0.000
</td>
<td style="text-align:right;">
0.000
</td>
<td style="text-align:right;">
0.009
</td>
<td style="text-align:right;">
0.009
</td>
<td style="text-align:right;">
-0.015
</td>
<td style="text-align:right;">
0.014
</td>
<td style="text-align:right;">
1.003
</td>
<td style="text-align:right;">
10290.682
</td>
<td style="text-align:right;">
3125.299
</td>
</tr>
<tr>
<td style="text-align:left;">
b\_likes\_24hours\_pms
</td>
<td style="text-align:right;">
0.000
</td>
<td style="text-align:right;">
0.000
</td>
<td style="text-align:right;">
0.009
</td>
<td style="text-align:right;">
0.009
</td>
<td style="text-align:right;">
-0.014
</td>
<td style="text-align:right;">
0.014
</td>
<td style="text-align:right;">
1.000
</td>
<td style="text-align:right;">
7960.863
</td>
<td style="text-align:right;">
2813.838
</td>
</tr>
<tr>
<td style="text-align:left;">
b\_likes\_24hours\_pmcs:avg\_day\_cs
</td>
<td style="text-align:right;">
0.011
</td>
<td style="text-align:right;">
0.011
</td>
<td style="text-align:right;">
0.008
</td>
<td style="text-align:right;">
0.008
</td>
<td style="text-align:right;">
-0.002
</td>
<td style="text-align:right;">
0.025
</td>
<td style="text-align:right;">
1.003
</td>
<td style="text-align:right;">
9845.246
</td>
<td style="text-align:right;">
2832.427
</td>
</tr>
</tbody>
</table>

## Interpretation

Based on the results in the brms model above, we do not see an
interaction effect between reward and habit strength in predicting tweet
frequency.
