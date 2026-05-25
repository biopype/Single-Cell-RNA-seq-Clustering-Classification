# Single-Cell RNA-seq Clustering Classification

A machine learning project for classifying single-cell RNA-seq (scRNA-seq) data using traditional ML and deep learning approaches. This project compares the performance of KNN, Random Forest, and a custom neural network on the PBMC3K dataset.

##  Overview

This project demonstrates a complete machine learning pipeline for single-cell RNA-seq data classification:

1. **Data Loading & Preprocessing** - Loading the PBMC3K dataset and performing standard scRNA-seq preprocessing (normalization, log transformation, HVG selection)
2. **Dimensionality Reduction** - PCA-based feature extraction reducing 2,000 highly variable genes to 100 components
3. **Model Comparison** - Training and evaluating three classification models: KNN, Random Forest, and a deep neural network
4. **Performance Evaluation** - Comprehensive metrics including accuracy, F1-score, and per-class analysis

##  Dataset

**PBMC3K (Peripheral Blood Mononuclear Cells)**

- **Total Cells**: 2,700
- **Total Genes**: 32,738
- **Gene Features Used**: 2,000 (highly variable genes)
- **PCA Components**: 100
- **Explained Variance**: 42.6%
- **Number of Cell Types/Clusters**: 9
- **Train/Test Split**: 80/20 (2,160 training, 540 test samples)

The dataset contains heterogeneous immune cell populations including T cells, B cells, NK cells, monocytes, and others identified through unsupervised Leiden clustering.

##  Project Structure

```
├── scproj01final.ipynb          # Main notebook with complete analysis
├── README.md                     # This file
└── results/                      # Output visualizations
    ├── accuracy_comparison.png       # Model accuracy comparison chart
    ├── f1_comparison.png            # Model F1 score comparison chart
    ├── confusion_matrix.png         # Deep learning confusion matrix
    ├── training_curves.png          # Training loss and validation accuracy curves
    └── per_class_metrics.png        # Per-class precision, recall, F1 scores
```

##  Methodology

### Data Preprocessing

```python
1. Normalization      → CPM normalization (counts per million)
2. Log Transform      → log1p transformation
3. HVG Selection      → Top 2,000 highly variable genes
4. PCA                → Reduce to 100 components
5. Feature Scaling    → StandardScaler normalization
```

### Train/Test Split Strategy

- **Stratified split**: Maintains class proportions (9 clusters)
- **Random state**: Set to 42 for reproducibility
- **Training set**: 2,160 samples (80%)
- **Test set**: 540 samples (20%)
  
##  Results

### Model Performance Comparison

| Model | Accuracy | F1 Score (Weighted) | Parameters |
|-------|----------|-------------------|-----------|
| **KNN (k=5)** | 56.67% | 0.534 | N/A |
| **Random Forest** | 92.41% | 0.922 | 100 trees |
| **Deep Learning** | 91.85% | 0.917 | 218,889 |

### Per-Class Performance (Deep Learning Model)

| Cluster | Precision | Recall | F1 Score | Support |
|---------|-----------|--------|----------|---------|
| 8 | 1.000 | 1.000 | 1.000 | 3 |
| 4 | 0.986 | 1.000 | 0.993 | 70 |
| 2 | 0.960 | 0.990 | 0.975 | 98 |
| 5 | 0.970 | 0.970 | 0.970 | 33 |
| 6 | 0.967 | 0.935 | 0.951 | 31 |
| 3 | 0.901 | 0.889 | 0.895 | 72 |
| 0 | 0.864 | 0.919 | 0.891 | 124 |
| 1 | 0.872 | 0.820 | 0.845 | 100 |
| 7 | 1.000 | 0.556 | 0.714 | 9 |

### Training Progress (Deep Learning)

| Epoch | Loss | Validation Accuracy |
|-------|------|-------------------|
| 10 | 0.0631 | 92.41% |
| 20 | 0.0287 | 92.78% |
| 30 | 0.0136 | 92.59% |
| 40 | 0.0152 | 92.22% |
| 50 | 0.0073 | 91.85% |

##  Installation

### Requirements

```bash
Python 3.7+
```

### Dependencies

Install required packages:

```bash
pip install scanpy anndata torch umap-learn seaborn leidenalg \
            scikit-learn matplotlib pandas numpy
```

### Package Versions (Recommended)

- `scanpy` - Single-cell analysis
- `anndata` - Annotated data structures
- `torch` - Deep learning framework
- `scikit-learn` - Classical ML models
- `umap-learn` - Dimensionality reduction
- `seaborn` - Statistical visualization
- `leidenalg` - Community detection for clustering

##  Usage

### Running the Notebook

```bash
jupyter notebook scproj01final.ipynb
```

The notebook is organized into sequential cells that can be run from top to bottom:

1. **Installation & Imports** - Install dependencies and import libraries
2. **Data Loading** - Load PBMC3K dataset via scanpy
3. **Preprocessing** - Normalization, HVG selection, PCA
4. **KNN Baseline** - Train and evaluate k-NN classifier
5. **Random Forest** - Train and evaluate Random Forest
6. **Deep Learning Setup** - Define neural network architecture
7. **Training** - Train the deep learning model for 50 epochs
8. **Evaluation** - Compare all three models
9. **Visualization** - Generate performance plots and confusion matrix

### Key Sections

**Load and Preprocess Data**
```python
adata = sc.datasets.pbmc3k()
sc.pp.normalize_total(adata, target_sum=1e4)
sc.pp.log1p(adata)
sc.pp.highly_variable_genes(adata, n_top_genes=2000)
```

**Train Models**
```python
# KNN
knn = KNeighborsClassifier(n_neighbors=5)
knn.fit(X_train_scaled, y_train)

# Random Forest
rf = RandomForestClassifier(n_estimators=100, random_state=42)
rf.fit(X_train_scaled, y_train)

# Deep Learning (see Model Details section)
model = CellClassifier(input_dim=100, num_classes=9)
# Train for 50 epochs...
```

**Evaluate Models**
```python
results_df = pd.DataFrame({
    "Model": ["KNN", "Random Forest", "Deep Learning"],
    "Accuracy": [acc_knn, acc_rf, acc_dl],
    "F1 Score": [f1_knn, f1_rf, f1_dl]
})
```

##  Model Details

### 1. K-Nearest Neighbors (KNN)

- **Hyperparameters**: k=5 neighbors
- **Performance**: 56.67% accuracy
- **Characteristics**: Fast inference, no training phase, struggles with high-dimensional data

### 2. Random Forest

- **Architecture**: 100 decision trees
- **Max Depth**: Unlimited
- **Performance**: 92.41% accuracy, 92.21% F1 score
- **Characteristics**: Excellent feature importance analysis, handles non-linear relationships well

### 3. Deep Neural Network (CellClassifier)

**Architecture**:
```
Input Layer:        100 features
Hidden Layer 1:     512 units → BatchNorm → ReLU → Dropout(0.3)
Hidden Layer 2:     256 units → BatchNorm → ReLU → Dropout(0.3)
Hidden Layer 3:     128 units → BatchNorm → ReLU → Dropout(0.2)
Output Layer:       9 units (softmax via CrossEntropyLoss)
```

**Total Parameters**: 218,889

**Training Configuration**:
- **Optimizer**: Adam (lr=0.001)
- **Loss Function**: CrossEntropyLoss
- **Batch Size**: 64
- **Epochs**: 50
- **Device**: CPU
- **Regularization**: BatchNorm + Dropout layers

**Performance**: 91.85% accuracy, 91.73% F1 score

##  Key Findings

### 1. Model Performance Hierarchy

**Random Forest > Deep Learning > KNN**

The Random Forest model achieved the highest test accuracy (92.41%), marginally outperforming the deep neural network (91.85%). Both significantly outperform the KNN baseline (56.67%).

### 2. Dimensionality Reduction Effectiveness

- Original data: 2,000 features (highly variable genes)
- After PCA: 100 features
- **Variance explained**: 42.6%
- Despite aggressive dimensionality reduction, classification performance remained excellent (>91%)

### 3. Per-Class Analysis

**Strongest Performance**:
- **Cluster 8**: Perfect classification (F1=1.0, n=3)
- **Cluster 4**: Near-perfect (F1=0.99, n=70)
- **Cluster 2**: Strong (F1=0.97, n=98)

**Challenging Cases**:
- **Cluster 7**: Lower recall (55.6%), small sample size (n=9)
- **Cluster 1**: Moderate performance (F1=0.85, n=100)

### 4. Training Dynamics

- **Rapid convergence**: Loss decreased from 0.0631 (epoch 10) to 0.0073 (epoch 50)
- **Validation accuracy peaked** at epoch 20 (92.78%)
- **Minimal overfitting**: Slight accuracy decline after epoch 20 suggests regularization is working effectively

### 5. Class Imbalance Effects

- Largest class (Cluster 0): 124 samples
- Smallest class (Cluster 8): 3 samples
- Model handles imbalance well, maintaining high precision across underrepresented classes

##  Visualizations Generated

1. **accuracy_comparison.png** - Bar chart comparing model accuracies
2. **f1_comparison.png** - Bar chart comparing F1 scores
3. **confusion_matrix.png** - 9×9 heatmap showing per-class predictions (deep learning)
4. **training_curves.png** - Dual-axis plot of training loss and validation accuracy
5. **per_class_metrics.png** - Per-class precision, recall, and F1 scores

##  Future Improvements

1. **Hyperparameter Optimization** - Grid/random search for deep learning model
2. **Data Augmentation** - Synthetic oversampling for minority classes
3. **Feature Selection** - Identify most informative PCA components
4. **Cross-Validation** - Implement k-fold for more robust evaluation
5. **Ensemble Methods** - Stack multiple models for improved performance
6. **Cell Type Annotation** - Map clusters to known immune cell types
7. **Visualization** - UMAP/tSNE plots of cell clusters
8. **Biological Validation** - Compare with known marker genes

##  References

- **Scanpy Documentation**: https://scanpy.readthedocs.io/
- **PBMC3K Dataset**: Originally from Zheng et al., 2017
- **PyTorch Documentation**: https://pytorch.org/
- **Scikit-learn**: https://scikit-learn.org/

##  Author

Created as a single-cell bioinformatics and machine learning demonstration project.

##  License

This project is provided as-is for educational and research purposes.

##  Notes

- All random seeds are fixed (random_state=42) for reproducibility
- The project uses CPU-based training; GPU acceleration can be enabled by modifying device setup
- Confusion matrix and per-class metrics are specifically for the deep learning model
- Model comparisons are on the same test set (stratified 20% holdout) for fair evaluation
