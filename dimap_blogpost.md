# Nonlinear Distribution Mapping

Nonlinear Distribution Mapping (NoDim) is a method to bring two scales measuring the same underlying quantity into agreement, by matching their respective distributions. It is described in detail in _Properzi et al. 2019_. 

## Idea
- estimate the underlying distrubion of both measures using Gaussian Mixture Models (GMMs)
- estimate the CDF of both distributions
- compute a mapping between both distributions

> **Note**: When assuming both distributions to be gaussian, the first two steps boil essentially down to compute $\mu$ and $\sigma$ for both measures. Same as we would do when z-scoring.

## Assumptions
1. The rank order of the two measures is preserved
2. Both measures are sampled from the same population

> **Note:** We would make these assumptions as well when we are using the z-scoring method.

3. Both measures describe the same underlying property, just at different scales, e.g.: temperature at scales farenheit and Celsius.  

> **Warning:** This is quite a strong assumtions, especially when used in biological systems. Remember, all we do is using statistics to align distributions. Statistics doesn't care about biology, so make sure that your assumptions are well motivated.

## Background

### Probability Density Function (PDF)
The probability density function (PDF) is a function that describes the relative likelihood for a random variable to take on a given value. It is defined as the derivative of the cumulative distribution function (CDF).

$PDF(x) = \frac{dCDF(x)}{dx}$

In case of a Gaussian distribution, the PDF is given by:

$PDF(x) = \frac{1}{\sqrt(2*pi*\sigma^2)} * exp(-(x-\mu)^2/(2*\sigma^2))$

### Cumulative Distribution Function (CDF)
The cumulative distribution function (CDF) is the probability that a random variable is less than or equal to a given value. It is defined as the integral of the PDF.

$CDF(x) = \int_{-\inf}^x PDF(x) dx$

The relationship between CDF and PDF is visualized in the following figure:

![CDF and PDF](../results/figures/normal_dist.jpg)

## Approach

1. approximate CDF. This can be done empiraclly or approximate. The first approach derives the CDF from the data itself. The second however, uses assumptions about parameters ($\mu, \sigma$) to get a theoretical approximation.  

2. Resample the CDFs with respect to to the other distribution, So that for each set of probability values for x, we get a corresponding probability value for y'. 

3. Compute a mapping between y and y' to derive a mapping. 

4. Use interpolation to derive values for the mapping between A and B. 

## Implementation

### Simulation of two measures
This follows the simulation described in [here](simulations.md). Two measures are derived by sampling 'subjects' from the same uniform distribtion. Scores are simulated using a linear model with two different slopes and intercepts. The slope for the first measure is set to -0.02 and the slope for the second measure is set to -0.01. The intercept for the first measure is set to 4 and the intercept for the second measure is set to 2. 

### Distributions of the two measures
The distributions are approximated to be Gaussian. Mean and STD are both estimated from the data. From the estimated parameters, PDF and CDF are estimated.

### Resampling
The CDFs are resampled equidistant between $\mu \pm 4 * \sigma^2$ for each distribution. The approximated CDFs are then computed at these points. 

### Mapping to probability
The resampled points are then used to interpolate between CDF and an input x. This is done to get a corespondance between CDF(x1) and x1. Now this interpolation is evaluated at points between 0 and 1, to get a mapping between CDF1(xp) and CDF2(xp). Both CDF1(xp) and CDF2(xp) can be plotted against each other to get a mapping between the two distributions. Again, the computed values are used to create an interpolation operator to get a mapping between the two distributions, that can be evaluated at any given point.

Long story short. You have two CDfs estimated from the data. You evaluate CDF_BPND at value x1, you take CDF_BPND(x1) = CDF_BPP(x2). And now you know what value x2 would corespond to x1 in the BPP distribution. This is illustrated in the following figure:

![](../results/figures/sim_nodim_mapping.png)

### Result
I chose to run two simulations, once with different populations (age-ranges), to see how different age-range would effect the mapping, and once with the excact same population to see if the mapping can result in exact results when transforming between two measures.

The result of the simulation using the exact same population as input for binding simulation is shown in the following figure:

![](../results/figures/sim_nodim_same_population.png)

The result of the simulation using different populations as input for binding simulation is shown in the following figure:
![](../results/figures/sim_nodim_different_population.png)

The first subfigure in both panels shows the simulated binding vs age. The second the underlying distributions plus mapped distributions, and the third bpnd values that are transformed to bpp values.

From figure one it is evident that if the underlying data is from  the exact same population, we derive at the exact result after mapping. 

From figure two it is evident that if the underlying data is from different populations, we derive at a different result after mapping. Subfigure three looks almost the same as we would see after z-scoring. This is not surprising, since both are relying on $\mu$ and $\sigma$ to map the distributions.

### Solution
However, the overal solution would be to draw matched samples from both datasets. Since we are going to map Cimbi36 to Altanserin, we would only need a subset of the Altanserin data to match the Cimbi36 data. Therefor we compute compute mean and std for Altanserin data that matches the age of the samples in the Cimbi36 data.

## Conclusion
The Nonlinear Distribution Mapping (NoDim) method is a method to bring two scales measuring the same underlying quantity into agreement, by matching there respective distributions. It works both ways, as illustrated above. In the end, it is not so different from z-scoring, since both methods are relying on $\mu$ and $\sigma$ of both tracers. However, NoDim allows to only use a subset of Altanserin data to match the Cimbi36 data to compute the respective parameters.  

## References

_Properzi, et al. (2019)_. Nonlinear Distributional Mapping (NoDiM) for harmonization across amyloid-PET radiotracers. NeuroImage, [https://doi.org/10.1016/j.neuroimage.2018.11.019](https://doi.org/10.1016/j.neuroimage.2018.11.019)

