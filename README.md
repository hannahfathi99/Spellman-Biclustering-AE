# Biclustering Gene Expression Data Using Hierarchical Biclustering and Denoising Autoencoder

![Bioinformatics](https://img.shields.io/badge/Domain-Bioinformatics-green)
![Deep Learning](https://img.shields.io/badge/Method-Denoising%20Autoencoder-blue)
![Machine Learning](https://img.shields.io/badge/ML-Biclustering-orange)
![Python](https://img.shields.io/badge/Python-3.x-yellow)
![TensorFlow](https://img.shields.io/badge/Framework-TensorFlow-red)

---

# Overview

This repository presents an end-to-end computational pipeline for discovering hidden gene expression patterns using **biclustering combined with deep learning-based denoising**.

The project investigates whether a **denoising autoencoder (AE)** can improve the quality and interpretability of biclusters extracted from high-dimensional gene expression data.

The study uses the **Spellman Yeast Cell-Cycle Dataset**, a benchmark biological dataset containing thousands of genes measured across multiple time points.

Unlike conventional clustering approaches that analyze only genes or only samples, biclustering simultaneously identifies:

- groups of genes with similar expression behavior
- groups of time points with coordinated activity

This allows detection of localized gene-time expression modules that may represent biologically relevant expression patterns.

---

# Research Questions

This project investigates the following questions:

1. Can denoising autoencoders improve biclustering quality in gene expression data?
2. Does representation learning reduce noise and increase bicluster coherence?
3. How does AE-based reconstruction affect MSR-based biclustering evaluation?
4. Can deep learning-based preprocessing improve the visualization and interpretation of gene expression structures?

---

# Project Objectives

The main objectives are:

- Analyze and preprocess the Spellman gene expression dataset
- Perform hierarchical biclustering on normalized expression data
- Compare hierarchical biclustering with spectral biclustering
- Develop a denoising autoencoder for noise reduction
- Apply biclustering after AE reconstruction
- Quantitatively compare raw and denoised representations
- Evaluate bicluster quality using MSR, variance, and visualization

---

# Dataset

## Spellman Yeast Cell-Cycle Dataset

Input dataset:

```

data/Spellman.csv

```

Dataset characteristics:

| Property | Value |
|---|---:|
| Number of genes | 4381 |
| Number of time points | 23 |
| Original matrix size | 4381 × 23 |

The dataset contains gene expression measurements collected across the yeast cell cycle.

Matrix representation:

```

Rows    → Genes
Columns → Time points
Values  → Expression levels

```

---

# Methodology Pipeline

The complete workflow:

```

    Spellman Expression Matrix
                  |
                  v
          Data Preprocessing
    (NaN/Inf checking, normalization,
    variance-based filtering)
                  |
                  |
    ------------------------------
    |                            |
    v                            v
Raw Expression            Denoising AE
Biclustering              Reconstruction
    |                            |
    |                            |
    ---------------+--------------
                   |
                   v
          Hierarchical &
        Spectral Biclustering
                   |
                   v
      MSR / Variance Evaluation
                   |
                   v
        Quantitative Comparison

```

---

# Data Preprocessing

## Missing Value Analysis

The dataset was evaluated for numerical issues:

- NaN values
- Infinite values

Results:

```

NaN values: 0
Inf values: 0

```

Therefore, no imputation was required.

---

## Normalization

Row-wise z-score normalization was applied:

$$
x'=\frac{x-\mu}{\sigma}
$$

This standardizes each gene expression profile:

- Mean ≈ 0
- Standard deviation ≈ 1

Normalization improves comparability between genes and stabilizes clustering and neural network training.

---

## Variance-Based Gene Filtering

Low-variance genes often contain limited temporal information.

Filtering strategy:

```

Remove lowest 20% genes based on temporal variance

```

Results:

Before filtering:

```

4381 genes

```

After filtering:

```

3504 genes × 23 time points

```

The filtered normalized matrix was used for all downstream analyses.

---

# Model 1: Raw Data Biclustering

## Hierarchical Biclustering

Implementation:

- Agglomerative clustering
- Ward linkage
- Euclidean distance

Separate clustering was performed for:

- Genes (rows)
- Time points (columns)

---

## Cluster Number Selection

The number of clusters was selected using silhouette analysis.

Final parameters:

```

Number of gene clusters (kr): 3

Number of time-point clusters (kc): 9

```

Therefore:

```

Total biclusters = kr × kc = 27

```

The same parameters were used for all experiments to ensure fair comparison.

---

# Spectral Biclustering

Spectral biclustering was implemented using:

```

sklearn.SpectralBiclustering

```

with bistochastic normalization.

It was used as a baseline comparison against hierarchical biclustering.

---

# Bicluster Evaluation

Each bicluster was evaluated using:

## Mean Squared Residue (MSR)

MSR measures bicluster coherence.

Lower MSR indicates:

- stronger gene-time consistency
- improved bicluster quality

---

## Variance

Variance measures expression variability within biclusters.

A valid bicluster should ideally combine:

- low MSR
- meaningful expression variation

---

## Valid Bicluster Definition

A global MSR threshold was calculated:

```

35th percentile of raw-data bicluster MSR distribution

```

Biclusters below this threshold were considered valid.

---

# Model 2: Denoising Autoencoder

A feed-forward denoising autoencoder was developed to learn compact representations and reduce expression noise.

---

# Autoencoder Architecture

```

Input Layer
23 features

      |
      v

Gaussian Noise
   σ = 0.20

      |
      v

Dense Layer
32 neurons
   ReLU

      |
      v

Latent Representation
   3 neurons

      |
      v

Dense Layer
32 neurons
   ReLU

      |
      v

Output Layer
23 features
   Linear

```

---

# Autoencoder Training Configuration

| Parameter | Value |
|-|-|
| Optimizer | Adam |
| Learning rate | 0.001 |
| Loss | Mean Squared Error |
| Batch size | 64 |
| Maximum epochs | 300 |
| Early stopping patience | 15 |
| Latent dimension | 3 |

The bottleneck representation forces the network to retain dominant expression patterns while reducing high-frequency noise.

---

# Experimental Results

## Model 1: Raw Expression Data

| Method | Total Biclusters | Valid Biclusters | Mean MSR | Mean Variance |
|-|-:|-:|-:|-:|
| Hierarchical | 27 | 8 | 0.0569 | 1.0330 |
| Spectral | 27 | 11 | 0.0452 | 1.0294 |

Raw biclustering detected meaningful structures, but reordered heatmaps showed diffuse boundaries and higher noise levels.

---

# Model 2: AE-Denoised Expression Data

After autoencoder reconstruction:

| Method | Total Biclusters | Valid Biclusters | Mean MSR | Mean Variance |
|-|-:|-:|-:|-:|
| Hierarchical | 27 | 27 | 0.0347 | 0.3417 |
| Spectral | 27 | 27 | 0.0342 | 0.3696 |

---

# Impact of Autoencoder Denoising

## Valid Bicluster Improvement

### Hierarchical Biclustering

```

Raw Data:
8 / 27 valid biclusters

AE-Denoised:
27 / 27 valid biclusters

```

### Spectral Biclustering

```

Raw Data:
11 / 27 valid biclusters

AE-Denoised:
27 / 27 valid biclusters

```

---

## MSR Improvement

Lower MSR indicates improved bicluster coherence.

### Hierarchical

```

0.0569 → 0.0347

```

### Spectral

```

0.0452 → 0.0342

```

The AE reconstruction reduced noise and improved the consistency of gene-time expression patterns.

---

# Visualization

All generated figures are available in:

```

figures/

```


---

# Repository Structure

```

Spellman-Biclustering-AE/

│
├── data/
│   └── Spellman.csv
│
├── notebooks/
│   └── Spellman_Biclustering_AE.ipynb
│
├── figures/
│
├── reports/
│   └── Spellman_Biclustering_AE_report.pdf
│
├── requirements.txt
├── LICENSE
└── README.md

````

---

# Installation and Usage

Clone repository:

```bash
git clone https://github.com/hannahfathi99/Spellman-Biclustering-AE.git
````

Install dependencies:

```bash
pip install -r requirements.txt
```

Run the notebook:

```bash
jupyter notebook notebooks/Spellman_Biclustering_AE.ipynb
```

---

# Technologies

Implemented with:

* Python
* NumPy
* Pandas
* Scikit-learn
* TensorFlow / Keras
* Matplotlib
* Seaborn
* Jupyter Notebook

---

# Limitations

Current limitations include:

* No gene ontology enrichment analysis was performed.
* Only one latent dimension configuration was evaluated.
* The autoencoder does not explicitly model temporal dependencies.
* Biological validation using known cell-cycle pathways remains future work.

---

# Future Work

Potential extensions:

* Gene ontology (GO) enrichment analysis
* Integration of biological networks
* Graph Neural Network-based representation learning
* Variational Autoencoders
* Temporal deep learning architectures
* Multi-omics integration

---

# Author

## Hannah Fathi

**M.Sc. Artificial Intelligence Student**

Research Interests:

* Bioinformatics
* Graph Neural Networks
* Deep Learning
* Computer Vision
* Machine Learning

---

# Conclusion

This project demonstrates a hybrid computational approach combining biclustering and deep learning-based denoising for gene expression analysis.

The denoising autoencoder improved bicluster coherence by reducing MSR values, increasing the number of valid biclusters, and producing clearer reordered expression structures.

The results suggest that learned representations can provide an effective preprocessing strategy for discovering structured patterns in high-dimensional biological datasets.

This repository provides a reproducible framework for exploring the intersection of:

**Artificial Intelligence × Deep Learning × Computational Biology**
