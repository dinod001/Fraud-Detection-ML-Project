# Model Training Analysis - Fraud Detection

## ✅ **DATA LEAKAGE CHECK: NO LEAKAGE DETECTED**

Your preprocessing pipeline is **CORRECT**:
- ✅ Train-test split is done **FIRST** (before any preprocessing)
- ✅ Preprocessor is **fit only on training data**
- ✅ Test data is **only transformed** (never fit)
- ✅ TargetEncoder correctly uses only training labels
- ✅ Test set remains imbalanced (as it should)

**Conclusion: No data leakage issues found!**

---

## ⚠️ **CRITICAL ISSUE: DOUBLE SMOTE APPLICATION**

### Problem Identified:
1. **Notebook 05** (`05_encoding_and_scaling.ipynb`): SMOTE is applied to training data before saving
2. **Notebook 06** (`06_model_training.ipynb`): SMOTE is applied **AGAIN** in the pipeline during cross-validation

This means SMOTE is being applied **TWICE**:
- Once globally to the entire training set
- Once again inside each CV fold

### Why This is a Problem:
- **Overfitting risk**: The model sees artificially balanced data twice
- **Unrealistic CV scores**: Cross-validation scores may be inflated
- **Incorrect data distribution**: The training data doesn't represent the real distribution

### Solution:
**Remove SMOTE from Notebook 05** and keep it **only in the pipeline** (Notebook 06). This ensures:
- SMOTE is applied correctly within each CV fold
- No data leakage between folds
- More realistic performance estimates

---

## 📊 **CLASS IMBALANCE ANALYSIS**

### Test Set Distribution:
- **Class 0 (Non-Fraud)**: 27,393 samples (94.9%)
- **Class 1 (Fraud)**: 1,462 samples (5.1%)
- **Ratio**: ~18.7:1 (Highly imbalanced)

### Performance Metrics Analysis:

| Model | Accuracy | Precision | Recall | F1-Score |
|-------|----------|-----------|--------|----------|
| Logistic Regression | 0.8951 | **0.2277** | 0.4480 | 0.3020 |
| Decision Tree | 0.8937 | **0.1927** | 0.3440 | 0.2471 |
| Random Forest | 0.8865 | **0.2095** | 0.4473 | 0.2853 |

### Why Low Precision?

**YES, the imbalanced test set is a major factor**, but there are additional issues:

1. **Class Imbalance Impact**:
   - With 18.7:1 ratio, even a small false positive rate creates many false positives
   - Example: If model predicts 2% of non-fraud as fraud → 548 false positives
   - With only 1,462 true fraud cases, this dramatically lowers precision

2. **Model Calibration Issues**:
   - Models may be too aggressive in predicting fraud (high recall, low precision)
   - Need to adjust decision threshold (currently using default 0.5)

3. **Double SMOTE Effect**:
   - The double SMOTE application may have caused models to overfit to balanced data
   - When tested on imbalanced data, performance degrades

---

## 🎯 **RECOMMENDATIONS**

### 1. **Fix Double SMOTE** (CRITICAL)
- Remove SMOTE from Notebook 05
- Keep SMOTE only in the pipeline (Notebook 06)

### 2. **Threshold Tuning** (IMPORTANT)
Instead of using default 0.5 threshold, optimize for your business needs:
- **For fraud detection**: Usually want higher precision (fewer false alarms)
- Use Precision-Recall curve to find optimal threshold
- Consider cost of false positives vs false negatives

### 3. **Use Appropriate Metrics**
For imbalanced data, focus on:
- **Precision-Recall Curve** (better than ROC for imbalanced data)
- **F1-Score** (you're already using this - good!)
- **Precision at specific recall levels** (e.g., "What's precision if we catch 80% of fraud?")

### 4. **Consider Additional Techniques**
- **Class weights**: Use `class_weight='balanced'` in models
- **Cost-sensitive learning**: Penalize false negatives more
- **Ensemble methods**: Combine multiple models
- **Anomaly detection**: Consider isolation forests or autoencoders

### 5. **Model Selection**
Based on your results:
- **Random Forest** has best balance (F1: 0.2853, Recall: 0.4473)
- Consider tuning threshold specifically for Random Forest
- Try XGBoost (you imported it but didn't use it)

---

## 📈 **EXPECTED IMPROVEMENTS AFTER FIXES**

After fixing double SMOTE and tuning thresholds:
- **Precision**: Should improve to 0.40-0.60 range
- **F1-Score**: Should improve to 0.40-0.50 range
- **More realistic CV scores**: Will better match test performance

---

## ✅ **SUMMARY**

1. **Data Leakage**: ✅ None detected - your pipeline is correct
2. **Double SMOTE**: ❌ **CRITICAL ISSUE** - needs immediate fix
3. **Class Imbalance**: ✅ Yes, it's a major factor, but fixable with threshold tuning
4. **Model Training**: ⚠️ Mostly correct, but double SMOTE needs fixing

**Next Steps**:
1. Fix double SMOTE issue
2. Retrain models
3. Tune decision thresholds
4. Re-evaluate performance

