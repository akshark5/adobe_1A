

# PDF Outline Extractor (Round 1A)

## Overview

The **PDF Outline Extractor** is a lightweight, offline Python-based system that ingests a PDF file (up to 50 pages) and outputs a structured JSON outline of its title and hierarchical headings (H1, H2, H3), including page numbers. It is optimized for CPU-only environments and completes processing in ≤10 seconds for a 50-page document.

## Tech Stack

* **Language**: Python 3.9+
* **PDF Parsing**: PyMuPDF (`fitz`)
* **Text Processing**: Python Standard Library (`re`, `statistics`, `collections`)
* **Containerization**: Docker (linux/amd64)
* **Data Format**: JSON

## Dependencies

* Python ≥ 3.9
* PyMuPDF

## Why These Libraries?

### PyMuPDF (`fitz`)

* Efficient text and layout extraction with font metadata.
* Lightweight, pure Python with strong AMD64 compatibility.
* Outperforms alternatives like PDFMiner (slower) and PDFBox (requires JVM).

### Python Standard Library

* Regex, statistics, and frequency utilities enable effective heading detection without external dependencies.
* Avoids bulky ML models to ensure speed and low system footprint.

## Setup & Installation

1. **Navigate into the folder**

   ```
   "Adobe\adobe_extractor_ready\app\main.py"
   ```
   **For inputting PDFs navigate to **
   ```
   "Desktop\adobe_extractor_ready\app\input"
   ```

2. **Build Docker Image**

   ```powershell
   docker build -t pdf-extractor -f dockerfile .
   ```

3. **Run Extraction**

   ```powershell
   powershell -ExecutionPolicy Bypass -File .\run.ps1
   
   ```

   All `*.pdf` files in `/app/input` will produce `.json` outputs in `/app/output`.

## Usage Example

```powershell
# Place PDFs in ./app/input
"Desktop\adobe_extractor_ready\app\input"

# View output
"Desktop\adobe_extractor_ready\app\output"
```

## How It Works

1. **Block Extraction**
   Uses PyMuPDF to read text spans along with font size, font name, bold flag, and coordinates.

2. **Line Merging**
   Merges adjacent text spans with similar style to form complete lines.

3. **Title Detection**
   On the first page, identifies the top-most, largest line (after filtering) as the document title.

4. **Heading Scoring**
   Assigns a heading score (0–9) based on font attributes, layout cues, and punctuation. Lines with scores ≥ 4 are classified as headings.

5. **Level Assignment**
   Uses numbering patterns and relative font size to assign H1, H2, or H3 levels.

6. **JSON Output**
   Outputs structured data:

   ```json
   {
     "title": "Sample Document",
     "outline": [
       { "level": "H1", "text": "Introduction", "page": 1 },
       { "level": "H2", "text": "Background", "page": 2 }
     ]
   }
   ```

## Heading Scoring Criteria

| Feature                                                       | Points |
| ------------------------------------------------------------- | ------ |
| Font size > body text size                                    | 2      |
| Font size ≥ 1.3× body text size                               | 2      |
| Bold text (font name includes 'bold' or bold flag set)        | 1      |
| Numbering pattern (e.g., "1.2", "3.1.1")                      | 1      |
| Ends with heading punctuation (`:`, `：`, `。`, `、`, `،`)     | 1      |
| Short line (≤ 6 words) without terminal period                | 1      |
| Ends with punctuation and followed by smaller, non-bold block | 1      |

**Maximum Score**: 9
**Threshold for Heading**: Score ≥ 4

## Performance & Constraints

* Execution Time: ≤ 10 seconds per 50-page PDF on 8-core CPU with 16 GB RAM
* Model Size: 0 MB (fully rule-based)
* Offline Operation: No network access required

## Extensibility

* Scoring thresholds can be tuned for specific document types.
* Additional heading levels (H4–H6) can be introduced via regex and font analysis.
* Lightweight ML models (< 200 MB) can be integrated for layout or semantic enhancements if needed.

## Options That Satisfy Major Benchmarks 

* An HTML-based model was initially tested but discarded due to inconsistent structure extraction and limited multilingual support.
* A document-type classifier (academic vs. general) was explored but omitted due to unreliable performance in the absence of labeled data and consistent structural features.

## Final Approach

The final system adopts a unified, font-style heuristic method combined with optional keyword-based semantic matching. This strategy offers strong accuracy and generalization across document types while meeting performance and size constraints.

---



  

---

