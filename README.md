# Connecting the Dots Challenge - Round 1B Solution

This project is a solution for Round 1B of the "Connecting the Dots" Challenge. It functions as an intelligent document analyst that processes a collection of PDF documents to extract and prioritize the most relevant sections based on a given user persona and their "job-to-be-done."

The system operates entirely offline within a Docker container, adhering to all specified constraints on model size, execution time, and resource usage.

---
## Methodology and Approach

Our approach is a multi-stage pipeline designed to first understand the structure of the documents and then apply semantic analysis to find the most relevant content.

### 1. Document Structuring (Round 1A Foundation)

The process begins by leveraging the `AdvancedPDFProcessor` developed in Round 1A. This module analyzes each PDF using a feature-based heuristic model. It evaluates text based on properties like **font size, boldness, and spatial layout** to identify and extract a hierarchical outline (Title, H1, H2, H3). This creates a structured "map" for each document, which is the foundation for all subsequent analysis.

### 2. Full-Text Section Extraction

Once the outline is established, a helper function (`extract_text_for_sections`) iterates through the headings. It extracts the complete text content that falls between one heading and the next. This process transforms the outline into a corpus of discrete, meaningful sections, each tied to a specific heading and document.

### 3. Semantic Ranking with Sentence Embeddings

This is the core intelligence of the solution. To determine relevance without relying on simple keywords, we use a powerful yet lightweight sentence embedding model.

* **Model:** We use **`sentence-transformers/all-MiniLM-L6-v2`**. This model was chosen because it offers an excellent balance of performance and size (~86 MB), runs efficiently on a CPU, and works perfectly offline, fitting all competition constraints.

* **Query Formulation:** The `persona` and `job_to_be_done` from the `config.json` file are combined into a single, rich query. For example: `"Persona: Travel Planner. Task: Plan a trip of 4 days for a group of 10 college friends."`

* **Similarity Calculation:** The model converts the query and the content of every extracted section into numerical vectors (embeddings). The relevance is then calculated by computing the **cosine similarity** between the query's vector and each section's vector. Sections with the highest similarity score are ranked as the most important.

### 4. Sub-Section Analysis via Extractive Summarization

To generate the `refined_text` for the `subsection_analysis` block, we employ an extractive summarization technique. For each of the top-ranked sections:
1.  The full text is split into individual sentences.
2.  Each sentence is then converted into an embedding using the same AI model.
3.  The similarity between each sentence's embedding and the original query's embedding is calculated.
4.  The top 3-5 sentences with the highest similarity scores are selected and joined together to create a concise, relevant summary that directly addresses the user's task.

---
## Models and Libraries Used

* **Python 3.9**: The core programming language.
* **PyMuPDF (`fitz`)**: Used for robust PDF parsing and text extraction.
* **Sentence-Transformers**: The primary library for loading and utilizing the `all-MiniLM-L6-v2` embedding model.
* **PyTorch**: Serves as the backend for the `sentence-transformers` library.

---
## How to Build and Run the Solution

The entire solution is containerized using Docker and is designed to run offline.

### Prerequisites

* Docker Desktop installed and running.

### 1. Prepare Your Input

1.  Create an `input` directory in the project root.
2.  Place all your PDF files inside the `input` directory.
3.  Create a `config.json` file inside the `input` directory with the following structure:
    ```json
    {
      "persona": {
        "role": "Travel Planner"
      },
      "job_to_be_done": {
        "task": "Plan a trip of 4 days for a group of 10 college friends."
      }
    }
    ```
4.  Create an empty `output` directory in the project root.

### 2. Build the Docker Image

Open a terminal in the project's root directory and run the build command. This will install all dependencies and download/cache the AI model inside the image.

```bash
docker build -t solution1b:latest .
```
3. Run the Solution
Execute the following command to run the container. It mounts the local input and output folders and runs with networking disabled, as per the challenge requirements.

For Windows (CMD):

DOS

docker run --rm -v "%cd%\input:/app/input" -v "%cd%\output:/app/output" --network none solution1b:latest
For Linux / macOS / PowerShell:

Bash

docker run --rm -v "$(pwd)/input:/app/input" -v "$(pwd)/output:/app/output" --network none solution1b:latest
The analysis will run, and upon completion, the results will be saved to a file named challenge1b_output.json inside your local output folder.
