# Machine Learning Fundamentals: Interview Notes

## 1. Types of learning
| Type | Data | Examples |
|---|---|---|
| Supervised | labelled (X, y) | regression, classification |
| Unsupervised | unlabelled | clustering, dimensionality reduction, anomaly detection |
| Semi-supervised | few labels | pseudo-labelling |
| Self-supervised | labels from the data itself | language model pre-training |
| Reinforcement | reward signals | game agents, retention policies (your JAR project) |

## 2. The ML workflow
Problem framing -> data collection -> EDA -> cleaning and split -> feature engineering -> baseline -> model selection and tuning -> evaluation on held-out data -> error analysis -> deployment -> monitoring and retraining.
**Always build a baseline first** (majority class, linear model, TF-IDF + logistic regression).

## 3. Bias-variance and generalisation
- **Bias:** error from wrong/simplistic assumptions (underfitting). **Variance:** sensitivity to training data (overfitting).
- Total error = bias^2 + variance + irreducible noise.
- Overfitting signs: training score high, validation score low. Fixes: more data, regularisation, simpler model, early stopping, dropout, feature selection, cross-validation, data augmentation.
- Underfitting fixes: more capacity, better features, less regularisation, train longer.

## 4. Data splitting and validation
- Train / validation / test split (e.g. 70/15/15). The **test set is touched once** at the end.
- **k-fold cross-validation**; **stratified** for imbalanced classes; **time-series split** (never shuffle time; train on past, validate on future); **group k-fold** when rows from one entity must stay together.
- **Data leakage:** information from outside the training set sneaks in (scaling or encoding fit on all data before splitting, target-derived features, duplicates across splits, future data). Fit preprocessing **inside** the cross-validation pipeline.
- **Out-of-fold (OOF) predictions** build unbiased meta-features for stacking (your toxic-comment and MCQ projects).

## 5. Metrics
### Classification
| Metric | Formula / meaning | Use |
|---|---|---|
| Accuracy | (TP+TN)/all | balanced classes only |
| Precision | TP/(TP+FP) | cost of false positives high (spam) |
| Recall (sensitivity) | TP/(TP+FN) | cost of false negatives high (disease) |
| F1 | harmonic mean of P and R | balance |
| Macro F1 | average F1 over classes, equal weight | imbalanced multi-class (your 0.826 score) |
| Micro / weighted F1 | pooled / class-weighted | |
| ROC-AUC | ranking quality over thresholds | balanced-ish |
| PR-AUC | precision-recall area | heavy imbalance |
| Log loss | penalises confident wrong probabilities | probabilistic outputs |
| MAP@k | mean average precision at k | ranking (your MCQ top-3 task) |
Confusion matrix: TP, FP, FN, TN. Threshold tuning trades precision and recall. Calibrate probabilities (Platt scaling, isotonic).
### Regression
MAE (robust), MSE/RMSE (penalises large errors), R^2, MAPE/RMSPE, quantile loss.

## 6. Class imbalance
Options: class weights (`class_weight='balanced'`), resampling (random over/under-sampling, **SMOTE**), threshold moving, metrics like PR-AUC/Macro F1, ensembling, collecting more minority data. SMOTE interpolates in feature space and can create unrealistic samples in high-dimensional sparse text, so it often hurts there (the finding in your toxic-comment project). Resample only inside training folds.

## 7. Core algorithms
| Algorithm | Idea | Strengths | Weaknesses |
|---|---|---|---|
| Linear regression | fit a line, minimise squared error | interpretable | assumes linearity |
| Logistic regression | linear model + sigmoid for probabilities | strong baseline, calibrated | linear boundary |
| k-NN | vote of k nearest points | simple, non-parametric | slow at inference, needs scaling |
| Naive Bayes | Bayes with independence assumption | fast, good for text | strong assumption |
| SVM | maximum-margin hyperplane, kernels | good on small/medium, sparse text (LinearSVC) | scaling, kernel cost |
| Decision tree | recursive splits (Gini / entropy) | interpretable | overfits |
| Random forest | bagging of trees + random features | robust, little tuning | big, slower |
| Gradient boosting (XGBoost, LightGBM, CatBoost) | trees fit residuals sequentially | top for tabular data | more tuning, can overfit |
| k-means | iterative centroids | simple clustering | needs k, spherical clusters |
| DBSCAN | density-based clusters | arbitrary shapes, noise | parameters sensitive |
| PCA | project onto top-variance directions | dimensionality reduction | linear |
| Gaussian mixture, hierarchical clustering | probabilistic / dendrogram | | |

### Ensemble methods
- **Bagging:** train on bootstrap samples and average (reduces variance): random forest.
- **Boosting:** sequential models focusing on errors (reduces bias): AdaBoost, GBM, XGBoost.
- **Stacking:** a meta-learner combines base-model predictions (use OOF predictions). **Voting:** hard or soft average.
- Diversity among base models matters more than their number.

## 8. Regularisation and optimisation
- **L2 (Ridge):** penalises squared weights, shrinks. **L1 (Lasso):** penalises absolute weights, produces sparsity (feature selection). **Elastic Net:** both.
- **Gradient descent:** update weights opposite to the gradient; batch, stochastic (SGD), mini-batch; learning rate controls step size; momentum, Adam adapt steps.
- **Convex vs non-convex** losses; local minima matter less in high dimensions than saddle points and plateaus.
- Feature scaling (standardise/normalise) matters for distance-based and gradient-based models, not for trees.

## 9. Feature engineering
Missing values (impute mean/median/mode/model, add "is missing" flag), categorical encoding (one-hot, ordinal, target encoding with care to avoid leakage, hashing), numeric transforms (log, binning, polynomial), date features, text (TF-IDF, embeddings), interaction features, aggregation features, outlier handling, feature selection (filter, wrapper, embedded, permutation importance), dimensionality reduction.

### TF-IDF
TF = term frequency in a document; IDF = log(N / document frequency) down-weights common words. `sublinear_tf` uses `1 + log(tf)`. Word n-grams capture phrases; character n-grams (char_wb) are robust to spelling.

## 10. Hyperparameter tuning
Grid search, random search (often better), Bayesian optimisation (Optuna), successive halving; tune with cross-validation; track experiments (Weights & Biases, MLflow). Avoid tuning on the test set.

## 11. Interpretability
Coefficients, tree feature importance (biased toward high-cardinality), **permutation importance**, **SHAP** (per-prediction contributions), partial dependence plots, LIME. Explain errors, not just metrics.

## 12. Production ML (MLOps basics)
Reproducible pipelines (scikit-learn `Pipeline`, versioned data/code/models), model registry, serving (FastAPI, batch vs online), monitoring (latency, data drift, concept drift, prediction distribution), retraining triggers, A/B and shadow testing, rollback, feature stores. Training-serving skew is a common bug.

## 13. Probability and statistics refreshers
Bayes' theorem, conditional independence, expectation and variance, normal distribution, central limit theorem, hypothesis testing (p-value, confidence interval, type I/II errors), correlation vs causation, A/B test sample size and multiple testing, MLE vs MAP.

## 14. Recommender and time-series (brief)
- Recommenders: collaborative filtering (user-user, item-item, matrix factorisation), content-based, hybrid; cold start; metrics like precision@k, recall@k, NDCG.
- Time series: trend, seasonality, stationarity; lag features, rolling stats, ARIMA, Prophet, gradient boosting with time features (your Solari forecasting project); never leak the future.

## Quick interview Q&A
- **Explain bias-variance.** Section 3.
- **How do you handle imbalanced data?** Section 6.
- **Precision vs recall?** Section 5.
- **Why use Macro F1?** Averages class F1 equally so minority classes count.
- **What is data leakage? Examples?** Section 4.
- **Bagging vs boosting?** Variance reduction by averaging vs bias reduction by sequential correction.
- **Why does gradient boosting beat random forest often?** Sequential error correction with shallow trees and shrinkage.
- **L1 vs L2?** Sparsity vs smooth shrinkage.
- **How do you pick k in k-means?** Elbow, silhouette, domain knowledge.
- **What is cross-validation for?** Reliable estimate of generalisation and model selection.
- **When would you choose logistic regression over a neural net?** Small/tabular data, need interpretability or a strong baseline.
- **How do you know a model is overfitting?** Gap between training and validation scores.
- **What is stacking and how do you avoid leakage in it?** Meta-model on base-model OOF predictions.
