# A Multimodal Data Extraction Pipeline with Table Layout Correction
<div>

Official implementation of our Canadian AI 2026 paper:

> [**A Multimodal Data Extraction Pipeline with Table Layout Correction**](https://proceedings.mlr.press/v318/yao26a.html) <br>
> Kecen Yao, Anton Shesternev, Ahmad Pesaranghader, and Erin Li<br>
> The 39th Canadian Conference on Artificial Intelligence, Vancouver, BC, Canada

</div>

Financial documents such as paystubs, invoices, and financial statements contain heterogeneous layouts and visually complex tables, making reliable information extraction challenging for both optical character recognition (OCR) based pipelines and end-to-end vision–language models (VLMs). In this paper, we present a pipeline that unifies layout analysis, one-shot multimodal table correction, and downstream extraction and reasoning without any model fine-tuning. The pipeline converts document images into a hybrid Markdown-HTML representation and applies a multi-modal correction module to rectify layout-level errors in tables, yielding demonstrable improvements in Tree-Edit-Distance based Similarity (TEDS) scores. Additionally, using this corrected representation, the system performs robust schema-based extraction and document-level question answering. Experimental results across paystub field extraction and finance question-answering (QA) tasks show that our approach consistently outperforms both OCR-only pipelines and direct VLM baselines. These results demonstrate that incorporating explicit table layout and multimodal table correction provides a scalable and generalizable path toward robust financial document understanding.

> **Code coming soon.** The source code and supporting materials are currently being prepared for public release — see [`src/`](src/) for status.

## Overview

To alleviate the burden of manual processing for client documents—particularly those containing complex tables and unstructured layouts—we propose a system that significantly improves the accuracy of information extraction from scanned financial documents. Conventional OCR-based methods often fail to capture the hierarchical and visual structure inherent in such documents, especially in tabular formats. Our approach incorporates layout analysis, visually grounded table correction, and document-specific prompting to enable accurate and scalable extraction across diverse document types, including paystubs, invoices, and general financial records. The architecture is designed to be easily adaptable to unseen document formats and generalizes across multiple downstream tasks—such as key-value pair extraction, document reconstruction, and document-level question answering—while remaining cost-efficient and suitable for deployment in enterprise environments.

## Approach

The pipeline is organized into two stages, applied across two task settings:

**Data Element Extraction Pipeline Flow**
```
Input Document (Image/PDF)
    ↓
1. Layout Analysis
    ↓
2. Table Correction (visually grounded)
    ↓
3. Data Extraction (document-specific)
    ↓
Structured Output
```

**Document QA Pipeline Flow**
```
Input Document + Questions
    ↓
1. Layout Analysis
    ↓
2. Table Correction (visually grounded)
    ↓
3. Question Answering
    ↓
QA Results
```

## Experiments Result

### Task 1. Data Element Extraction

#### Payslip

Dataset: 53 payslip images, 159 key-value pairs
- 10 CIBC in-house payslip samples (redacted)
- 43 samples randomly sampled from the [Kaggle Finance Document Image dataset](https://www.kaggle.com/datasets/mehaksingal/personal-financial-dataset-for-india)

For each image, we extract three fields: employer_name, employee_name, gross_income.

| Pipeline | Data Element Extraction Accuracy (%) |
|----------|-------|
| gpt-4o-vision | 97.5 |
| Azure read + gpt-4o | 94.9 |
| Mistral-ocr + gpt-4o | 95.6 |
| Nanonets-OCR-s + gpt-4o | 95.6 |
| Nanonets-OCR-s + table correction + gpt-4o | 98.1 |
| Azure layout (text reconstruction) + gpt-4o * | 98.7 |
| Azure layout (markdown) + gpt-4o | 99.3 |
| **Azure layout (markdown) + table correction + gpt-4o** ** | **100** |

\* Current Documind Approach &nbsp;&nbsp; ** Our Best Approach

#### Invoice

Dataset: 174 invoice images, 1,218 key-value pairs, randomly sampled from [inv-cdip](https://github.com/salesforce/inv-cdip/tree/master).

For each image, we extract seven fields: Invoice_number, Purchase_order, Invoice_date, due_date, amount_due, total_amount, total_tax_amount.

| Pipeline | Data Element Extraction Accuracy (%) |
|----------|-------|
| gpt-4o-vision | 89.0 |
| Azure read + gpt-4o | 87.2 |
| Mistral-ocr + gpt-4o | 86.2 |
| Nanonets-OCR-s + gpt-4o | 88.2 |
| Nanonets-OCR-s + table correction + gpt-4o | 89.7 |
| Azure layout (markdown) + gpt-4o | 89.9 |
| **Azure layout (markdown) + table correction + gpt-4o** * | **91.1** |

\* Our Best Approach

---

### Task 2. Document Question Answering

Dataset: 154 finance document images, 992 question-answer pairs, stratified sampled from [sujet-ai/Sujet-Finance-QA-Vision-100k](https://huggingface.co/datasets/sujet-ai/Sujet-Finance-QA-Vision-100k/viewer/default/train?views%5B%5D=train&row=4) (labels manually corrected due to a high rate of mislabeling in the source dataset).

| Pipeline | Question Answer Accuracy (%) |
|----------|-------|
| gpt-4o-vision | 66.6 |
| Azure read + gpt-4o | 66.7 |
| Mistral-ocr + gpt-4o | 62.3 |
| Nanonets-OCR-s + gpt-4o | 62.3 |
| Nanonets-OCR-s + table correction + gpt-4o | 65.0 |
| Azure layout (markdown) + gpt-4o | 71.3 |
| **Azure layout (markdown) + table correction + gpt-4o** * | **74.0** |

\* Our Best Approach

---

### Task 3. Multi-modal Table Correction

Dataset: 300 Pubtabnet image-HTML pairs, randomly sampled from the [Pubtabnet test set](https://github.com/ajjimeno/icdar-task-b/tree/master) complex type.

| Correction Method | TEDS (structure)* | TEDS (structure + text)* |
|----------|-------|-------|
| None (no table correction) | 0.82 | 0.69 |
| Text only correction | 0.82 | 0.69 |
| Multi-modal correction 1 ** | 0.87 | 0.75 |
| Multi-modal correction 2 *** | 0.86 | 0.75 |
| One-shot + multi-modal correction 1 | 0.87 | 0.75 |
| **One-shot + multi-modal correction 2** (our approach) | **0.88** | **0.76** |

\* Tree-Edit-Distance-based Similarity &nbsp;&nbsp; ** Prompt (simplified): "help me correct html by looking at the table image" &nbsp;&nbsp; *** Prompt (simplified): "help me convert this table image to html and take the input html as a reference"

## Citation

```bibtex
@inproceedings{yao2026multimodal,
  title={A Multimodal Data Extraction Pipeline with Table Layout Correction},
  author={Yao, Kecen and Shesternev, Anton and Pesaranghader, Ahmad and Li, Erin},
  booktitle={The 39th Canadian Conference on Artificial Intelligence},
  pages={64--76},
  year={2026},
  organization={PMLR}
}
```

## License

The published paper, “A Multimodal Data Extraction Pipeline with Table Layout Correction,” is available under the Creative Commons Attribution 4.0 International License (CC BY 4.0).

The CC BY 4.0 license applies to the published paper and does not automatically apply to the source code in this repository. The source code is currently being prepared for public release, and a separate software license will be added when it becomes available.

Third-party datasets, models, APIs, and other dependencies remain subject to their respective licenses and terms of use.
