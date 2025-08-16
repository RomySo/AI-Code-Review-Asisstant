# CodeReviewer.AI  
> ### Transferable Code Review via Language-Agnostic Augmentation with LLMs  
> AI-powered binary classification of **C++ code review quality** using real and synthetic data — built for **transferability** across programming languages.

![Code Review Diagram](FinalPPT/GraphicalAbstract.png) 

---

## Task Overview

- **Input**: A code change composed of:
  - `oldf`: the original code
  - `patch`: the code update or diff
- **Output**: A binary label:
  - `0` — High quality (no additional review needed)
  - `1` — Low quality (requires further review)

---

## Motivation

In real-world software engineering, the emergence of new programming languages often brings a significant challenge: the lack of labeled data for tasks such as code review. This project addresses a key question:

> How can we enable effective code quality classification in low-resource programming languages?

To explore this, we examine the following:

- Can **large language models (LLMs)** be used to translate code from a well-established, high-resource language (eg.,Java)?
- Is it possible to fine-tune classifiers effectively on these **synthetically generated examples**?   
- Can models trained on synthetic data achieve performance comparable to those trained on **real labeled** data?

To address this challenge, we develop and evaluate two distinct models, each fine-tuned on a different data source: one using real labeled data, and the other using synthetically generated data.

---

## Dataset

### Real Data
- C++ and Java code review examples.

### Synthetic Data
- Java examples translated to C++ using **LLM-based code translation**.
- Translated data is used to simulate scenarios where real C++ labels are unavailable.

---

## Modeling Approach

We use **CodeBERT**, a pretrained transformer model as our base model.

### Two Fine-Tuned Models:

| Model     | Description                                             |
|-----------|---------------------------------------------------------|
| `M_source`| Fine-tuned on **real** C++ examples                     |
| `M_aug`   | Fine-tuned on **synthetic** C++ examples (Java → C++)   |

### Training Configuration

Below is a side-by-side comparison of the training setup for the two models:

| Parameter               | `M_source`                                   | `M_aug`                                     |
|-------------------------|----------------------------------------------|---------------------------------------------|
| **Base Model**          | CodeBERT                                     | CodeBERT                                    |
| **Trainer**             | `WeightedLossTrainer` (custom)               | Default Trainer                             |
| **Epochs**              | 8 (tested: 6, 10, 15)                        | 10 (tested: 15)                             |
| **Learning Rate**       | 2e-5 (tested: 3e-5, 1e-5, 2e-7)              | 1e-5 (tested: 2e-5)                         |
| **Batch Size**          | 32 (tested: 8, 16, 64)                       | 32 (tested: 16)                             |
| **Dropout**             | 0.3 (in classification head)                 | None                                        |
| **Gradient Accumulation**| 4 steps                                     | 4 steps                                     |
| **Warmup Ratio**        | 0.1                                          | 0.1                                         |
| **Weight Decay**        | 0.01                                         | 0.01                                        |
| **Eval/Checkpoint Every**| 250 steps                                   | 100 steps                                   |
| **Early Stopping**      | After 5 no-F1-improvement evals              | None                                        |
| **Best Model Criterion**| F1-score                                     | F1-score                                    |


---

## Evaluation

### Metrics
We evaluate both models on the **same held-out C++ test set**, using the following metrics:
- Accuracy
- Precision
- Recall
- F1-score

This comparison reveals how well the synthetic data performs in training compared to real examples.

---

## Workflow

Each model follows a systematic pipeline to ensure robust training and evaluation:

1. **Data Loading and Preprocessing**  
   Load raw code diffs and extract modified lines (additions and deletions) along with surrounding context lines.

2. **Exploratory Data Analysis (EDA)**  
   Analyze data distribution and quality to guide modeling choices.

3. **Tokenization**  
   Tokenize the preprocessed code using the CodeBERT tokenizer.

4. **Training and Fine-Tuning**  
   Fine-tune CodeBERT on either real C++ data (`M_source`) or synthetic translated data (`M_aug`).

5. **Evaluation**  
   Evaluate the models on a held-out C++ test set using accuracy, precision, recall, and F1-score metrics.

---
