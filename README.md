# HybridRAG-QM-FAQ

## Semi-automatic KG Ontology for QM FAQ
- **Main Goal**
  - This notebook semi-auttomatically creates the KG using a pre-built JSON file. Also the JSON can be iteratively improvised by visualizing the KG in Neo4j as explained below.
  - The KG uses Neo4j Graph DB that contains both triplets (subject, relattion, object) and DocumentChunk nodes (raw chunks with chunk embeddings), with a MENTIONS relation connecting DocumentChunk nodes to the entity nodes mentioned in it.
  - The KG also needs a vector index on the chunk embeddings for similarity search. For now, it is based on cosine similarity, but it can be improved by combining dense (e.g., cosine similarity) and sparse (e.g., BM25) similarity metrics by updating the CQL query.
  - This HybridRAG pipeline formats the context of the prompt by finding the top-k chunks by Vector Search on Neo4j Chunk Embedding Index, then finds the triplets that has some node(s) mentioned in the Chunk and the 1-hop neighbours of these nodes.
- **Pre-processing steps to create JSON of triplets** [Diagram Given below]:
  - Summarize each section (1-2 paragraphs) using the Prompt-V3_1.
  - Turned a 30-page PDF into 4-pages Summary PDF.
  - Highlighted some of the selected sentences from the Summary PDF.
  - Take each of the selected sentences from the Summary PDF and give the Prompt-V3_2.
  - Analyse manually to be consistent with the 4-step verification.
- **JSON format**:
  - This is a sample of the JSON format used here for the triplets.
  - {
    "raw_sentence": "The time-dependent wave function, $\Psi(x,t)$, is determined by the Hamiltonian operator, $\hat{H}$.",
    "suggested_triplets": [{
      "subject_name": "time-dependent wave function",
      "subject_class": "Mathematical_Object",
      "relationship": "IS_DETERMINED_BY",
      "object_name": "Hamiltonian operator",
      "object_class": "Operator"
    },
    }]

- **Important steps in this implementation for converting the JSON into Neo4j GraphDB**:
  - Neo4j Util Functions to get credentials and driver.
  - Delete existing Graph and Indexes in Neo4j Graph DB.
  - Add triplets from JSON into Neo4j Graph DB.
  - Add DocumentChunk nodes with raw chunks in the "text" attribute and chunk embedding in the "embedding" attribute, and a MENTIONS relation connecting DocumentChunk node to all the nodes mentioned in it.
  - Add a vector index on the "embedding" attribute of all the DocumentChunk nodes.
  - After One round of this entire process some issues like disconnected islands were found and it was manually fixed in the JSON itself as mentioned in the later part of this notebook. After the JSON was updated, this notebook was run again. A few repetitions in this way gave the final KG.

- **KG Visualizations**: 2 Important visualizations are given here:
  1. Final KG before adding the DocumentChunk nodes and MENTIONS relations. How the disconnected components relates to different sub-topics as shown here.
  2. Final KG after adding the DocumentChunk nodes and MENTIONS relations. How the entire KG became more connected in this way as shown here.

<img width="2228" height="1366" alt="image" src="https://github.com/user-attachments/assets/c9744a93-64ea-4387-be93-dfef25e495a3" />

- **Final KG before adding the DocumentChunk nodes and MENTIONS relations**: How the disconnected components relates to different sub-topics as shown here.
<img width="1426" height="1340" alt="image" src="https://github.com/user-attachments/assets/a39272b9-a805-4ce1-bfd5-50d35fd17644" />
  
- **Final KG after adding the DocumentChunk nodes and MENTIONS relations**: How the entire KG became more connected in this way as shown here.
<img width="1080" height="1456" alt="image" src="https://github.com/user-attachments/assets/4b10b3fc-a51e-4c5f-9336-6ab84c8363e5" />

