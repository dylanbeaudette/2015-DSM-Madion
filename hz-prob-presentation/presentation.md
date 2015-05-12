





Aggregate representation of genetic soil horizons via proportional-odds logistic regression
========================================================
transition: none
width: 1024
height: 800
css: custom.css

D.E. Beaudette, P. Rouder, J.M. Skovlin

<br><br><br><br><br><br><br><br>
<span style="color: white; font-size:50%;">This document is based on `aqp` version 1.8-7 and `soilDB` version 1.5-5`.</span>


Describing soil morphology in aggregate is hard.
========================================================
<span class="oneliner">Can we do better than selecting a "representative pedon" from a collection?</span>
![alt text](static-figures/mvo-soil-montage-narrow.jpg)

- hz depths / designations: overlap, consistency, frequency
- style and convention: variation over time and by describer
- transition and infrequent horizons: BA, AB, BCt, etc.
- lumpers vs. spliters: <span style="font-size:75%; font-stretch: condensed;">A-Bt1-Bt2-R</span> vs. <span style="font-size:75%; font-stretch: condensed;">A1-A2-AB-Bt1-Bt2-Bt3-Cr-R</span>

Yes, aggregation over generalized horizon labels.
========================================================
<span class="oneliner">generalized horizon labels are expert-guided "micro-correlation" decisions</span>
![alt text](static-figures/genhz-sketch.png)


1. determine the core concept: e.g. **A-Bt1-Bt2-Bt3-Cr-R**
2. assess existing data, relevant management or scientific needs
2. aggregate mechanically or by model, in AQP terminology:
 - correlate &#8594;&nbsp; slice() &#8594;&nbsp; slab() &#8594;&nbsp; ML profile
 - correlate &#8594;&nbsp; slice() &#8594;&nbsp; model &#8594;&nbsp; ML profile 



========================================================
Examples using 54 profiles correlated to Loafercreek soil series
- fine-loamy, mixed, superactive, thermic ultic haploxeralfs
- foothills of the Sierra Nevada Mountains, MLRA 18
- recreation, range, vinyards, low-density residential
<img src="presentation-figure/plot-sample-data-1.png" title="plot of chunk plot-sample-data" alt="plot of chunk plot-sample-data" style="display: block; margin: auto;" />

correlate
========================================================

<img src="presentation-figure/plot-sample-data-zoom-1.png" title="plot of chunk plot-sample-data-zoom" alt="plot of chunk plot-sample-data-zoom" style="display: block; margin: auto;" />


slice
========================================================

<img src="presentation-figure/slice-data-1-1.png" title="plot of chunk slice-data-1" alt="plot of chunk slice-data-1" style="display: block; margin: auto;" />


slab
========================================================

<img src="presentation-figure/slice-data-2-1.png" title="plot of chunk slice-data-2" alt="plot of chunk slice-data-2" style="display: block; margin: auto;" />


empirical probability depth-functions
========================================================

<img src="presentation-figure/slab-prob-depth-functions-1.png" title="plot of chunk slab-prob-depth-functions" alt="plot of chunk slab-prob-depth-functions" style="display: block; margin: auto;" />




Proportional-Odds Logistic Regression
========================================================

<img src="presentation-figure/PO-proportions-1.png" title="plot of chunk PO-proportions" alt="plot of chunk PO-proportions" style="display: block; margin: auto;" />


Proportional-Odds Logistic Regression
========================================================

<img src="presentation-figure/compare-proportions-1.png" title="plot of chunk compare-proportions" alt="plot of chunk compare-proportions" style="display: block; margin: auto;" />



========================================================



========================================================




Ideas to explore
========================================================

1. external validation
2. simulation from a model
3. model stability
4. model limitations (e.g. minimum sample size, ... ?)
5. more realistic estimates of SE (incorporation of correlation structure via GEE)
6. other uses of model coefficients

