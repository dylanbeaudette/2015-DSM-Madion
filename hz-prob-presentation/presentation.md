





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



Quick detour: definitions
========================================================
class: smaller

## proportional-odds (PO) logistic regression
$$P[Y \geq j | X] = \frac{1}{1 + exp[-(\alpha_{j} + X \beta]} $$
Extension of logistic regression model; predictions constrained by horizon designation and order. RCS basis functions accommodate non-linearity.

## Shannon Entropy (H index)
$$ H = -\sum_{i=1}^{n}{p_{i} * ln(p_{i})}  $$
$H$ is an index of uncertainty associated with predicted probabilities, $\mathbf{p}$, of encountering horizons $i$ through $n$ at some depth. Larger values suggest **more** confusion.

## Brier scores
$$ B = \frac{1}{n} \sum_{i=1}^{n}{ ( p_{i} - y_{i} )^{2}  }  $$
$B$ is an index of agreement between predicted probabilities, $\mathbf{p}$, and horizons, $\mathbf{y}$, over depth-slices $i$ through $n$ associated with a specific horizon. Larger values suggest **less** agreement between probabilities and observed horizon labels.


========================================================
class: smaller

Examples using 54 profiles correlated to Loafercreek soil series

- fine-loamy, mixed, super-active, thermic ultic haploxeralfs
- extent: foothills of the Sierra Nevada Mountains, MLRA 18
- uses: recreation, range, vineyard, low-density residential
<img src="presentation-figure/plot-sample-data-1.png" title="plot of chunk plot-sample-data" alt="plot of chunk plot-sample-data" style="display: block; margin: auto;" />

<span class="oneliner">colors represent generalized horizon labels (GHL)</span>



Assignment of GHL: expert knowledge + data
========================================================

<img src="presentation-figure/plot-sample-data-zoom-1.png" title="plot of chunk plot-sample-data-zoom" alt="plot of chunk plot-sample-data-zoom" style="display: block; margin: auto;" />

<span class="oneliner">colors represent generalized horizon labels (GHL)</span>


slice(): resample along 1-cm increments
========================================================

<img src="presentation-figure/slice-data-1-1.png" title="plot of chunk slice-data-1" alt="plot of chunk slice-data-1" style="display: block; margin: auto;" />

<span class="oneliner">colors represent generalized horizon labels (GHL)</span>


slab(): slice-wise probability calculation
========================================================

<img src="presentation-figure/slice-data-2-1.png" title="plot of chunk slice-data-2" alt="plot of chunk slice-data-2" style="display: block; margin: auto;" />

<span class="oneliner">results are interpretable and directly tied to the original data, but over-fit</span>


slice() and fit PO-logistic regression model
========================================================

<img src="presentation-figure/slice-and-fit-1-1.png" title="plot of chunk slice-and-fit-1" alt="plot of chunk slice-and-fit-1" style="display: block; margin: auto;" />

<span class="oneliner">proportional-odds logistic regression generalizes the process</span>


GHL probability depth-functions / model
========================================================

<img src="presentation-figure/compare-proportions-1.png" title="plot of chunk compare-proportions" alt="plot of chunk compare-proportions" style="display: block; margin: auto;" />



Quantifying uncertainty
========================================================
<img src="presentation-figure/shannon-1.png" title="plot of chunk shannon" alt="plot of chunk shannon" style="display: block; margin: auto;" />


Quantifying uncertainty
========================================================
![alt text](static-figures/model-robustness.png)


Most likely horizonation
========================================================
<img src="presentation-figure/plot-ml-hz-1.png" title="plot of chunk plot-ml-hz" alt="plot of chunk plot-ml-hz" style="display: block; margin: auto;" />


Conclusions: aggregate soil morphology
========================================================
class: smaller
left: 40%

<img src="presentation-figure/ml-hz-conclusions-1.png" title="plot of chunk ml-hz-conclusions" alt="plot of chunk ml-hz-conclusions" style="display: block; margin: auto;" />

***

- fact: sampling by **genetic horizon** will continue to be important
- we can do better than picking a single, **representative profile**
- soil series **defined** by GHL rules, PO-LR model, and properties aggregated by GHL
- variability between descriptions **smoothed** as sample size increases-- *given thoughtful correlation*
- continuous **depth-functions** of genetic, or diagnostic horizons
- most-likely horizonation, based on depth-function crossings
- quantitative estimates of uncertainty: Brier scores, Shannon Entropy, etc.



Conclusions: further work
========================================================
class: smaller

![alt text](static-figures/mvo-soil-montage-extra-narrow.jpg)

- simulation of likely profile "sketches" from model
- minimum sample sizes, model diagnostics, etc.
- more realistic estimates of SE, e.g. correlation structure via GEE
- pedogenic interpretation of model coefficients
- management of GHL (mico-correlation decisions)
- ???

<br><br>
<center>
Thank You!
<hr>
http://aqp.r-forge.r-project.org
</center>
