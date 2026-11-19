# scikit-learn: Interview Notes and Cheat Sheet

## 1. The API in one view
Every estimator follows: `fit(X, y)`, then `predict(X)` or `transform(X)`.
- **Estimator:** `fit`. **Predictor:** `predict`, `predict_proba`, `score`. **Transformer:** `fit`, `transform`, `fit_transform`.
- Data shape: `X` is `(n_samples, n_features)`, `y` is `(n_samples,)`.
- Fitted attributes end with an underscore (`coef_`, `classes_`, `feature_importances_`).

```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report

X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)
clf = LogisticRegression(max_iter=1000, class_weight="balanced").fit(X_tr, y_tr)
print(classification_report(y_te, clf.predict(X_te)))
```

## 2. Pipelines and ColumnTransformer (avoid leakage)
```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.ensemble import RandomForestClassifier

pre = ColumnTransformer([
    ("num", Pipeline([("imp", SimpleImputer(strategy="median")), ("sc", StandardScaler())]), num_cols),
    ("cat", Pipeline([("imp", SimpleImputer(strategy="most_frequent")), ("oh", OneHotEncoder(handle_unknown="ignore"))]), cat_cols),
])
model = Pipeline([("pre", pre), ("rf", RandomForestClassifier(n_estimators=300, n_jobs=-1, random_state=0))])
model.fit(X_tr, y_tr)
```
A pipeline fits preprocessing only on the training fold during cross-validation, so there is no leakage, and it makes the whole model one deployable object (`joblib.dump(model, "model.pkl")`).

## 3. Cross-validation and tuning
```python
from sklearn.model_selection import StratifiedKFold, cross_val_score, GridSearchCV, RandomizedSearchCV

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=0)
scores = cross_val_score(model, X, y, cv=cv, scoring="f1_macro")

search = RandomizedSearchCV(model, {"rf__max_depth": [None, 10, 20], "rf__n_estimators": [200, 400]},
                            n_iter=6, cv=cv, scoring="f1_macro", n_jobs=-1)
search.fit(X_tr, y_tr); search.best_params_, search.best_score_
```
Splitters: `KFold`, `StratifiedKFold`, `GroupKFold`, `TimeSeriesSplit`, `LeaveOneOut`, `ShuffleSplit`. For time data use `TimeSeriesSplit` only.

## 4. Preprocessing toolbox
| Task | Tool |
|---|---|
| Scale | `StandardScaler`, `MinMaxScaler`, `RobustScaler` (outliers) |
| Encode categories | `OneHotEncoder`, `OrdinalEncoder`, `TargetEncoder` (newer versions) |
| Impute | `SimpleImputer`, `KNNImputer`, `IterativeImputer` |
| Text | `CountVectorizer`, `TfidfVectorizer(ngram_range=(1,2), sublinear_tf=True, max_features=...)`, `HashingVectorizer` |
| Features | `PolynomialFeatures`, `FunctionTransformer`, `KBinsDiscretizer` |
| Selection | `SelectKBest`, `RFE`, `SelectFromModel`, `VarianceThreshold` |
| Reduction | `PCA`, `TruncatedSVD` (sparse), `TSNE`, `UMAP` (separate package) |

## 5. Models you must know how to call
```python
from sklearn.linear_model import LinearRegression, Ridge, Lasso, LogisticRegression, SGDClassifier
from sklearn.svm import SVC, LinearSVC
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier, HistGradientBoostingClassifier, StackingClassifier, VotingClassifier
from sklearn.neighbors import KNeighborsClassifier
from sklearn.naive_bayes import MultinomialNB
from sklearn.cluster import KMeans, DBSCAN
```
- `HistGradientBoosting*` is fast on large data and handles missing values natively.
- `LinearSVC` and `SGDClassifier` scale to high-dimensional sparse text.
- `CalibratedClassifierCV(LinearSVC())` adds probabilities.
- Key params: `C` (inverse regularisation in SVM/LogReg), `alpha` (Ridge/Lasso), `max_depth`, `min_samples_leaf`, `n_estimators`, `learning_rate`, `class_weight`.

### Stacking (as in your toxic-comment project)
```python
from sklearn.ensemble import StackingClassifier
stack = StackingClassifier(
    estimators=[("svm", CalibratedClassifierCV(LinearSVC(C=0.1, class_weight="balanced"))),
                ("lr", LogisticRegression(C=1.0, class_weight="balanced", max_iter=2000)),
                ("sgd", SGDClassifier(loss="log_loss", alpha=1e-5, class_weight="balanced"))],
    final_estimator=LogisticRegression(), cv=5, stack_method="predict_proba", n_jobs=-1)
```
`cv=` makes the meta-learner train on out-of-fold predictions (no leakage).

## 6. Metrics API
```python
from sklearn.metrics import (accuracy_score, precision_recall_fscore_support, f1_score, roc_auc_score,
                             average_precision_score, confusion_matrix, classification_report,
                             mean_squared_error, mean_absolute_error, r2_score, make_scorer)
f1_score(y_te, pred, average="macro")
roc_auc_score(y_te, proba[:, 1]); roc_auc_score(y_te, proba, multi_class="ovr")
mean_squared_error(y, p, squared=False)      # RMSE (newer: root_mean_squared_error)
```
`scoring=` strings: `accuracy`, `f1_macro`, `roc_auc`, `neg_mean_squared_error` (negative so higher is better).

## 7. Imbalanced data helpers
`class_weight="balanced"`; `imblearn` (`SMOTE`, `RandomUnderSampler`, `Pipeline` from imblearn so resampling happens only on training folds); `sample_weight` in `fit`; threshold tuning via `precision_recall_curve`.

## 8. Clustering and dimensionality reduction
```python
KMeans(n_clusters=4, n_init="auto", random_state=0).fit(X_scaled)
silhouette_score(X_scaled, labels); inertia_ (elbow)
PCA(n_components=0.95).fit(X_scaled); explained_variance_ratio_
```
Scale before k-means and PCA.

## 9. Interpretation
`model.feature_importances_` (trees), `coef_` (linear), `sklearn.inspection.permutation_importance`, `PartialDependenceDisplay`; SHAP for detailed explanations.

## 10. Persistence and serving
```python
import joblib; joblib.dump(pipeline, "model.joblib"); pipeline = joblib.load("model.joblib")
```
Serve with FastAPI (`/health`, `/predict`), load the model once at startup, validate input with Pydantic, log inputs and latency, version the artifact. Do not unpickle untrusted files.

## 11. Common pitfalls
- Fitting scalers/encoders/vectorisers on the whole dataset before splitting (leakage).
- Using accuracy on imbalanced data.
- Forgetting `stratify=y`, `random_state`, or shuffling time series.
- Mixing train/test preprocessing; not saving the preprocessing with the model.
- Tuning hyperparameters on the test set.
- Trusting tree feature importance for correlated or high-cardinality features.
- High-cardinality one-hot explosion (use hashing, target encoding or embeddings).

## Quick interview Q&A
- **`fit` vs `transform` vs `fit_transform`?** Learn parameters; apply them; both in one call. Call `fit_transform` on train only, `transform` on validation/test.
- **Why use a Pipeline?** Prevent leakage, bundle preprocessing and model, simplify CV and deployment.
- **GridSearch vs RandomizedSearch?** Exhaustive vs sampled; random is more efficient in large spaces.
- **How do you handle categorical features?** Encode (one-hot, ordinal, target); use tree models that handle them natively when available.
- **How do you evaluate a model on imbalanced data?** F1/PR-AUC, confusion matrix, class weights.
- **Random forest vs gradient boosting parameters?** RF: n_estimators, max_features, depth; GB: learning_rate, n_estimators, depth, subsample.
- **How do you save and serve a model?** `joblib` + FastAPI.
- **How do you choose between LinearSVC and LogisticRegression for text?** Both are strong; LinearSVC is often best on accuracy/F1, LogReg gives probabilities.
