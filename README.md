# Assignment 3: Pre-training, Fine-tuning, and Prompting Strategies for Bug Fixing

**Course:** CSCI 455/555 — GenAI for Software Development, Spring 2026  
**Instructor:** Prof. Antonio Mastropaolo  

A comparative study of pre-training, fine-tuning, and retrieval-augmented generation
(RAG) strategies for automated bug fixing in Java code, evaluated on the
`code_x_glue_cc_code_refinement` test set.

---
## **Some of the model files were too large to push to github, so please find the paths and the google drive links to them below**:
**[data/model_finetuned_A/model.safetensors](https://drive.google.com/drive/folders/1ZXlOMQdtKaKR6zoXT8vUT3vLyRAvfD5_?usp=sharing)**<br>
**[data/model_finetuned_B/model.safetensors](https://drive.google.com/drive/folders/1b2t10B75Qq5V9H1cYtheHDxmdAOWIQWr?usp=sharing)**<br>
**[data/model_pretrained/model.safetensors](https://drive.google.com/drive/folders/1DoaRqhRvTVndFx7vS3cPF6PuEsZZpVEc?usp=sharing)**<br>
Find the base directory here: [Models Folder](https://drive.google.com/drive/folders/11FlmkjDGhhqsS8Reuq6EbhOM6kPA_F6S?usp=sharing)

## Pipelines Overview

### Pipeline A — Pre-train then Fine-tune
Pre-train a T5-small model on 50,000 Java methods using span corruption,
then fine-tune on bug-fixing pairs from the CodeGLUE refinement dataset.

### Pipeline B — Fine-tune from Scratch
Fine-tune a T5-small model directly on the bug-fixing task without
any pre-training, providing a baseline to isolate the effect of pre-training.

### Pipeline C — Retrieval-augmented Generation (RAG)
- **Few-shot:** Retrieve relevant bug-fix examples from a knowledge base
  and include them in the prompt for Qwen2.5-Coder-1.5B
- **Zero-shot:** Baseline prompt without any retrieved examples

---

## Dependencies and Reproduction

### Requirements
- Python 3.10+
- CUDA-capable GPU recommended (tested on NVIDIA RTX 4050 with CUDA 11.8)

### Installation
```bash
# 1. Clone this repository and navigate into it
git clone https://github.com/AlanGonzalez10123/genai_Assignment3
cd genai_Assignment3

# 2. Install dependencies (get installed by Section 0 in notebook)
pip install transformers==4.46.0 torch==2.2.0 datasets tokenizers \
            codebleu faiss-cpu gensim tqdm sentencepiece
```

### Running the Notebook

Open `assignment3.ipynb` in Jupyter and run all cells top-to-bottom.
The notebook is fully self-contained and will:

1. **Section 0** — install/verify all dependencies *(run once, then restart kernel)*
2. **Section 1** — train SentencePiece tokenizer on Java code
3. **Section 2** — pre-train T5-small via span corruption (Pipeline A)
4. **Section 3** — fine-tune pre-trained model on bug fixing (Pipeline A)
5. **Section 4** — fine-tune T5-small from scratch on bug fixing (Pipeline B)
6. **Section 5** — build knowledge base and retriever for RAG
7. **Section 6** — run RAG inference (few-shot and zero-shot)
8. **Section 7** — evaluate all four configurations (CodeBLEU + Exact Match)
9. **Section 8** — summarize results

> **Note (Windows):** `sentencepiece==0.1.99` may fail to build from source.
> Install a pre-built wheel instead:
> `pip install sentencepiece --prefer-binary`

---

## Output Locations

| Output | Path |
|---|---|
| Tokenizer model | `Data/java_code_tokenizer.model` |
| Pre-trained model | `Data/pretrain_t5_small/java_model` |
| Fine-tuned models | `Data/t5_bugfix_pretrained`, `Data/t5_bugfix_scratch` |
| RAG knowledge base | `Data/rag_knowledge_base` |
| Predictions (all configs) | `Data/predictions/` |
| Results summary | `Data/results_summary.json` |

---

## Model Architectures

### Pipeline A: Pre-train + Fine-tune
- **Pre-training:** T5-small (60M parameters), span corruption objective,
  50,000 Java method corpus
- **Fine-tuning:** Same model, fine-tuned on `code_x_glue_cc_code_refinement`
  training set

### Pipeline B: Fine-tune from Scratch
- **Model:** T5-small (60M parameters)
- **Training:** Direct fine-tuning on `code_x_glue_cc_code_refinement`
  training set

### Pipeline C: RAG
- **Base model:** Qwen2.5-Coder-1.5B (open weights)
- **Retriever:** FAISS index over bug-fix pairs from training set
- **Prompting:** K-shot retrieval (K=3) with relevant examples

---

## Evaluation

| Metric | Description |
|---|---|
| CodeBLEU | Weighted combination of n-gram matching, AST similarity, and dataflow |
| Exact Match | Binary exact match between predicted and reference code |

All pipelines evaluated on the instructor-provided
`code_x_glue_cc_code_refinement` test set.