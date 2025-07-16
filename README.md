# Resume Parser Tool

## 1. Overview

This project builds an **end‑to‑end pipeline** to extract structured candidate data from IT‑sector resumes (PDFs) and automatically evaluate extraction quality.

## 2. Technology Stack

* **Python / Jupyter Notebook** for development
* **PDF & OCR**

  * PyMuPDF for direct text extraction
  * Tesseract OCR for scanned pages
* **LLM Integration**

  * Azure OpenAI (GPT‑4) via LangChain’s `AzureChatOpenAI`
  * Pydantic models for strict JSON schemas
* **Data Handling & Schemas**

  * Pydantic for defining `Resume`, `FieldEval`, and `ResumeEval` schemas
  * Pandas for managing metadata and results

## 3. Workflow

1. **Data Acquisition**

   * Clone the data repo and focus on the `INFORMATION-TECHNOLOGY` PDFs.

2. **Text Extraction**

   * For each PDF:

     * Attempt direct text extraction with PyMuPDF.
     * Fallback to Tesseract OCR if no text is found.

3. **Entity Extraction**

   * Define a `Resume` schema with fields:

     ```
     name, email, phone, skills,
     education, experience, certifications, languages
     ```
   * Prompt Azure’s GPT model to output a JSON object conforming exactly to that schema.

4. **Automated Evaluation**

   * Define `FieldEval` and `ResumeEval` schemas to capture per‑field correctness.
   * Use a second LLM pass to compare each parsed field against the extracted resume text, producing “correct/incorrect” judgments.

5. **Metrics Aggregation**

   * **Field‑level accuracy**: percent of resumes with each field correctly extracted.
   * **Resume‑level accuracy**: fraction of correctly extracted fields per resume.

## 4. Observations & Results

I ran two evaluation configurations across **120** IT‑sector resumes:

1. **OCR‑only** (no direct PDF text extraction)
2. **PDF text + OCR** (use embedded text when available, fallback to OCR)

Below are the **field‑level** and **resume‑level** metrics for each.

---

### 4.1 Field‑Level Accuracy

| field          | OCR‑only accuracy | PDF+OCR accuracy |
| -------------- | ----------------- | ---------------- |
| name           | 1.000             | 1.000            |
| email          | 1.000             | 1.000            |
| phone          | 0.992             | 1.000            |
| skills         | 0.592             | 0.950            |
| education      | 0.617             | 0.833            |
| experience     | 0.642             | 0.817            |
| certifications | 0.883             | 0.950            |
| languages      | 0.983             | 1.000            |

* **Key improvements** when adding direct PDF text extraction:

  * **Skills** jumped from 59% to 95%
  * **Education** from 62% to 83%
  * **Experience** from 64% to 82%
  * **Certifications** from 88% to 95%
* Contact fields (name, email, phone, languages) reach **100%** accuracy with PDF+OCR, showing that relying solely on OCR risks missing easy‑to‑read embedded text.

---

### 4.2 Resume‑Level Accuracy

Using the hybrid PDF‐text + OCR pipeline on **120** resumes, we computed, for each resume, how many of the eight fields were correctly extracted. Here are the detailed findings:

#### Distribution of Correct Fields per Resume

| # Correct Fields | # Resumes | % of Total |
| ---------------- | --------: | ---------: |
| 8 (perfect)      |        84 |      70.0% |
| 7                |        24 |      20.0% |
| 6                |         7 |       5.8% |
| 5                |         4 |       3.3% |
| 4 (lowest)       |         1 |       0.8% |

* **84 resumes** had **all 8** fields correct.
* **108 resumes** had **≥7** fields correct.
* Only **1 resume** scored as low as **4/8**.

#### Insights

* **High accuracy on scalar fields**
  Name, email, phone, and languages achieve near-perfect extraction once both embedded text and OCR are enabled.

* **List fields remain challenging**
  Skills, education, experience, and certifications—each containing multiple entries—account for almost all residual mismatches.

* **Placeholder handling**
  Literal placeholders (e.g. “Company Name”) are correctly omitted by the parser but currently marked as errors by the evaluator; evaluation rules should treat such boilerplate as “no data” rather than missing content.

## 5. Limitations

* **Model Dependence**: Extraction and evaluation quality rely on the chosen Azure OpenAI model.
* **OCR Noise**: Scanned documents can introduce recognition errors.
* **Resume Variability**: Unconventional layouts or wording may challenge the prompts.
* **Cost & Latency**: Cloud LLM calls incur usage costs and runtime delays.

## 6. Future Improvements

* **Layout-Aware Parsing:** Incorporate document structure (headings, tables) to improve context detection.
* **Multi-Agent, Task-Specific Pipelines:** Deploy specialized agents—one for contacts, one for skills, one for education, etc.—so each can be finely tuned and maintained.
* **Fine-Tuning:** Adapt or fine-tune a custom model on annotated resume data for greater accuracy and lower cost.
* **Parallel Processing:** Execute independent stages (text extraction, OCR, LLM parsing, evaluation) concurrently to boost throughput and reduce end-to-end latency.
