**Date:** 10.08.2026
**Tags:** #RAG #data #datasets #huggingface

The Hugging Face `datasets` library is the standard way to load data for AI work. It handles downloading, caching, format conversion, and streaming out of the box.

```bash 
pip install datasets huggingface_hub
```

There is 2 types for using datasets. **Standard Download** and **Streaming**. The differences is:

| **Feature / Aspect**         | **Standard Download (streaming=False)**                          | **Streaming (streaming=True)**                                               |
| ---------------------------- | ---------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| **Storage Requirement**      | Requires local disk space equal to the full dataset size.        | **Zero disk space** required (data stays on the cloud).                      |
| **Startup Time**             | Slow initially (must wait for the full download to finish).      | **Instant** (starts processing the first row in milliseconds).               |
| **Data Object Type**         | `Dataset` (In-memory Apache Arrow table).                        | `IterableDataset` (Python Generator / Stream).                               |
| **RAM / Memory Usage**       | Low to moderate (uses memory-mapping via Apache Arrow).          | **Constant & minimal** (only loads the active batch into RAM).               |
| **Random Access (Indexing)** | **Supported** (e.g., `dataset[42]` or `dataset[10:20]`).         | **Not supported** (must iterate sequentially using `next()` or `for`).       |
| **Data Shuffling**           | Full global shuffle across the entire dataset.                   | **Buffer-based shuffle** (e.g., `shuffle(buffer_size=10000)`).               |
| **Splitting Data**           | Direct utilities like `train_test_split()`.                      | Manual splitting using stream operations like `.take()` and `.skip()`.       |
| **Offline Capability**       | Works offline once the dataset is cached locally.                | **Requires an active internet connection** throughout the execution.         |
| **Execution Speed**          | Fast after initial download (bound by local disk read speeds).   | Bound by network bandwidth and latency for every iteration.                  |
| **Best Use Case**            | Repeated model training, fine-tuning, and complex data analysis. | Quick prototyping, inspect large datasets, or constrained disk environments. |

Shortly;

- **cons for standart downloading**: Storage & Cold-Start Bottleneck

- **cons for streaming:** No Random Access / Indexing
	- you couldn't global shuffle. (or train_test_split)
	- network dependecy (it affects gpu starvation while drop/pings.)

### Choose file format:

|Format|Size|Read Speed|Best For|
|---|---|---|---|
|CSV|Large|Slow|Human readability, spreadsheets|
|JSON|Large|Slow|APIs, nested data|
|Parquet|Small|Fast|Analytics, columnar queries|
|Arrow|Small|Fastest|In-memory processing (what `datasets` uses internally)|
For disk: Parquet - For RAM : Arrow

### Data splits

Every ML project needs three splits:

- **Train:** The model learns from this (typically 80%)
- **Validation:** You check progress and overfitting during training (typically 10%) 
- **Test:** Final evaluation after training is done (typically 10%)

Some datasets come pre-split. When they don't, split them yourself:



```Python
from datasets import load_dataset

dataset = load_dataset("stanfordnlp/imdb", split="train")

# 1. Split into 80% train+val and 20% test
split = dataset.train_test_split(test_size=0.2, seed=42)

# 2. Split the 80% portion into train (70% total) and val (10% total) -> 0.125 of 80% = 10%
train_val = split["train"].train_test_split(test_size=0.125, seed=42)

train_ds = train_val["train"]  # %70
val_ds = train_val["test"]     # %10
test_ds = split["test"]        # %20

print(f"Train: {len(train_ds)}, Val: {len(val_ds)}, Test: {len(test_ds)}")
```

> **Note on `seed=42`:** Always set a seed for **reproducibility**. The same seed produces the exact same split every time you run the code.
### Download and cache models

Models are large files consisting of weights, tokenizers, and config files. The `huggingface_hub` library handles downloading and caching them automatically.

```Python
from huggingface_hub import hf_hub_download, snapshot_download

# Download a single file (e.g. only config)
model_path = hf_hub_download(
    repo_id="sentence-transformers/all-MiniLM-L6-v2",
    filename="config.json"
)
print(f"Cached at: {model_path}")

# Download the entire model repository
model_dir = snapshot_download("sentence-transformers/all-MiniLM-L6-v2")
print(f"Full model at: {model_dir}")
```

- **Cache Location:** Models cache to `~/.cache/huggingface/hub/` (Windows: `C:\Users\<user>\.cache\huggingface\hub\`).

- **Fast Reloads:** Once downloaded, subsequent calls load instantly from the local cache without redownloading.
---

---
date: 2026-08-10
tags:
  - RAG
  - data
  - datasets
  - huggingface
  - python
---

# Hugging Face Datasets: Data Pipeline & Splitting Workflow

This pipeline covers end-to-end data preprocessing using Hugging Face `datasets`: loading, feature engineering via `.map()`, cleaning via `.filter()`, custom train/val/test splitting, class distribution verification, and multi-format exports (`.parquet`, `.json`, `.csv`).

## Full Pipeline Implementation

```python
from datasets import load_dataset
from collections import Counter

# 1. Load raw dataset split
dataset = load_dataset("dair-ai/emotion", split="train")

# 2. Feature Engineering: Add text length column
def add_text_length(example):
    example["text_length"] = len(example["text"])
    return example

# 3. Filtering: Drop short/noisy samples (< 20 chars)
def filter_short_texts(example):
    return example["text_length"] >= 20

dataset = dataset.map(add_text_length)
dataset = dataset.filter(filter_short_texts)

# Extract label names list for O(1) index lookups
label_names_list = dataset.features["label"].names

# 4. Custom Data Splitting (80% Train, 10% Val, 10% Test)
# Step 1: Split 20% for Test
split_step1 = dataset.train_test_split(test_size=0.2, seed=42)

# Step 2: Split remaining 80% into Train and Val (0.125 of 80% = 10% total)
split_step2 = split_step1["train"].train_test_split(test_size=0.125, seed=42)

train_dataset = split_step2["train"]       # 70% -> ~80% of total valid train set
validation_dataset = split_step2["test"]    # 10%
test_dataset = split_step1["test"]          # 20%

# 5. Verify Class Distribution
label_counts = Counter(dataset["label"])
label_counts_train = Counter(train_dataset["label"])
label_counts_validation = Counter(validation_dataset["label"])
label_counts_test = Counter(test_dataset["label"])

print(f"Train dataset size: {len(train_dataset)}")
print(f"Validation dataset size: {len(validation_dataset)}")
print(f"Test dataset size: {len(test_dataset)}")

print("-" * 40)
print("Label counts (Overall):")
for label, count in label_counts.items():
    print(f"{label_names_list[label]}: {count} adet")

print("-" * 40)
print("Label counts (Train):")
for label, count in label_counts_train.items():
    print(f"{label_names_list[label]}: {count} adet")

# 6. Disk Export
train_dataset.to_json("train_dataset.json")
train_dataset.to_csv("train_dataset.csv")
train_dataset.to_parquet("train_dataset.parquet")