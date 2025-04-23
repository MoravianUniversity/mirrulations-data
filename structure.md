## Overview

The data from [regulations.gov](https://www.regulations.gov/) is split into two broad categories:

- **Raw Data**: The original data obtained from the source, organized by agency and docket ID, separating binary and textual content.
- **Derived Data**: Processed outputs derived from the raw data using future tools like AI models, entity extractors, and summarization pipelines.

Each element (docket, document, comment) is uniquely identified, and attachments (usually `.pdf`, `.doc`, or `.docx`) are stored alongside extracted or structured text versions.

---

## Raw Data Structure

The `raw-data` directory contains the original source data, separated by agency and docket ID.

```
raw-data
└── <agency>
    └── <docket id>
        ├── binary-<docket id>
        │   ├── comments_attachments
        │   │   ├── <comment id>_attachment_<counter>.<extension>
        │   │   └── ...
        │   └── documents_attachments
        │       ├── <document id>_attachment_<counter>.<extension>
        │       └── ...
        └── text-<docket id>
            ├── comments
            │   ├── <comment id>.json
            ├── docket
            │   ├── <docket id>.json
            ├── documents
            │   ├── <document id>.json
            │   ├── <document id>_content.htm
```

---

## Derived Data Structure

The `derived-data` directory contains outputs of processing pipelines, such as generated summaries, named entity recognition, and extracted text.

```
derived-data
└── <agency>
    └── <docket id>
        ├── mirrulations
        │   ├── ai_summary
        │   │   ├── comment
        │   │   ├── comment_attachments
        │   │   └── document
        │   ├── entities
        │   │   ├── comment
        │   │   ├── comment_attachment
        │   │   └── document
        │   └── extracted_txt
        │       └── comment_attachment
        │           └── <comment id>_attachment_<counter>_extracted.txt
        ├── MoravianResearch
        │   └── <projectName>
        │       ├── comment
        │       ├── docket
        │       └── document
        └── trotterf
            └── <projectName>
                └── <fileType>
                    └── <fileID>.txt
```

---


## Example and Explanation

### Example: `USTR-2015-0010` in Context of Full Structure

```
raw-data
└── USTR
    └── USTR-2015-0010
        ├── binary-USTR-2015-0010
        │   ├── comments_attachments
        │   │   ├── USTR-2015-0010-0002_attachment_1.pdf
        │   │   ├── USTR-2015-0010-0003_attachment_1.pdf
        │   └── documents_attachments
        │       ├── USTR-2015-0010-0001_attachment_1.pdf
        │       ├── USTR-2015-0010-0016_attachment_1.doc
        └── text-USTR-2015-0010
            ├── comments
            │   ├── USTR-2015-0010-0002.json
            │   ├── USTR-2015-0010-0003.json
            ├── docket
            │   └── USTR-2015-0010.json
            ├── documents
            │   ├── USTR-2015-0010-0001.json
            │   ├── USTR-2015-0010-0001_content.htm
            │   ├── USTR-2015-0010-0016.json
```

```
derived-data
└── USTR
    └── USTR-2015-0010
        ├── mirrulations
        │   ├── summary
        │   │   ├── comment
        │   │   ├── comment_attachments
        │   │   └── document
        │   ├── entities
        │   │   ├── comment
        │   │   ├── comment_attachment
        │   │   └── document
        │   └── extracted_txt
        │       └── comment_attachment
        │           ├── USTR-2015-0010-0002_attachment_1_extracted.txt
        │           └── USTR-2015-0010-0003_attachment_1_extracted.txt
```

### Explanation

* At the root level, there are two top-level directories: `raw-data` and `derived-data`.
  * Inside `raw-data`, data is organized first by the agency (e.g., `USTR`) and then by docket ID (e.g., `USTR-2015-0010`).
    * Each docket folder has two subfolders: `binary-<docket id>` and `text-<docket id>`.
    * The `binary-<docket id>` folder contains:
      * `comments_attachments`: Files named using the comment ID and attachment number, e.g., `USTR-2015-0010-0002_attachment_1.pdf`.
      * `documents_attachments`: Files named using the document ID and attachment number, e.g., `USTR-2015-0010-0016_attachment_1.doc`.
    * The `text-<docket id>` folder contains:
      * `comments`: JSON files named by comment ID, e.g., `USTR-2015-0010-0002.json`.
      * `docket`: A JSON file named after the docket ID, e.g., `USTR-2015-0010.json`.
      * `documents`: Includes metadata and content files, such as `USTR-2015-0010-0001.json` and `USTR-2015-0010-0001_content.htm`.

  * Inside `derived-data`, the structure mirrors `raw-data` starting from the agency and docket ID, but contains generated outputs.
    * Subfolders include `mirrulations`, `MoravianResearch`, and `trotterf`, which are organized by project or processing pipeline.
    * Under `mirrulations`, you’ll find:
      * `summary`, `entities`, and `extracted_txt`, each of which is broken down by type (comment, comment_attachment, document).
      * Extracted text files follow the naming convention `<comment id>_attachment_<counter>_extracted.txt`.

## Summary

- **Top-Level Separation**: `raw-data` for original unprocessed content, `derived-data` for all post-processing artifacts.
- **Agency and Docket Organization**: Both structures mirror agency/docketId hierarchy for consistency.
- **Binary/Text Separation**: In `raw-data`, attachments and text content are split for cleaner access and processing.
- **Structured Outputs**: In `derived-data`, various project folders house data outputs for derived information like summaries and text extraction(comment attachments).
