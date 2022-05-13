---
title: 'musclesyneRgies'
tags:
  - electromyography
  - NMF
  - dimensionality reduction
  - neurophysiology
  - biomechanics
authors:
  - name: Alessandro Santuz
    orcid: 0000-0002-6577-5101
    affiliation: "1, 2, 3" 
affiliations:
 - name: Department of Training and Movement Sciences, Humboldt-Universität zu Berlin, Berlin, Germany
   index: 1
 - name: Berlin School of Movement Science, Humboldt-Universität zu Berlin, Berlin, Germany
   index: 2
 - name: Institute for Biomechanics, ETH Zurich, Zurich, Switzerland
   index: 3
date: 10 February 2022
bibliography: paper.bib
output:
  html_document:
    keep_md: yes
---

# Summary

A

# Statement of need

The great amount of muscles and joints in the body of vertebrate animals makes the problem of motor control a high-dimensional one: while producing and controlling movement, the central nervous system is constantly dealing with an over-abundant number of degrees of freedom. Amongst the existing theories that attempt to describe the coordination of movements, one proposed by Nikolai Bernstein [@Bernstein1967] assumes that the central nervous system can simplify the production of movements by implementing orchestrated, synergistic activations of functionally related muscle groups rather than by sending commands to each muscle individually. With the end of the twentieth century and the advent of modern computational tools, the first rigorous mathematical models of muscle synergies came to life, based on linear decomposition [@Lee1999] of electromyographic (EMG) data [@Tresch1999]. In the past two decades, several different approaches have been used to model muscle synergies as low-dimensional sets of muscle activations and non-negative matrix factorization (NMF) has often proved to be one of the most reliable and widely employed [@Rabbi2020]. Yet, little consensus exists on what the best practices to preprocess EMG data are, which NMF algorithm should be used, what convergence criteria should be adopted and so on [@Devarajan2014; @Oliveira2014; @Santuz2017]. Researchers with little to none coding experience will find in the R package `musclesyneRgies` a complete framework for the preprocessing, factorization and visualization of EMG data, with sensible defaults deriving from peer-reviewed studies on the topic. More advanced users will find `musclesyneRgies` to be fully customizable, depending on the specifics of the study design (e.g. the considered biological system, the motor task, the measurement devices used, etc.). `musclesyneRgies` aims at filling the existing gap of tools available to researchers of all levels in fields that deal with the analysis of vertebrate movement control such as neuroscience, biomechanics, biomedical engineering or sport science.

# Typical workflow

The typical workflow when using `musclesyneRgies` consists of five main steps:

1. Data preparation
2. Raw data processing
3. Synergy extraction
4. Synergy classification
5. Plots.

Using the native pipe operator (R >= `4.1.0` is required), a typical analysis pipeline can be synthetically written as follows:

```r
SYNS_classified <- lapply(RAW_DATA, filtEMG) |>       # Filter raw data
  lapply(function(x) normEMG(x, cycle_div = 100)) |>  # Time-normalization
  lapply(synsNMF) |>                                  # Synergy extraction
  classify_kmeans()                                   # Synergy classification
```

Extensive documentation is available on [GitHub](https://github.com/alesantuz/musclesyneRgies) and the [Comprehensive `R` Archive Network](https://CRAN.R-project.org/package=musclesyneRgies). Below, a typical analysis workflow is provided as an example.

## Data preparation

The data set must be a list of objects of class `EMG`, each of which is a list with two elements:

- `cycles` data frame containing cycle timings, with as many columns as many cycle subdivisions are wanted (column names are not needed but can be present)
- `emg` data frame containing raw EMG data in columns, with the first column being time information in the same units as in `cycles` (column names must be present and reflect the muscle names).

Those two elements should look similar to the following:

```r
library(musclesyneRgies)
data("RAW_DATA")
head(RAW_DATA[[1]]$cycles)
```

```
##      V1    V2
## 1 1.414 2.074
## 2 2.448 3.115
## 3 3.488 4.141
## 4 4.515 5.168
## 5 5.549 6.216
## 6 6.596 7.249
```

```r
head(RAW_DATA[[1]]$emg[, 1:6])
```

```
##    time         ME        MA       FL        RF        VM
## 1 0.014   0.201416 -6.445313 22.65930 -0.100708 -0.906372
## 2 0.015  -2.316284 -0.100708 24.16992  1.812744 -1.913452
## 3 0.016  -7.351685 -7.150269 23.46497  0.704956 -5.337524
## 4 0.017  -5.538940 -3.222656 27.49329  5.236816 -4.330444
## 5 0.018 -10.675049 -5.740356 23.16284 -0.704956  2.014160
## 6 0.019 -12.487793 -3.927612 19.94019  2.014160 -5.136108
```

It is also possible to read directly from ASCII files such as tab-separated txt or comma-separated csv and then use the following function to automatically create the list of objects of class `EMG` needed for the subsequent steps using the function `rawdata`.

## Raw data processing

Raw EMG is commonly filtered before factorization. If the length of the data set needs reduction, the function `subsetEMG` can help. If no subsetting is needed, raw data can be filtered done as follows:


```r
filtered_EMG <- lapply(
  RAW_DATA,
  function(x) {
    filtEMG(x)
  }
)
```

Defaults can be overridden by specifying the arguments as in the follosing example:


```r
another_filtered_EMG <- lapply(
  RAW_DATA,
  function(x) {
    filtEMG(x,
      demean = FALSE,
      rectif = "halfwave",
      HPf = 30,
      HPo = 2,
      LPf = 10,
      LPo = 2,
      min_sub = FALSE,
      ampl_norm = FALSE
    )
  }
)
```
The filtered EMG can be time-normalized. In the following example, the filtered EMG are time-normalized including only three cycles and trimming first and last to remove unwanted filtering effects. Each cycle is divided into two parts, each normalised to a length of 100 points:

```r
norm_EMG <- lapply(
  filtered_EMG,
  function(x) {
    normEMG(x,
      trim = TRUE,
      cy_max = 3,
      cycle_div = c(100, 100)
    )
  }
)
```

## Synergy extraction

Muscle synergies can then be extracted via NMF using:

```r
SYNS <- lapply(norm_EMG, synsNMF)
```
As all other functions, also `synsNMF` can be tweaked at need by specifying the relevant parameters in the arguments.

## Synergy classification

Since NMF does not return muscle synergies in any specific functional order, a reordering is needed:

```r
SYNS_classified <- classify_kmeans(SYNS)
```

## Plots

At all stages, it is possible to obtain plots of the considered data sets, from the raw EMG and until the classified synergies.

# Availability
The latest development version of `musclesyneRgies` is freely available on [GitHub](https://github.com/alesantuz/musclesyneRgies). A stable release is freely available via the [Comprehensive `R` Archive Network](https://CRAN.R-project.org/package=musclesyneRgies). Documentation and examples
are contained in each version's manual pages, vignettes and readme file. To install the latest development version, `devtools` needs to be installed beforehand and then `musclesyneRgies` can be installed directly from GitHub with the following:
```r
install.packages("devtools")
library(devtools)
install_github("alesantuz/musclesyneRgies")
```
The latest stable release appearing on CRAN can be installed with:
```r
install.packages("musclesyneRgies")
```

# Acknowledgments

The author is grateful, for their many contributions, to (in alphabetical order): Turgay Akay, Adamantios Arampatzis, Leon Brüll, Antonis Ekizos, Lukas Hauser, Lars Janshen, Victor Munoz-Martel, Dimitris Patikas, Arno Schroll. An up-to-date list of contributors is available on [GitHub](https://github.com/alesantuz/musclesyneRgies/graphs/contributors).

# References