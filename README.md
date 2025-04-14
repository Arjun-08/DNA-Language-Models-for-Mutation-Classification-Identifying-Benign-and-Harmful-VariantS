
#  DNA Variant Classifier with CNN (Pathogenic vs. Benign)

This project implements a deep learning pipeline using a 1D Convolutional Neural Network (CNN) to classify single-nucleotide variants (SNVs) in DNA sequences as **Pathogenic (1)** or **Benign (0)** based on sequence context from the GRCh38 human reference genome and ClinVar annotations.

---

##  Dataset

### 1. [ClinVar VCF (GRCh38)](https://ftp.ncbi.nlm.nih.gov/pub/clinvar/vcf_GRCh38/clinvar.vcf.gz)
- Annotated variants with clinical significance (Benign, Pathogenic, etc.)

### 2. [GRCh38 Reference Genome (FASTA)](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/001/405/GCA_000001405.15_GRCh38/seqs_for_alignment_pipelines.ucsc_ids/GCA_000001405.15_GRCh38_no_alt_analysis_set.fna.gz)
- Used to extract 101bp surrounding sequence for each SNV.

### Extraction Process
- Filter only **SNVs**
- Extract a 101bp sequence centered on the variant
- Replace the central base (reference) with the alternate allele
- Label:
  - `"Pathogenic"` → `1`
  - `"Benign"` → `0`
- Limit to `MAX_VARIANTS = 5000` for quick training

---

##  Model

A 1D CNN architecture implemented in PyTorch:

```
Input: One-hot encoded sequence (4 x 101)

Conv1D (4 → 64) + ReLU + MaxPool
Conv1D (64 → 128) + ReLU + MaxPool
Flatten → Dense (128 units) → Dropout
Output: Sigmoid (1 unit)
```

- Loss: Binary Cross Entropy
- Optimizer: Adam (`lr=0.001`)
- Epochs: 10
- Batch Size: 32

---

##  Training Progress

```
Epoch 01 - Loss: 0.5231 - Train Accuracy: 79.17%
Epoch 05 - Loss: 0.3462 - Train Accuracy: 83.90%
Epoch 10 - Loss: 0.1104 - Train Accuracy: 95.70%
```

---

##  Evaluation

On a 20% test split (1000 samples):

```
Test Accuracy: 77.60%
ROC AUC Score: 0.7497

Precision/Recall (Benign):     0.8765 / 0.8394
Precision/Recall (Pathogenic): 0.4416 / 0.5178
```

 The model performs well on benign examples, but struggles to recall pathogenic variants — likely due to class imbalance.

---

##  Example Prediction

```python
Input Sequence:
CACATCGTGCTTCTGGCGTCGTGAACTTCGCGTGCCTCCGCTCGTTTGCAACACGGTTCATTGTCGTGTCCCAGGCGGGCTCAGGCGGGCATCCCATTTAG

Predicted:
Probability: 0.8031
Label: Pathogenic (1)
```

---

##  How to Run

### 1. Install dependencies

```bash
pip install biopython pandas cyvcf2 tqdm scikit-learn matplotlib
```

### 2. Download and prepare dataset

```bash
# Either run the notebook or use:
python extract_variants.py
```

### 3. Train the model

```bash
python train_cnn.py
```

### 4. Predict on a new sequence

```python
from predict import predict_single_sequence
prob, label = predict_single_sequence("ACGT...101bp")
```



##  Future Improvements

- Handle class imbalance (SMOTE, focal loss)
- Use multiple sequence alignment or conservation scores
- Try Transformer-based DNA models (e.g., DNABERT, GPN-MSA)
- Fine-tune on ClinVar variants with family history or experimental validation

---

##  References

- ClinVar Database – NCBI
- GRCh38 Reference Genome – NCBI
- PyTorch, BioPython, cyvcf2

---

##  Author

**Arjun Sagar**  
[GitHub]([https://github.com/](https://github.com/Arjun-08)) • [Email](mailto:nvarjunmani07@gmail.com)

