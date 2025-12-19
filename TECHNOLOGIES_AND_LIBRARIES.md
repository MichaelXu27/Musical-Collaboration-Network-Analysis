# Technologies and Libraries Used

## Overview
This project is a **Musical Collaboration Network Analysis** application built with Python, leveraging graph analysis, machine learning, and music data APIs.

---

## Programming Language
- **Python 3** - Core language for all analysis and computation

---

## Core Data Processing & Analysis Libraries

### Data Manipulation & Processing
- **Pandas** (`pandas`) - Data manipulation, DataFrame operations, Parquet file handling
- **NumPy** (`numpy`) - Numerical computing, array operations
- **PyArrow** (`pyarrow`) - Parquet file serialization and deserialization

### Graph & Network Analysis
- **NetworkX** (`networkx`) - Graph construction, network analysis, link prediction algorithms
  - Used for: Common Neighbors, Jaccard Coefficient, Adamic-Adar Index, Preferential Attachment
  - Graph generation and manipulation (directed/undirected graphs)
  - Community detection integration
- **Python-louvain** (`community` as `community_louvain`) - Community detection using Louvain algorithm
  - Used in hybrid models for identifying graph communities

---

## Machine Learning & Evaluation
- **Scikit-Learn** (`sklearn`) - ML utilities and evaluation metrics
  - `sklearn.preprocessing.MinMaxScaler` - Feature normalization
  - `sklearn.metrics.roc_auc_score` - AUC-ROC evaluation
  - `sklearn.metrics.precision_score` - Precision metric
  - `sklearn.metrics.recall_score` - Recall metric
  - `sklearn.metrics.precision_recall_curve` - PR curve computation

---

## Data Source & API Integration

### Discogs API
- **Data Source**: Discogs XML files
  - `discogs_20251101_artists.xml` - Artist metadata (ID, name, genres)
  - `discogs_20251101_releases.xml` - Release information with collaborations
- **Data Format**: XML parsing with `xml.etree.ElementTree`

### Spotify API
- **Spotipy** (`spotipy`) - Spotify Web API client library
  - `from spotipy.oauth2 import SpotifyClientCredentials` - OAuth 2.0 authentication
  - Used for: Retrieving artist genres, music metadata enrichment
- **Python-dotenv** (`dotenv`) - Environment variable management for API credentials

---

## Data Storage & File Format
- **Parquet Format** - Efficient columnar data storage
  - Files: `discogs_edges.parquet`, `discogs_artists.parquet`, `discogs_artists_and_genres.parquet`
- **GEXF Format** - Gephi network visualization format
  - Generated for network visualization with different link prediction metrics

---

## Visualization & Plotting
- **Matplotlib** (`matplotlib.pyplot`) - Static visualization
  - Histograms for score distributions
  - Performance metric plots

---

## Notebook & Computation Environment
- **Jupyter** (`jupyter`) - Interactive notebook environment
- **Jupyter Notebook** (`notebook`) - Notebook server
- **IPython Kernel** (`ipykernel`) - Python kernel for Jupyter

---

## XML Processing
- **xml.etree.ElementTree** - Standard library XML parsing
  - Iterative parsing for large files (10GB+ Discogs XML files)
  - Memory-efficient element-by-element processing

---

## Utility Libraries
- **re** (Regular Expressions) - Standard library pattern matching
  - Used for extracting years from date strings in XML
- **itertools** - Standard library for iterator tools
  - Generating node pairs for link prediction
- **os** - Standard library for file system operations
- **glob** - Standard library for file path patterns
- **io** - Standard library for I/O operations

---

## Key Data Pipeline

```
Discogs XML Files
    ↓
    └─→ XML Parsing (iterparse, ElementTree)
         ↓
         └─→ Edge/Artist Extraction
              ↓
              └─→ Pandas DataFrames
                   ↓
                   └─→ Parquet Storage
                        ↓
                        ├─→ Structural-Based Link Prediction
                        │   └─→ NetworkX (CN, Jaccard, AA, PA)
                        │
                        ├─→ Genre-Based Link Prediction
                        │   └─→ Spotify API (genres)
                        │
                        └─→ Hybrid Model
                            └─→ NetworkX + Louvain + Spotify
                                 ↓
                                 └─→ Sklearn Evaluation Metrics
                                      ↓
                                      └─→ GEXF (Gephi Visualization)
```

---

## Summary Table

| Category | Technologies |
|----------|-------------|
| **Language** | Python 3 |
| **Data Processing** | Pandas, NumPy, PyArrow |
| **Graph Analysis** | NetworkX, Python-louvain |
| **ML & Evaluation** | Scikit-Learn |
| **APIs** | Discogs (XML), Spotify API (Spotipy) |
| **Authentication** | Python-dotenv, SpotifyClientCredentials |
| **Visualization** | Matplotlib, Gephi (GEXF export) |
| **Notebooks** | Jupyter, IPython Kernel |
| **File Formats** | Parquet, GEXF, XML |
| **Utilities** | re, itertools, os, glob, xml.etree.ElementTree |

---

## Requirements File Dependencies

All dependencies are specified in `requirements.txt`:

```
pandas
numpy
pyarrow
networkx
spotipy
lxml
tqdm
matplotlib
scikit-learn
jupyter
notebook
ipykernel
python-louvain  # (imported as 'community')
python-dotenv
```

---

## Notebooks Overview

### 1. **final_data_processing.ipynb**
- **Purpose**: Data pipeline from raw XML to clean Parquet files
- **Key Libraries**: XML parsing, Pandas, NetworkX (for export)
- **Output**: `discogs_edges.parquet`, `discogs_artists.parquet`

### 2. **link_prediction.ipynb**
- **Purpose**: Structural-based link prediction using graph metrics
- **Key Libraries**: NetworkX, Sklearn metrics
- **Methods**: Common Neighbors, Jaccard, Adamic-Adar, Preferential Attachment
- **Era-based**: 5 historical eras of music

### 3. **genre_based.ipynb**
- **Purpose**: Genre-based link prediction using Spotify data
- **Key Libraries**: Spotipy, Pandas, Sklearn metrics
- **Hypothesis**: Artists sharing genres more likely to collaborate

### 4. **community_based.ipynb**
- **Purpose**: Community detection and analysis
- **Key Libraries**: NetworkX, Python-louvain
- **Methods**: Louvain community detection algorithm

### 5. **optimal.ipynb**
- **Purpose**: Hybrid model combining structural + genre + community features
- **Key Libraries**: All above plus full ML pipeline
- **Model**: Weighted score combining multiple features
- **Evaluation**: AUC-ROC, Precision-Recall, Hits@K

---

## API Specifications

### Discogs
- **Data Format**: XML (dump files)
- **Data Size**: 10GB+
- **Parsing Method**: Iterative parsing (iterparse)
- **Credentials**: Public data dumps (no authentication)

### Spotify
- **Authentication**: OAuth 2.0 (Client Credentials Flow)
- **Credentials Storage**: `.env` file (managed by python-dotenv)
- **Endpoints Used**: Artist search, artist metadata, genre retrieval
- **Rate Limits**: Standard Spotify API rate limiting applies

---

## Notes on Architecture

1. **Era-Based Analysis**: Data is split into 5 musical eras for temporal link prediction
2. **Temporal Validation**: Train/test splits respect chronological order
3. **Scalability**: Uses iterative XML parsing and memory-efficient operations
4. **Multi-Method Evaluation**: Multiple metrics (AUC, Precision, Recall, Hits@K)
5. **Visualization**: Export to GEXF for Gephi network visualization
