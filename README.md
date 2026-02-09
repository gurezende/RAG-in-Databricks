# Databricks Native Vector Search for RAG Applications

This project demonstrates how to build a Retrieval-Augmented Generation (RAG) system entirely within the Databricks ecosystem. By using native Databricks Vector Search, you can eliminate the need for external vector databases like Pinecone or ChromaDB and manage your entire data-to-embedding pipeline in one place.

## Features

* One-Stop-Shop: Leverages Databricks native infrastructure for catalogs, schemas, and vector indexes.
* Delta Lake Integration: Uses Change Data Feed (CDF) to automatically sync data updates to your search index.
* Semantic Search: Implements high-dimensional text embeddings for precise information retrieval.

## Setup and Installation

1. Prerequisites
You can run this tutorial using the Databricks Free Edition. Ensure you have an OpenAI API key for the final LLM generation step.

2. Install Required Packages
Run the following magic command in your Databricks notebook to install the necessary SDKs:

```Python
%pip install databricks-vectorsearch openai --quiet
dbutils.library.restartPython()
```

3. Environment Configuration
Securely add your API key using Databricks widgets:

```Python
dbutils.widgets.text(name="OPENAI_API_KEY", defaultValue="your_key_here")
OPENAI_API_KEY = dbutils.widgets.get('OPENAI_API_KEY')
```

## Implementation Guide

### Step 1: Initialize Clients

Connect to the Databricks Workspace and Vector Search client.

```Python
from databricks.vector_search.client import VectorSearchClient
from databricks.sdk import WorkspaceClient

ws = WorkspaceClient()
vsc = VectorSearchClient(disable_notice=True)
```

### Step 2: Prepare the Data Foundation

Create a dedicated catalog and schema, then enable Change Data Feed (CDF) on your source table to allow efficient incremental indexing.

```SQL
-- Create infrastructure
CREATE CATALOG IF NOT EXISTS vector_search;
CREATE SCHEMA IF NOT EXISTS vector_search.vectors;

-- Enable CDF for syncing
ALTER TABLE vector_search.vectors.docs SET TBLPROPERTIES (delta.enableChangeDataFeed = true);
```

### Step 3: Create the Vector Search Index

* Configuration of the search index can be performed via the Databricks UI under the Compute > Vector Search tab:
* Create a Vector Search Endpoint.
* Create a Search Index tied to your docs table.
* Select text as the source column and `databricks-gte-large-en` as the embedding model.

### Step 4: Perform Semantic Search

Retrieve relevant context based on a user query.

```Python
index = vsc.get_index(endpoint_name="my_endpoint", index_name="vector_search.vectors.search_index")

results = index.similarity_search(
    query_text="How do I cancel my subscription?",
    columns=["text"],
    num_results=2
)
```

### Step 5: Generate Answer

Pass the retrieved context to the LLM (e.g., GPT-4o-mini) to generate a grounded response.

#### Summary of Workflow

1. Connect the notebook to Vector Search.
2. Create catalogs and schemas.
3. Ingest data into Delta tables.
4. Index and sync embeddings.
5. Query and generate RAG responses.

## About

This project was created by [Gustavo R. Santos](https://gustavorsantos.me)
