

# Aggregate Representation of Genetic Soil Horizons

D.E. Beaudette, P. Roudier, J.M. Skovlin

## Abstract

Published soil survey reports typically describe soils in terms of aggregate information: soil properties, interpretations, and limitations that are based on a collection of field-described soil profiles. While aggregate soil properties are readily estimated via standard statistical functions (mean, median, etc.), an aggregate representation of horizonation (e.g. genetic or functional horizon designation and depth) is typically difficult to construct. Variation in horizon designation among different soil scientists and different soil description systems, changes in horizon designation standards over time, variable depths at which horizons occur, and the various uncertainties associated with these are all factors that complicate the process of delivering an aggregate representation of horizonation.
In this paper we propose alternatives to the typical "representative profile"-- e.g. the selection of a single soil profile to represent a collection. Two possible methods for aggregating a collection of soil profiles into synthetic profiles are presented, describing depth-wise probability functions for each horizon. Both methods rely on an expert-guided description of generalized horizon designation (e.g. a subset of horizon designation labels that convey a reasonable "morphologic story") along with associated rules (regular expression patterns) used to correlate field-described to generalized horizon designation. The first method is based on (1-cm interval) slice-wise evaluation of generalized horizon designation; the second is based on a proportional-odds logistic regression model fit to depth-slices. These methods are demonstrated using USDA-NRCS soil survey data (USA), based on genetic horizons, and on S-Map soil survey data (NZ), based on functional horizons.





```r
# load sample data
data(loafercreek, package = "soilDB")
```

## Introduction

Published soil survey reports typically describe soils in terms of *aggregate* information-- soil properties, interpretations, and limitations that are based on a collection of field-described soil profiles. While aggregate soil properties are readily estimated via standard statistical functions (mean, median, etc.), an aggregate representation of *horizonation* (e.g. genetic horizon designation and depth) is typically difficult to construct. Variation in horizon designation "style" among different soil scientists, changes in horizon designation standards over time, variable depths at which genetic horizons occur, and the possible lack of a specific genetic horizon are all factors that complicate the process of delivering an aggregate representation of horizonation.

### Ideas:

* Horizonation as assessed by soil scientists is somewhat subjective, and ther is always an error associated with it. This error is very rarely acknowledged: horizon depths are always considered as "crisp" numbers, while they really are "fuzz" numbers. 
* At this stage of the introduction it would be good to explain how traditional aggregates (mean, etc) only gets you so far. Soil profile is a fundamental object for the understanding of soil processes, so it's important to come up w/ statistical/quantitative aggregate for these. Link this with importance of pedological understanding of soil, and with making a truly useful product out of soil survey data. 

We demonstrate two possible methods for aggregating a collection of soil profiles into "synthetic profiles"; describing depth-wise probability functions for each genetic horizon. Both methods rely on an expert-guided description of generalized horizon designation (e.g. horizon designations that are deemed representative) along with associated rules (regular expression patterns) used to correlate field-described to generalized horizon designation. The first method is based on (1-cm interval) slice-wise evaluation of generalized horizon designation; the second is based on a proportional-odds logistic regression model fit to depth-slices. Specialized classes for soil profile collections and depth-slicing algorithms are implemented in the [aqp](http://cran.at.r-project.org/web/packages/aqp/index.html) package for R [C&G reference here].


### Issues to be resolved
* estimation of confidence intervals via GAMM ? [ideas here](http://www.fromthebottomoftheheap.net/2014/06/16/simultaneous-confidence-intervals-for-derivatives/)

* are we violating any assumptinos of OR logistic regression ? [ideas](http://www.ats.ucla.edu/stat/r/dae/ologit.htm) and [more info](http://www.kenbenoit.net/courses/ME104/ME104_Day8_CatOrd.pdf)

* how can we accomodate the lower effective DF due to massive autocorrelation within "sliced" data? This is possible with generalized least squares `Gls()`, but it is not clear if this can be used for OR-LR.

* does it make more sense to use the original horizons, and then add thickness weights-- or use the sliced data? I like the slicing approach, but it does give unreasonably small standard error estimates for coefficients.

* `rlm()` doesn't seem to require ordered factors, and `slab()` barfs with them.

* see `olm()` for other types of multi-category LR

* slice-wise eval of predicted genhz via [Brier Score](http://en.wikipedia.org/wiki/Brier_score#cite_note-Brier-1), or even better: `verification::rps()` for ranked probability scores



### Sample Data Set

54 soil profiles from the Sierra Foothill region of California were used to demonstrate two approaches for determining aggregate representation of genetic horizon boundaries. These soil profiles represent some of the data that have been "correlated" to the [Loafercreek](https://soilseries.sc.egov.usda.gov/OSD_Docs/L/LOAFERCREEK.html) soil series. These data are included within the **soilDB** package for R.


```r
# discreet colors used to plot horizon probability depth-functions
cols <- c(grey(0.33), "goldenrod4", "orange", "orangered", "chocolate", "green", "blue")

# graphical check: sample 15
par(mar = c(0, 0, 0, 0))
plot(loafercreek[sample(1:length(loafercreek), 15), ], name = "hzname", id.style = "side", cex.names = 0.9)
```

<img src="figure/plot-data.png" title="plot of chunk plot-data" alt="plot of chunk plot-data" style="display: block; margin: auto;" />

## Methods


### Horizon Generalization

Generalized horizon designations represent an expert-guided selection of designations that were consistently observed in the field, and meaningful in terms of theory and management.


```r
# check on existing horizonation
sort(table(loafercreek$hzname), decreasing = TRUE)
```

```
## 
##    A  Bt1  Bt2   Cr  Bt3   Oi    R  Crt   BA 2Bt3 2Bt4  ABt  BCt   Bt  Bt4  CBt  2Cr   2R   Bw    C 
##   53   51   50   30   23   19   18   15    9    4    3    3    3    3    3    3    2    2    2    2 
##   Rt 2BCt 2Bt2  2CB 2Crt   AB   Ad   Ap    B 
##    2    1    1    1    1    1    1    1    1
```

```r
# graphical range in horizon mid-point, sorted by class-wise median depth
loafercreek$mid <- with(horizons(loafercreek), (hzdept + hzdepb)/2)
hz.designation.by.median.depths <- names(sort(tapply(loafercreek$mid, loafercreek$hzname, median)))

bwplot(mid ~ factor(hzname, levels = hz.designation.by.median.depths), data = horizons(loafercreek), 
    ylim = c(155, -5), ylab = "Horizon Mid-Point Depth (cm)", scales = list(y = list(tick.number = 10)), 
    panel = function(...) {
        panel.abline(h = seq(0, 140, by = 10), v = 1:length(hz.designation.by.median.depths), col = grey(0.8), 
            lty = 3)
        panel.bwplot(...)
    })
```

<img src="figure/eval-horizonation.png" title="plot of chunk eval-horizonation" alt="plot of chunk eval-horizonation" style="display: block; margin: auto;" />


Once a set of generalized horizons have been determined (``A, BA, Bt1, Bt2, Bt3, Cr, R``), a corresponding set of regular expression (REGEX) rules were developed to convert field-described designations into generalized designations. This process typically requires expert-guided review of: 1) regional patterns in horizonation style, 2) mophologic property differences by groups of field-described designation, and, 3) patterns with depth. The expert's field experience is thus preserved within the vector of REGEX rules. 


```r
## generalize horizon names using REGEX rules
n <- c("A", "BA", "Bt1", "Bt2", "Bt3", "Cr", "R")
p <- c("^A$|Ad|Ap|^ABt$", "AB$|BA$|Bw", "Bt1$|^Bt$", "^Bt2$", "^Bt3|^Bt4|CBt$|BCt$|2Bt|2CB$|^C$", "Cr", 
    "R")
loafercreek$genhz <- generalize.hz(loafercreek$hzname, n, p)

# check unclassified names
table(loafercreek$genhz, loafercreek$hzname)
```

```
##           
##            2BCt 2Bt2 2Bt3 2Bt4 2CB 2Cr 2Crt 2R  A AB ABt Ad Ap  B BA BCt Bt Bt1 Bt2 Bt3 Bt4 Bw  C
##   A           0    0    0    0   0   0    0  0 53  0   3  1  1  0  0   0  0   0   0   0   0  0  0
##   BA          0    0    0    0   0   0    0  0  0  1   0  0  0  0  9   0  0   0   0   0   0  2  0
##   Bt1         0    0    0    0   0   0    0  0  0  0   0  0  0  0  0   0  3  51   0   0   0  0  0
##   Bt2         0    0    0    0   0   0    0  0  0  0   0  0  0  0  0   0  0   0  50   0   0  0  0
##   Bt3         1    1    4    3   1   0    0  0  0  0   0  0  0  0  0   3  0   0   0  23   3  0  2
##   Cr          0    0    0    0   0   2    1  0  0  0   0  0  0  0  0   0  0   0   0   0   0  0  0
##   R           0    0    0    0   0   0    0  2  0  0   0  0  0  0  0   0  0   0   0   0   0  0  0
##   not-used    0    0    0    0   0   0    0  0  0  0   0  0  0  1  0   0  0   0   0   0   0  0  0
##           
##            CBt Cr Crt Oi  R Rt
##   A          0  0   0  0  0  0
##   BA         0  0   0  0  0  0
##   Bt1        0  0   0  0  0  0
##   Bt2        0  0   0  0  0  0
##   Bt3        3  0   0  0  0  0
##   Cr         0 30  15  0  0  0
##   R          0  0   0  0 18  2
##   not-used   0  0   0 19  0  0
```

```r
# remove non-matching generalized horizon names
loafercreek$genhz[loafercreek$genhz == "not-used"] <- NA
loafercreek$genhz <- factor(loafercreek$genhz)

# keep track of generalized horizon names for later
hz.names <- levels(loafercreek$genhz)

# plot generalized horizons via color, sorted by depth

loafercreek$genhz.soil_color <- cols[match(loafercreek$genhz, hz.names)]
new.order <- order(profileApply(loafercreek, max))
par(mar = c(0, 0, 0, 0))
plot(loafercreek, color = "genhz.soil_color", divide.hz = FALSE, plot.order = new.order)
legend("bottom", legend = hz.names, col = cols, pch = 15, bty = "n", horiz = TRUE, cex = 2)
```

<img src="figure/generalize-hz-names.png" title="plot of chunk generalize-hz-names" alt="plot of chunk generalize-hz-names" style="display: block; margin: auto;" />


### Horizon Aggregation by Slicing

Slicing and slice-wise evaluation of horizon probability described in [AQP: A Toolkit for Soil Scientists](10.1016/j.cageo.2012.10.020).


```r
# slice out color and horzizon name into 1cm intervals: no aggregation
max.depth <- 150
slice.resolution <- 1
slice.vect <- seq(from = 0, to = max.depth, by = slice.resolution)
s <- slice(loafercreek, slice.vect ~ soil_color + genhz)

# convert horizon name to factor
s$genhz <- factor(s$genhz, levels = hz.names)

# check horizon proportions at select slices (in this case the first 5 slices)
sapply(1:5, function(i) {
    round(prop.table(table(s[, i]$genhz)), 3)
})
```

```
##     [,1] [,2]  [,3]  [,4]  [,5]
## A      1    1 0.941 0.778 0.704
## BA     0    0 0.000 0.019 0.074
## Bt1    0    0 0.059 0.204 0.222
## Bt2    0    0 0.000 0.000 0.000
## Bt3    0    0 0.000 0.000 0.000
## Cr     0    0 0.000 0.000 0.000
## R      0    0 0.000 0.000 0.000
```

```r
# graphical check: profiles 1:15, top 25 slices
par(mar = c(0, 0, 0, 0))
plot(s[1:15, 1:25], name = "genhz", id.style = "side", cex.names = 0.9)
```

<img src="figure/slice-data1.png" title="plot of chunk slice-data" alt="plot of chunk slice-data" style="display: block; margin: auto;" />

```r
# plot depth-ranges of generalized horizon slices
bwplot(hzdept ~ genhz, data = horizons(s), ylim = c(155, -5), ylab = "Generalized Horizon Depth (cm)", 
    scales = list(y = list(tick.number = 10)), asp = 1, panel = function(...) {
        panel.abline(h = seq(0, 140, by = 10), v = 1:length(hz.names), col = grey(0.8), lty = 3)
        panel.bwplot(...)
    })
```

<img src="figure/slice-data2.png" title="plot of chunk slice-data" alt="plot of chunk slice-data" style="display: block; margin: auto;" />

```r
# compute slice-wise probability: slice-wise P always sum to 1 BUG: this doesn't work with ordered
# factors
a.1 <- slab(loafercreek, ~genhz, cpm = 1)

# convert to long-format for plotting
a.1.long <- melt(a.1, id.vars = "top", measure.vars = hz.names)
```


### Horizon Aggregation by Proportional-Odds Model

```r
## check assumptions of PO model: pp. 351 in (Harell, 2001)
plot.xmean.ordinaly(genhz ~ hzdept, data = horizons(s))
```

<img src="figure/or-model.png" title="plot of chunk or-model" alt="plot of chunk or-model" style="display: block; margin: auto;" />

```r
# proportional-odds logistics regression: fits well, ignore standard errors using sliced data
# properly weights observations... but creates optimistic SE rcs required when we include depths >
# 100 cm...  should we use penalized PO-LR? see pentrace()
dd <- datadist(horizons(s))
options(datadist = "dd")
(l.genhz <- orm(genhz ~ rcs(hzdept), data = horizons(s), x = TRUE, y = TRUE))
```

```
## 
## Logistic (Proportional Odds) Ordinal Regression Model
## 
## orm(formula = genhz ~ rcs(hzdept), data = horizons(s), x = TRUE, 
##     y = TRUE)
## 
## Frequencies of Responses
## 
##    A   BA  Bt1  Bt2  Bt3   Cr    R 
##  368  134 1005 1115  889  947 1515 
## 
## Frequencies of Missing Values Due to Each Variable
##  genhz hzdept 
##   2181      0 
## 
## 
##                         Model Likelihood               Discrimination    Rank Discrim.    
##                               Ratio Test                      Indexes          Indexes    
## Obs          5973    LR chi2    12046.63    R2                  0.892    rho     0.941    
## Unique Y        7    d.f.              4    g                   7.785                     
## Median Y        5    Pr(> chi2)  <0.0001    gr               2404.699                     
## max |deriv| 0.002    Score chi2 10553.58    |Pr(Y>=median)-0.5| 0.422                     
##                      Pr(> chi2)  <0.0001                                                  
## 
##           Coef     S.E.   Wald Z Pr(>|Z|)
## y>=BA      -1.3081 0.1002 -13.06 <0.0001 
## y>=Bt1     -2.0080 0.1044 -19.24 <0.0001 
## y>=Bt2     -6.1038 0.1906 -32.03 <0.0001 
## y>=Bt3     -9.8494 0.2398 -41.07 <0.0001 
## y>=Cr     -12.3512 0.2515 -49.10 <0.0001 
## y>=R      -15.1321 0.2652 -57.05 <0.0001 
## hzdept      0.2235 0.0072  30.88 <0.0001 
## hzdept'    -0.2359 0.0369  -6.39 <0.0001 
## hzdept''    0.3250 0.1238   2.63 0.0087  
## hzdept'''   0.7375 0.3210   2.30 0.0216
```

```r
# predict along same depths: columns are the class-wise probability fitted.ind --> return all
# probability estimates
p <- data.frame(predict(l.genhz, data.frame(hzdept = slice.vect), type = "fitted.ind"))

# re-name, rms model output give funky names
names(p) <- hz.names

# add depths
p$top <- slice.vect

# melt to long format for plotting
p.long <- melt(p, id.vars = "top", measure.vars = hz.names)
```


## Results


```r
# combine sliced data / predictions
g <- make.groups(sliced.mode.1 = a.1.long, PO.model = p.long)

# remove P(hz) < 1%
g$value[which(g$value < 0.01)] <- NA

# plot all three methods using panels
p.1 <- xyplot(top ~ value | variable, groups = which, data = g, type = "l", ylim = c(155, -5), xlim = c(-0.1, 
    1.2), auto.key = list(columns = 2, points = FALSE, lines = TRUE), as.table = TRUE, par.settings = list(superpose.line = list(lwd = 1, 
    lty = 1, col = c("blue", "black", "red"))), layout = c(7, 1), scales = list(y = list(alternating = 3, 
    tick.number = 10), x = list(alternating = 1)), xlab = "Probability", ylab = "Depth (cm)", strip = strip.custom(bg = grey(0.85)), 
    panel = function(...) {
        panel.abline(h = seq(0, 140, by = 10), v = seq(0, 1, by = 0.2), col = grey(0.8), lty = 3)
        panel.xyplot(...)
    })

p.2 <- xyplot(top ~ value | which, groups = variable, data = g, type = "l", ylim = c(155, -5), xlim = c(-0.1, 
    1.2), auto.key = list(columns = 4, points = FALSE, lines = TRUE), as.table = TRUE, par.settings = list(superpose.line = list(col = cols, 
    lwd = 2, lty = 1)), layout = c(2, 1), scales = list(y = list(alternating = 3, tick.number = 10), 
    x = list(alternating = 1)), xlab = "Probability", ylab = "Depth (cm)", strip = strip.custom(bg = grey(0.85)), 
    asp = 2, panel = function(...) {
        panel.abline(h = seq(0, 140, by = 10), v = seq(0, 1, by = 0.2), col = grey(0.8), lty = 3)
        panel.xyplot(...)
    })

# p.3 <- levelplot(value ~ variable * top | which, data=g, ylim=c(155, -5), col.regions=cols.simple,
# cuts=n.cuts, scales=list(y=list(alternating=3, tick.number=10), x=list(alternating=1)),
# xlab='Probability', ylab='Depth (cm)', strip=strip.custom(bg=grey(0.85)), asp=2,
# panel=function(...) { panel.abline(h=seq(0, 140, by=10), v=1:length(hz.names), col=grey(0.5),
# lty=3) panel.levelplot(...)  })
```


### ML Horizon boundaries


```r
## compute ML horizons by slice note that P(hz) sums to 1 with mode=1 and PO-model

# extract ML-horizon boundaries
a.1.ml <- get.ml.hz(a.1, hz.names)
p.ml <- get.ml.hz(p, hz.names)

# add hz-boundaries by slicing vs. PO model
p.1 + layer(panel.text(x = 1, y = a.1.ml$top[-1], label = expression(symbol("¬")), col = "blue", cex = 1)) + 
    layer(panel.text(x = 1, y = p.ml$top[-1], label = expression(symbol("¬")), col = "red", cex = 1))
```

<img src="figure/ml-hz-boundaries1.png" title="plot of chunk ml-hz-boundaries" alt="plot of chunk ml-hz-boundaries" style="display: block; margin: auto;" />

```r
# plot via groups with lines
print(p.2)
```

<img src="figure/ml-hz-boundaries2.png" title="plot of chunk ml-hz-boundaries" alt="plot of chunk ml-hz-boundaries" style="display: block; margin: auto;" />

```r
# # plot via shading: print(p.3)

# print hz boundary tables
cbind(a.1.ml, p.ml)
```

```
##    hz top bottom confidence pseudo.brier  hz top bottom confidence pseudo.brier
## 1   A   0      7         67    0.1585430   A   0      8         55   0.23941909
## 2 Bt1   7     28         62    0.2345521 Bt1   8     28         64   0.20275770
## 3 Bt2  28     50         59    0.2861600 Bt2  28     49         63   0.23440353
## 4 Bt3  50     64         47    0.4277904 Bt3  49     66         51   0.36994560
## 5  Cr  64     88         54    0.3238298  Cr  66     90         55   0.31719843
## 6   R  88    203         94    0.0362802   R  90    151         90   0.05508596
```




```r
# generate NA-free values/predictions for Brier Score calc
s.sub <- horizons(s)[, c("genhz", "hzdept")]
p.s <- data.frame(predict(l.genhz, s.sub, type = "fitted.ind"))

# re-name, rms model output give funky names
names(p.s) <- hz.names

# combine original data + predictions
p.s <- cbind(s.sub, p.s)

# eval Brier Score by gen hz note that predictions at any given depth slice will always be the same
p.bs <- ddply(p.s, "genhz", function(x.i) {
    # save the gen hz probabilities into new df
    x.pr <- x.i[, hz.names]
    # init new matrix to store most-likely gen hz class
    m <- matrix(0, ncol = ncol(x.pr), nrow = nrow(x.pr))
    # same structure as x.pr
    dimnames(m)[[2]] <- names(x.pr)
    # set appropriate genhz to 1
    for (i in 1:nrow(x.i)) {
        ml.hz.i <- x.i$genhz[i]
        m[i, ml.hz.i] <- 1
    }
    # compute bs for this gen hz
    bs <- sum((x.pr - m)^2, na.rm = TRUE)/nrow(x.pr)
})

# fix names
names(p.bs) <- c("genhz", "brier.score")

# remove NAs from table
p.bs <- na.omit(p.bs)

# larger values -> predictions are less consistently correct
print(p.bs)
```

```
##   genhz brier.score
## 1     A   0.3916243
## 2    BA   1.2260199
## 3   Bt1   0.4324220
## 4   Bt2   0.5061655
## 5   Bt3   0.6701144
## 6    Cr   0.5428294
## 7     R   0.1815343
```

```r
# add fake prob for plotting along x-axis of depth vs. prob
p.s$fake.prob <- 1.1

# make plot of jittered slice-depths vs. fake probability, colored by genhz label
p.3 <- xyplot(jitter(hzdept) ~ jitter(fake.prob, factor = 2), groups = genhz, data = p.s, cex = 0.25, 
    pch = 15, par.settings = list(superpose.symbol = list(col = alpha(cols, 0.5))))

# combine with model output
p.2 + p.3
```

<img src="figure/brier-scores.png" title="plot of chunk brier-scores" alt="plot of chunk brier-scores" style="display: block; margin: auto;" />
 


### Model Stability
PO-model stability was evaluated by repeatedly re-fitting a model to 25 randomly sampled profiles (out of 53 total), 250 times. Predictions from the 250 models were then combined and visualized below. ML horizon boundaries were derived from these data by evaluating the 5th-50th-95th percentiles of ML horizon boundaries computed within each simulation.


```r
# encapsulate model fitting / prediction within a single function
f.test <- function(s.i, slice.vect, hz.names) {
    # fit model to a subset of the original data
    l.genhz <- try(lrm(genhz ~ rcs(hzdept), data = horizons(s.i)))
    
    # sometimes the subset doesn't permit model fitting, in that case return NULL
    if (inherits(l.genhz, "try-error")) 
        return(list(predictions = NULL, model.disc.index = NULL))
    
    # otherwise generate predictions
    p <- data.frame(predict(l.genhz, data.frame(hzdept = slice.vect), type = "fitted.ind"))
    names(p) <- hz.names
    p$top <- slice.vect
    p.long <- melt(p, id.vars = "top", measure.vars = hz.names)
    mdi <- l.genhz$stats["R2"]
    return(list(predictions = p.long, model.disc.index = mdi))
}

# init an empty list
n.reps <- 250
l.res <- list()

## TODO: it would be nice to evaluate the model at each iteration using those observations that were
## left out loop a bunch of times, re-fitting the model to a subset of only 25 profiles
for (i in 1:n.reps) {
    len.vect <- 1:length(loafercreek)
    s.idx <- sample(len.vect, size = 25, replace = FALSE)
    # s.left.out.idx <- setdiff(len.vect, s.idx)
    l.res[[paste("rep.", i, sep = "")]] <- f.test(s[s.idx, ], slice.vect, hz.names)
}

# convert the result from a list into a data.frame, and remove very small probs.
d.res <- ldply(l.res, function(i) i$prediction)
d.res$value[which(d.res$value < 0.01)] <- NA

# extract model discrimination index
res.mdi <- unlist(sapply(l.res, function(i) i$model.disc.index))

# plot with transparency
xyplot(top ~ value | variable, data = d.res, type = "l", ylim = c(155, -5), xlim = c(-0.1, 1.1), auto.key = list(columns = 3, 
    points = FALSE, lines = TRUE), as.table = TRUE, par.settings = list(plot.line = list(lwd = 1, lty = 1, 
    col = rgb(0, 0, 0.75, alpha = 0.05))), layout = c(7, 1), scales = list(y = list(alternating = 3, 
    tick.number = 10), x = list(alternating = 1)), xlab = "Probability", ylab = "Depth (cm)", strip = strip.custom(bg = grey(0.85)), 
    panel = function(...) {
        panel.abline(h = seq(0, 140, by = 10), v = seq(0, 1, by = 0.2), col = grey(0.8), lty = 3)
        panel.xyplot(...)
    })
```

<img src="figure/model-robustness1.png" title="plot of chunk model-robustness" alt="plot of chunk model-robustness" style="display: block; margin: auto;" />

```r
# plot model accuracy: not all models are equal! (in terms of quality)
plot(density(na.omit(res.mdi)), main = "Model Discrimination Index (R2)")
```

<img src="figure/model-robustness2.png" title="plot of chunk model-robustness" alt="plot of chunk model-robustness" style="display: block; margin: auto;" />

```r
# cast to wide format
d.res.wide <- cast(d.res, .id + top ~ variable, value = "value")
d.res.wide$bottom <- d.res.wide$top + 1

# compute ML horizonation, by rep consider weighting by model Dxy
sim.ml <- ddply(d.res.wide, ".id", get.ml.hz, o.names = hz.names)

# aggregate ML horizonation over reps
ddply(sim.ml, "hz", plyr::summarize, top = paste(round(quantile(top, probs = c(0.05, 0.5, 0.95))), collapse = "-"), 
    bottom = paste(round(quantile(bottom, probs = c(0.05, 0.5, 0.95))), collapse = "-"), psuedo.brier = paste(round(quantile(pseudo.brier, 
        probs = c(0.05, 0.5, 0.95)), 3), collapse = "-"))
```

```
##    hz      top      bottom      psuedo.brier
## 1   A    0-0-0       6-8-9 0.191-0.239-0.293
## 2 Bt1    6-8-9    26-28-31 0.164-0.207-0.248
## 3 Bt2 26-28-31    46-49-53 0.188-0.228-0.282
## 4 Bt3 46-49-53    61-66-70 0.286-0.362-0.468
## 5  Cr 61-66-70    85-90-96 0.228-0.322-0.416
## 6   R 85-90-96 151-151-151  0.033-0.05-0.069
```

### Simultaneous confidence intervals for derivatives of splines in GAMs
not yet ready for prime time...



```r
## Load mgcv and fit the model
require(mgcv)

# method setup
ctrl <- list(niterEM = 0, msVerbose = FALSE, optimMethod = "L-BFGS-B")

# fitting model
m2 <- gamm(formula = value ~ s(top), data = subset(g, variable == "A"), correlation = corARMA(form = ~1 | 
    top, p = 2), control = ctrl)

## prediction data
want <- seq(1, nrow(subset(g, variable == "A")), length.out = 200)
pdat <- with(subset(g, variable == "A"), data.frame(top = top[want]))

## download the derivatives gist tmpf <- tempfile()
## download.file('https://gist.githubusercontent.com/gavinsimpson/ca18c9c789ef5237dbc6/raw/295fc5cf7366c831ab166efaee42093a80622fa8/derivSimulCI.R',
## tmpf, method = 'wget') source(tmpf)

library(MASS)

lp <- predict(m2$gam, newdata = pdat, type = "lpmatrix")
coefs <- coef(m2$gam)
vc <- vcov(m2$gam)

set.seed(35)
sim <- mvrnorm(25, mu = coefs, Sigma = vc)

want <- grep("top", colnames(lp))

fits <- lp[, want] %*% t(sim[, want])
dim(fits)  ## 25 columns, 1 per simulation, 200 rows, 1 per evaln point

ylims <- range(fits)
plot(value ~ top, data = subset(g, variable == "A"), pch = 19, ylim = ylims, type = "n")
matlines(pdat$top, fits, col = rgb(0.1, 0.1, 0.1, alpha = 0.25), lty = 1)
```
