# 🤖 Tech Clarity Copilot - RAG + Llama-3.2-1B Fine-Tuning 🚀

## About

This repository contains the code for fine-tuning the **`unsloth/Llama-3.2-1B`** model using **Supervised Fine-Tuning (SFT)**. The goal is to train the model, which we call "Tech Clarity Copilot," to consistently structure **RAG (Retrieval-Augmented Generation)** responses into a clear, structured format for non-technical users.

Originally, the RAG and SFT workflows lived in two separate Jupyter notebooks. This created a major friction point:
the user had to manually run the RAG notebook to retrieve context for a question, copy that context, and then paste it into the SFT notebook for training or inference.

Not only was this slow and complicated, but it also meant the two systems were never truly connected.

To fix this, we merged the RAG and SFT pipelines into a single, unified workflow. This new design provides an end-to-end pipeline where retrieval and fine-tuned model inference happen seamlessly without any manual steps.

---

## 🏃‍♀️ Reproduction Instructions and Execution Order

The entire core pipeline, including environment setup, data preparation, model training, and sample evaluation, is contained within a single executable script/notebook `sft_rag_clarity_copilot.ipynb`.

### Required Files
- `sft_rag_clarity_copilot.ipynb` (Main RAG + SFT pipeline)
- `tech_clarity_copilot_rag_doc.md` (RAG Knowledge Base)

### Order of Execution

1.  **Open the file:** Start with `sft_rag_clarity_copilot.ipynb`.
2.  **Upload Knowledge Base:** When running via Google Colab, make sure to upload the knowledge base file `tech_clarity_copilot_rag_doc.md` to current session storage.
3.  **Run all cells:** Execute all cells in sequential order. This will handle dependency installation, model loading, dataset definition, training, and final inference.
---

## 💻 Code and Documentation

### Logical Code Blocks and Comments

The following steps are executed sequentially within the script, with comments explaining each major logical block:

#### *SFT Section*

| Logical Block (Cell) | Purpose |
| :--- | :--- |
| **Package Installation, Imports** | Installs `unsloth`, `trl`, `transformers`, and `langchain` dependencies, handling different environments (Colab vs. local). |
| **Model & Tokenizer Loading** | Loads the base `unsloth/Llama-3.2-1B` model and its tokenizer, enabling 4-bit quantization and setting `max_seq_length` to 2048 for memory efficiency. |
| **LoRA Configuration** | Sets the **LoRA hyperparameters** (`r=4`, `lora_alpha=8`) and applies the LoRA adapters to the model's target layers (`q_proj`, `k_proj`, `v_proj`). |
| **Data Definition** | Defines the in-memory **RAG training examples** (`rag_examples`) and converts the Python list into a Hugging Face `Dataset`. |
| **Data Formatting** | *a)* Applies the `llama-3` chat template. *b)* **Converts RAG samples to Llama-3 conversation format**, including the explicit four-section **System Message**. *c)* Applies the chat template to create the final `text` field for SFT. |
| **Base Model Evaluation** | Defines the `generate_tech_response` function and runs the **base model** on sample tasks to show pre-fine-tuning performance (unstructured output). |
| **SFT Training Setup** | Configures the `SFTTrainer` with the defined LoRA and training hyperparameters (`learning_rate`, `num_train_epochs=0.3`, `batch_size=2`, `gradient_accumulation_steps=6`). |
| **Training & Saving** | Runs `trainer.train()`, and saves the final fine-tuned model and tokenizer to the `outputs/final_model` directory. |
| **Fine-Tuned Evaluation** | Runs the **fine-tuned model** on the same sample tasks to demonstrate the improved, structured output that aligns with the System Message. |

#### *RAG Section*
| Logical Block (Cell) | Purpose |
| :--- | :--- |
| **Imports and Setup** | Imports essential Python libraries for the RAG chain (LangChain components, Vector Stores, Text Splitters) and the fine-tuned model loading (`FastLanguageModel`). |
| **Load Knowledge Base** | Defines a utility function (`load_text_from_file`) to ingest documents (.md, .pdf, .docx). Loads the primary RAG source documentation from `./tech_clarity_copilot_rag_doc.md`. |
| **Document Chunking** | Uses the `RecursiveCharacterTextSplitter` to segment the raw knowledge base into smaller, manageable chunks (`CHUNK_SIZE=1200`, `CHUNK_OVERLAP=150`) for effective embedding. |
| **Embeddings + Vector Store** | Configures and initializes Azure OpenAI Embeddings (`text-embedding-3-small`). Creates a FAISS Vector Store from the processed document chunks, saving them in an index optimized for retrieval. |
| **Retriever Setup** | Defines the `retriever` object, which is configured to perform a similarity search and fetch the top `k=5` most relevant document chunks based on a user's question. |
| **Load Fine-Tuned SFT Model** | Loads the fine-tuned LoRA adapter model ("Tech Clarity Copilot") from the checkpoint directory (`outputs/final_model`) using `FastLanguageModel.from_pretrained`, then sets the model to inference mode. |
| **Prompt Templates** | Defines the structure for model input: `RAG_PROMPT_TEMPLATE` (includes the retrieved context) and `NON_RAG_PROMPT_TEMPLATE` (general knowledge fallback). |
| **Helper Functions** | Defines key orchestration functions: a) `llm_inference_wrapper` (Tokenizes, runs `model.generate()`, and decodes the LLM response). b) `format_docs` (Concatenates retrieved text chunks). c) `ask_question` (Manages the complete RAG sequence: $\text{Query} \rightarrow \text{Retriever} \rightarrow \text{LLM}$ wrapper). |
| **Demo** | Runs the full RAG pipeline using a list of `demo_questions`. This step validates the entire system's functionality and produces the final, structured output for evaluation. |

---

## 💾 Data-Code Mapping

The following outlines which data components are used by which parts of the code:

| Data Component | Type/File | Code Consumption Point(s) |
| :--- | :--- | :--- |
| **Base LLM** | `unsloth/Llama-3.2-1B` (Hugging Face) | `FastLanguageModel.from_pretrained` (Used to load and initialize the model for SFT). |
| **SFT Training Data** | `rag_examples` (Python list / Hugging Face `Dataset`) | `dataset.map(convert_to_conversation_format)` (Defines the training content). |
| **Evaluation Set** | `dataset` indices `[0, 1, 2]` | **Base Model Evaluation** and **Fine-Tuned Evaluation** cells (Used to demonstrate improvement). |
| **RAG Knowledge Base** | **None** (The RAG context is hardcoded into `retrieved_context` in `rag_examples` for SFT). | `rag_examples` and `generate_tech_response` function (The context is part of the instruction to the model). |
