# Customer Segmentation and Similarity Search

Behaviour-based customer segmentation on the Online Retail II dataset using RFM features, K-Means clustering, PCA and t-SNE visualisation, similarity search and a decision-tree surrogate for explainability.

## Problem

Treating every customer the same wastes marketing budget on customers who are already loyal or already inactive. The goal of this project is to group customers by how they buy, so that retention, loyalty and win-back actions can be targeted at each group.

## Dataset

**Online Retail II** (UCI Machine Learning Repository): transactions from a UK-based online gift-ware retailer between December 2009 and December 2011.

| Item | Value |
|---|---|
| Raw rows | 1,067,371 |
| Raw columns | 8 |
| Rows after cleaning | 779,425 |
| Customers after cleaning | 5,878 |

| Column | Description |
|---|---|
| Invoice | Invoice number (a leading "C" marks a cancellation) |
| StockCode | Product code |
| Description | Product name |
| Quantity | Units on the line (negative for returns) |
| InvoiceDate | Purchase timestamp |
| Price | Unit price in GBP |
| CustomerID | Customer identifier |
| Country | Customer country |

Each row is one product line on one invoice. The two Excel sheets (2009-2010 and 2010-2011) are combined before cleaning.

## Approach

1. **Cleaning**: removed rows with missing CustomerID, cancelled invoices, non-positive Quantity or Price, and duplicate rows.
2. **Feature engineering** (one row per customer):

| Feature | Definition |
|---|---|
| Recency | Days from last purchase to the snapshot date (last date + 1 day) |
| Frequency | Number of unique invoices |
| TotalSpending | Sum of Quantity x Price |
| AvgOrderValue | TotalSpending divided by Frequency |
| NumDistinctProducts | Number of unique stock codes purchased |
| IsUK | 1 if the customer is in the United Kingdom, else 0 |

3. **Preprocessing**: outliers capped using the IQR rule, then features standardised with `StandardScaler`.
4. **Model comparison**: K-Means, Agglomerative (Ward linkage), Gaussian Mixture Model and DBSCAN.
5. **Evaluation**: Silhouette score, Davies-Bouldin index and Calinski-Harabasz index.
6. **Visualisation**: PCA (2 components, 70.3% of variance retained) and t-SNE (perplexity 30, 4,000-customer sample).
7. **Similarity search**: cosine similarity, Euclidean distance and k-nearest neighbours for finding look-alike customers.
8. **Explainability**: a depth-4 decision tree trained to reproduce the cluster labels.

## Model Comparison

| Model | Silhouette (higher is better) | Davies-Bouldin (lower is better) |
|---|---|---|
| **K-Means (k = 5)** | **0.339** | **1.062** |
| Agglomerative (Ward) | 0.276 | 1.216 |
| Gaussian Mixture | 0.091 | 2.227 |
| DBSCAN | -0.024 | 0.837 |

K-Means also had the highest Calinski-Harabasz score (2,984). DBSCAN produced 15 small clusters with a negative silhouette, so its low Davies-Bouldin value is not meaningful. K-Means was selected as the final model.

## Customer Segments

Average values per segment after outlier capping:

| Segment | Customers | Recency (days) | Frequency | Total spending (GBP) | Avg order value (GBP) | Distinct products |
|---|---|---|---|---|---|---|
| Premium / High-Value | 1,108 | 46.3 | 12.8 | 4,185 | 405 | 172 |
| Moderate / Average | 2,085 | 83.6 | 3.7 | 845 | 231 | 51 |
| High Average-Order, Low-Frequency (1) | 659 | 198.5 | 3.3 | 2,048 | 605 | 70 |
| High Average-Order, Low-Frequency (2) | 492 | 196.5 | 3.7 | 1,821 | 451 | 63 |
| Dormant / At-Risk | 1,534 | 476.1 | 1.8 | 403 | 221 | 27 |

Suggested actions:

| Segment | Action |
|---|---|
| Premium / High-Value | Loyalty rewards, early access, retention focus |
| Moderate / Average | Cross-sell and upsell campaigns |
| High Average-Order, Low-Frequency | Bundles and reminders to increase purchase frequency |
| Dormant / At-Risk | Win-back offers and re-engagement campaigns |

## Explainability

A decision tree (`max_depth=4`, `min_samples_leaf=20`) was trained to predict the K-Means cluster from the six features. It reproduces the cluster labels for 91.6% of customers (measured on the training data), and its feature importances show what separates the segments:

| Feature | Importance |
|---|---|
| Recency | 0.319 |
| TotalSpending | 0.266 |
| IsUK | 0.204 |
| AvgOrderValue | 0.166 |
| Frequency | 0.046 |
| NumDistinctProducts | 0.000 |

## Similarity Search

```python
find_similar_customers(customer_id, how_many=5, method="cosine", same_group_only=False)
```

`method` accepts `"cosine"`, `"euclidean"` or `"knn"`. Setting `same_group_only=True` restricts results to the customer's own segment.

## Limitations

- The Silhouette score of 0.339 indicates moderate cluster separation with overlap between segments, which is typical for behavioural data.
- Decision-tree fidelity is measured on the same data it was trained on.
- IsUK carries notable weight in the clustering and in the surrogate tree.
- Spending outliers are capped, which reduces differences among the highest spenders.
- The data covers 2009 to 2011 and contains no demographic information.

## Tech Stack

Python, pandas, NumPy, scikit-learn, matplotlib, seaborn, Jupyter Notebook

## Getting Started

```bash
git clone https://github.com/vikramxstats/<repository-name>.git
cd <repository-name>
pip install pandas numpy matplotlib seaborn scikit-learn openpyxl jupyter
```

1. Download the Online Retail II dataset from the UCI Machine Learning Repository.
2. Set the file path in the data loading cell of the notebook.
3. Run `Customer_Segmentation.ipynb` from top to bottom.

## Author

**Vikram Singh**
MSc Statistics and Computing, Banaras Hindu University

[LinkedIn](https://www.linkedin.com/in/vikram-singh-dst27) | [GitHub](https://github.com/vikramxstats)
