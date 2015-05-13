





Aggregate representation of genetic soil horizons via proportional-odds logistic regression
========================================================
transition: none
width: 1024
height: 800
css: custom.css

D.E. Beaudette, P. Rouder, J.M. Skovlin

<br><br><br><br><br><br><br><br>
<span style="color: white; font-size:50%;">This document is based on `aqp` version 1.8-7 and `soilDB` version 1.5-5`.</span>


Describing soil morphology in aggregate is hard
========================================================
![alt text](static-figures/mvo-soil-montage-narrow.jpg)

- hz depths / designations: overlap, consistency, frequency
- style and convention: variation over time and by describer
- transition and infrequent horizons: BA, AB, BCt, etc.
- lumpers vs. spliters: <span style="font-size:75%; font-stretch: condensed;">A-Bt1-Bt2-R</span> vs. <span style="font-size:75%; font-stretch: condensed;">A1-A2-AB-Bt1-Bt2-Bt3-Cr-R</span>

<span class="oneliner">can we do better than selecting a "representative pedon" from a collection?</span>


Aggregation over generalized horizon labels
========================================================
![alt text](static-figures/genhz-sketch.png)


1. determine the core concept: e.g. **A-Bt1-Bt2-Bt3-Cr-R**
2. assess existing data, relevant management or scientific needs
3. assign generalized horizon labels (GHL)
4. aggregate over GHL, in R / AQP syntax:
 - empirical: slice() &#8594;&nbsp; slab() &#8594;&nbsp; probability depth-functions
 - model-based:  slice() &#8594;&nbsp; orm() &#8594;&nbsp; probability model
5. determine most-likely horizonation

<span class="oneliner">generalized horizon labels are expert-guided, "micro-correlation" decisions</span>


========================================================
Examples using 54 profiles correlated to Loafercreek soil series
- fine-loamy, mixed, superactive, thermic ultic haploxeralfs
- extent: foothills of the Sierra Nevada Mountains, MLRA 18
- uses: recreation, range, vineyard, low-density residential
<img src="presentation-figure/plot-sample-data-1.png" title="plot of chunk plot-sample-data" alt="plot of chunk plot-sample-data" style="display: block; margin: auto;" />


assignment of GHL, using expert knowledge
========================================================

<img src="presentation-figure/plot-sample-data-zoom-1.png" title="plot of chunk plot-sample-data-zoom" alt="plot of chunk plot-sample-data-zoom" style="display: block; margin: auto;" />


slice(): resample along 1-cm increments
========================================================

<img src="presentation-figure/slice-data-1-1.png" title="plot of chunk slice-data-1" alt="plot of chunk slice-data-1" style="display: block; margin: auto;" />


slab(): slice-wise probability calculation
========================================================

<img src="presentation-figure/slice-data-2-1.png" title="plot of chunk slice-data-2" alt="plot of chunk slice-data-2" style="display: block; margin: auto;" />

slice() and fit PO-logistic regression model
========================================================

<img src="presentation-figure/slice-and-fit-1-1.png" title="plot of chunk slice-and-fit-1" alt="plot of chunk slice-and-fit-1" style="display: block; margin: auto;" />



GHL probability depth-functions / model
========================================================

<img src="presentation-figure/compare-proportions-1.png" title="plot of chunk compare-proportions" alt="plot of chunk compare-proportions" style="display: block; margin: auto;" />


Quantifying uncertainty
========================================================
<img src="presentation-figure/hmmm-2-1.png" title="plot of chunk hmmm-2" alt="plot of chunk hmmm-2" style="display: block; margin: auto;" />


Most likely horizonation
========================================================
<img src="presentation-figure/plot-ml-hz-1.png" title="plot of chunk plot-ml-hz" alt="plot of chunk plot-ml-hz" style="display: block; margin: auto;" />


Model diagnostics and coefficients
========================================================

```
                        Model Likelihood               Discrimination    Rank Discrim.    
                              Ratio Test                      Indexes          Indexes    
Obs          5996    LR chi2    12104.33    R2                  0.892    rho     0.942    

          Coef     S.E.   Wald Z Pr(>|Z|)
y>=BA      -1.3076 0.1001 -13.06 <0.0001 
y>=Bt1     -2.0056 0.1043 -19.24 <0.0001 
y>=Bt2     -6.1505 0.1911 -32.18 <0.0001 
y>=Bt3     -9.8848 0.2401 -41.17 <0.0001 
y>=Cr     -12.3874 0.2518 -49.20 <0.0001 
y>=R      -15.1682 0.2655 -57.14 <0.0001 
hzdept      0.2244 0.0072  30.97 <0.0001 
hzdept'    -0.2383 0.0370  -6.45 <0.0001 
hzdept''    0.3297 0.1238   2.66 0.0078  
hzdept'''   0.7363 0.3211   2.29 0.0219  
```


Model stability
========================================================
![alt text](static-figures/model-robustness.png)



Ideas to explore
========================================================

1. external validation
2. simulation from a model
3. model stability
4. model limitations (e.g. minimum sample size, ... ?)
5. more realistic estimates of SE (incorporation of correlation structure via GEE)
6. other uses of model coefficients

