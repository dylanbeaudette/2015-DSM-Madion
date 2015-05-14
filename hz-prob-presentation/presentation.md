





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


Assignment of GHL: expert knowledge + data
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


Quantifying uncertainty
========================================================
![alt text](static-figures/model-robustness.png)


Most likely horizonation
========================================================
<img src="presentation-figure/plot-ml-hz-1.png" title="plot of chunk plot-ml-hz" alt="plot of chunk plot-ml-hz" style="display: block; margin: auto;" />


Conclusions
========================================================
class: smaller
![alt text](static-figures/mvo-soil-montage-extra-narrow.jpg)

1. ML profile, aggregation, overlap -> horizon transitions
2. simulation from model
3. ...
4. remaining issues:
  * model limitations (e.g. minimum sample size, ... ?)
  * more realistic estimates of SE (incorporation of correlation structure via GEE)
  * pedogenic interpreation of model coefficients?


