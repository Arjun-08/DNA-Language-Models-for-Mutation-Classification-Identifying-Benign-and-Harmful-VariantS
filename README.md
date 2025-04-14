
#  DNA Mutation Classification (Benign vs. Pathogenic variants)

This repository contains **three distinct deep learning approaches** to classify human single-nucleotide variants (SNVs) from ClinVar as either **Benign (0)** or **Pathogenic (1)** using the GRCh38 reference genome. All models are implemented in PyTorch, with transformers trained using masked language modeling (MLM) and fine-tuning.

---

##  Dataset Overview

| Source | Details |
|--------|---------|
| **ClinVar VCF (GRCh38)** | ~5.5M total variants |
| **Used in this project** | Filtered 5000 SNVs (Pathogenic or Benign only) |
| **Reference genome** | [GCA_000001405.15_GRCh38](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/001/405/...) |
| **Window size** | ±50 bp (CNN models), ±128 bp (Transformer model) |
| **Context length** | 101 bp (CNNs), 256 tokens (GPN-style Transformer) |

We limit to **5000 SNVs** for practical training in Colab (speed + memory) and to test models efficiently under real-world constraints.

---

##  Model Implementations

###  Model 1: CNN on One-Hot Encoded ALT Sequences

- Uses only **alternate allele sequence** (after mutation inserted).
- Input: 101bp one-hot encoded sequence.
- Architecture: 2-layer CNN with dense classifier.
- Loss: `BCELoss`
- Optimizer: Adam

 **Performance**:
```
Train Accuracy: 95.7%
Test Accuracy: 77.6%
ROC AUC: 0.7497
F1 Score (Pathogenic): 0.4766
```

 

---

###  Model 2: CNN on Paired REF+ALT Sequences + Ensemble (with ΔLL)

- Concatenates both **reference** and **alternate** sequences → [4 x 202]
- Trained a CNN on this paired representation
- Supports optional delta log-likelihood input from external models (e.g., GPN, ESM)
- Combines CNN + ΔLL using logistic regression (ensemble)

 **Best Performance (Ensemble)**:
```
CNN Test Accuracy: 81.3%
ROC AUC: 0.7635
F1 (Pathogenic): 0.4776

Ensemble Accuracy: 82.3%
Ensemble AUC: 0.7828
```



---

###  Model 3: Transformer-Based MLM + Classification (GPN-Style)

- Implements **masked language model (MLM)** pretraining (like GPN)
- Custom vocabulary: `"ACGT-?"`, with `?` as `[MASK]`
- Uses **TransformerEncoder** with 4 layers, 8 heads
- Fine-tunes a classifier head on pooled embedding

 **Pretraining Logs** (MLM on masked sequences from 5000 SNVs):
```
Epoch 1 - MLM Loss: 0.2680
Epoch 2 - MLM Loss: 0.2228
Epoch 3 - MLM Loss: 0.2214
```

 **Fine-tuning Logs** (Classifier on same dataset):
```
Epoch 1 - Loss: 0.4837, Accuracy: 81.67%
Epoch 2 - Loss: 0.4757, Accuracy: 82.05%
Epoch 3 - Loss: 0.4739, Accuracy: 82.05%
```

 **Evaluation (on test set)**:
```
Final Accuracy: 82.00%
Classification Report:
    Benign    → Precision: 0.82, Recall: 1.00
    Pathogenic → Precision: 0.00, Recall: 0.00 (class imbalance issue)
```



---

##  Comparison Table

| Model | Input | Uses REF+ALT? | Pretrained? | Accuracy | AUC | F1 (Pathogenic) |
|-------|-------|----------------|-------------|----------|-----|-----------------|
| CNN (ALT only) | 101 bp (ALT) | ❌ | ❌ | 77.6% | 0.7497 | 0.4766 |
| CNN (REF+ALT) | 101+101 bp | ✅ | ❌ | 81.3% | 0.7635 | 0.4776 |
| + Ensemble (ΔLL) | + ΔLL score | ✅ | ✅ | 82.3% | 0.7828 | 0.4242 |
| Transformer (MLM+CLS) | 256 bp | ✅ | ✅ | 82.0% | ~ | 0.00 (imbalanced) |

**The low F1 score is due to class imbalance, since benign variants dominate, the model rarely predicts pathogenic ones, leading to poor precision and recall for that class.**
---

##  How to Run

### 1. Install dependencies

```bash
pip install biopython pandas cyvcf2 tqdm scikit-learn matplotlib transformers datasets
```

### 2. Run models

- `model1_alt_cnn.ipynb` — Basic CNN on ALT only
- `model2_pair_ref_alt_ensemble.py` — CNN on paired REF/ALT + ensemble
- `model3_transformer_mlm_cls.ipynb` — MLM pretraining + Transformer classifier

---

##  Visualizations 
(please refer PPT or codes for these)

Each model includes:
- Training/validation loss
- ROC AUC and PR curves
- Confusion matrices
- Ensemble performance (Model 2)
- MLM loss curves (Model 3)

---

## Why These Models?

| Purpose | Choice |
|--------|--------|
| Baseline | Model 1 (simple CNN) |
| Realistic biological modeling | Model 2 (both REF and ALT) |
| Language-model style DNA representation | Model 3 (MLM + Transformer) |
| Speed & interpretability | CNNs |
| Context-rich & scalable | Transformers |

---
## Citation

Benegas, G., Albors, C., Aw, A. J., Ye, C., & Song, Y. S. (2024).
A DNA language model based on multispecies alignment predicts the effects of genome-wide variants.
Nature Biotechnology.
https://doi.org/10.1038/s41587-024-02023-1

@article{benegas2024dna,
  title={A DNA language model based on multispecies alignment predicts the effects of genome-wide variants},
  author={Benegas, Gonzalo and Albors, Carlos and Aw, Alan J and Ye, Chengzhong and Song, Yun S},
  journal={Nature Biotechnology},
  year={2024},
  doi={10.1038/s41587-024-02023-1}
}


##  Contact

If you have any questions or suggestions, please feel free to reach out to me at nvarjunmani07@gmail.com.


