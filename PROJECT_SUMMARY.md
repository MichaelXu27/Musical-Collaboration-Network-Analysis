# Music Collaboration Network Analysis: Link Prediction Across Musical Eras

## Project Overview

This project performs large-scale link prediction on a Discogs music collaboration network spanning 130+ years of music history (1897-2025). The analysis employs multiple machine learning and graph-based approaches to predict future artist collaborations, with era-specific modeling to account for significant shifts in music culture, technology, and artist ecosystems across five distinct musical periods.

---

## Key Objectives

1. **Build and preprocess** a comprehensive Discogs collaboration network with 1000+ artists and 10,000+ edges
2. **Implement multiple link prediction models** using structural, genre-based, and community-detection approaches
3. **Develop era-based predictions** to preserve musical context and improve model interpretability
4. **Compare predictive approaches** to identify the strongest signals for collaboration likelihood
5. **Create hybrid models** combining complementary prediction techniques for optimal performance

---

## Technical Approach

### 1. Data Processing & Preparation (`final_data_processing.ipynb`)

**Data Source:** Discogs XML database (~4GB) containing complete discography and artist relationships

**Processing Pipeline:**
- Parse and extract artist metadata and collaboration edges from Discogs XML
- Create temporal edge list with release years for chronological analysis
- Integrate Spotify API data to obtain artist genre classifications (1000+ artists)
- Generate parquet-formatted datasets for efficient downstream analysis
  - `discogs_artists.parquet` (~1000 artists)
  - `discogs_edges.parquet` (~10,000 collaborations)
  - `discogs_artists_and_genres.parquet` (enriched with Spotify genres)

**Data Cleaning:**
- Handle missing and malformed entries from large-scale XML parsing
- Remove duplicate edges and self-loops
- Validate data consistency across multiple sources

---

### 2. Structural Link Prediction (`link_prediction.ipynb`)

**Methodology:** Classical graph-based link prediction following Liben-Nowell & Kleinberg (2003)

**Five Musical Eras (Temporal Splits):**
- **Era 1** (1897-1945): Early Recording era
- **Era 2** (1946-1965): Post-WWII / Early Rock
- **Era 3** (1966-1982): Classic Rock / Studio Revolution
- **Era 4** (1983-1999): MTV and Digital Sampling
- **Era 5** (2000-2025): Internet / Streaming era

**Rationale:** Era-based splits preserve musical context and account for:
- Changes in artist ecosystems and musical genres
- Evolution of recording technology and industry
- Significantly different collaboration patterns across periods

**Train/Test Strategy:**
- 80/20 temporal split within each era (chronological cutoff)
- Train graph: collaborations ≤ cutoff year
- Test set: future collaborations only involving artists known in training period
- Ensures model learns from historical patterns only

**Structural Features Computed:**
1. **Common Neighbors (CN):** Direct triadic closure - shared neighbors between artists
2. **Jaccard Coefficient:** Normalized neighborhood similarity
3. **Adamic-Adar Index:** Weighted CN accounting for neighbor degree
4. **Preferential Attachment (PA):** Probability based on node degree

**Evaluation Metrics:**
- **AUC-ROC:** Area under receiver-operating characteristic curve
- **Precision/Recall:** Classification performance at various thresholds
- **Hits@K:** Ranking quality - fraction of true edges in top-K predictions
- **Score Distribution:** Separation between positive and negative samples

---

### 3. Community-Based Link Prediction (`community_based.ipynb`)

**Hypothesis:** Artists in the same community are more likely to collaborate

**Methods:**
- **Community Detection:** Louvain method for optimal modularity-based partitioning
- **Modularity Analysis:** Quantifying community cohesion within network structure
- **Community Overlap Metrics:** Binary indicators for co-membership
- **Visualization:** Network graphs with community coloring (Gephi format)

**Feature Engineering:**
- Community detection on training graph
- Compute community overlap as binary feature (same community = 1)
- Combine with structural features for enhanced predictions
- Analyze how community structure changes across eras

**Findings:**
- Community structure strongly predicts collaboration in Era 5 (modern music)
- Traditional rock eras show weaker community signals
- Community overlap combined with structural features improves ranking

---

### 4. Genre-Based Link Prediction (`genre_based.ipynb`)

**Hypothesis:** Artists sharing similar musical genres are more likely to collaborate

**Approach:**
- Extract genre labels from Spotify API (artists with ≥1 genre match)
- Compute genre similarity metrics for all artist pairs
- Combine genre information with structural network features

**Genre Features:**
1. **Genre Jaccard:** Normalized intersection of genre sets
2. **Shared Genre Count:** Raw overlap in genre classifications
3. **Same Genre Binary:** Simple presence/absence indicator
4. **Genre Profile Diversity:** Variance in artist genre portfolios

**Analysis:**
- Genre homophily varies significantly across eras
- Era 5 shows strongest genre correlation (specialized streaming era)
- Earlier eras have more genre-crossing collaborations
- Pure genre-based features underperform vs. structural methods alone

---

### 5. Hybrid Optimization Model (`optimal.ipynb`)

**Unified Framework:** Weighted combination of all prediction signals

**Hybrid Score Function:**
$$\text{Score}(u,v) = w_1 \times CN + w_2 \times JC + w_3 \times \text{GenreJaccard} + w_4 \times \text{GenreCount} + w_5 \times \text{SameGenre} + w_6 \times \text{CommOverlap}$$

Where:
- CN = Normalized Common Neighbors
- JC = Jaccard Coefficient
- GenreJaccard = Genre set similarity
- GenreCount = Normalized shared genre count
- SameGenre = Binary shared genre flag
- CommOverlap = Binary community co-membership

**Weight Optimization:**
- Grid search approach: iterate weights 0 to 1 in 0.1 increments
- Optimization metric: Hits@K for top-10,000 predictions
- Hold-one-out feature ablation to identify contribution of each signal
- Final weights learned on validation set

**Optimal Weights (Era 5):**
- CN_WEIGHT: 1.0
- JACCARD_WEIGHT: 1.0
- GENRE_JACCARD_WEIGHT: 0.0
- GENRE_COUNT_WEIGHT: 0.0
- SAME_GENRE_WEIGHT: 0.2
- COMMUNITY_WEIGHT: 0.8

**Results:**
- Final AUC-ROC: 0.85+ (Era 5)
- Hits@1000: significant lift over single-feature approaches
- Community overlap is strongest modern signal
- Structural features dominate genre-based signals

---

## Key Findings

### Cross-Era Comparison

| Era | Period | Primary Signal | AUC | Notes |
|-----|--------|---|---|---|
| Era 1 | 1897-1945 | Common Neighbors | ~0.75 | Small networks, sparse data |
| Era 2 | 1946-1965 | Preferential Attachment | ~0.78 | Growing artist base |
| Era 3 | 1966-1982 | Jaccard, PA | ~0.80 | Peak collaboration era |
| Era 4 | 1983-1999 | Multi-modal | ~0.82 | Digital era emergence |
| Era 5 | 2000-2025 | Communities + Structural | ~0.85 | Streaming ecosystem |

### Insights

1. **Structural Features Dominate:** Common Neighbors and Jaccard coefficient are most reliable across all eras
2. **Community Effect Strengthens:** Modern era shows strong community-based patterns (algorithmic playlists, genre consolidation)
3. **Genre Homophily Limited:** Shared genres predict collaboration better in specialized modern era; weaker in classical periods
4. **Temporal Dynamics Matter:** Separate models per era outperform single global model by 5-10% AUC
5. **Complementary Signals:** Hybrid models leverage non-redundant information from different feature classes

---

## Technical Stack

**Languages & Libraries:**
- **Python 3.x**: Primary analysis language
- **pandas/NumPy**: Data manipulation and numerical computation
- **NetworkX**: Graph construction, traversal, and analysis
- **scikit-learn**: Metrics (AUC, Precision/Recall), preprocessing (scaling)
- **python-louvain**: Community detection via modularity optimization
- **Spotipy**: Spotify API integration for genre enrichment
- **Gephi**: Network visualization in gexf format

**Data Formats:**
- **Parquet**: Efficient columnar storage for large datasets
- **XML**: Original Discogs database format (iterative parsing for memory efficiency)
- **GEXF**: Graph interchange format for Gephi visualization

---

## Reproducibility & Artifacts

**Output Datasets:**
- `data/final_data_processed/` - Processed artist and edge data
- `outputs/link_prediction_outputs/` - Era-specific prediction results
- `gephi_data/raw_data/` - Visualization files for each era and method

**Notebooks (Executable Pipelines):**
1. `final_data_processing.ipynb` - Data extraction and preparation
2. `link_prediction.ipynb` - Structural prediction on 5 eras
3. `genre_based.ipynb` - Genre similarity analysis
4. `community_based.ipynb` - Community detection approach
5. `optimal.ipynb` - Hybrid model optimization

**Model Evaluation:**
- Comprehensive metrics for all approaches (AUC, Precision, Recall, Hits@K)
- Visual comparisons via matplotlib (score distributions, PR curves)
- Network-level visualizations in Gephi

---

## Impact & Applications

**Academic/Research Value:**
- Demonstrates novel era-based temporal link prediction framework
- Validates hybrid feature combination for graph mining
- Shows evolution of music collaboration patterns over 130 years

**Practical Applications:**
- Artist recommendation systems for music platforms
- Collaborative opportunity identification for musicians
- Music industry trend analysis and forecasting
- A&R decision support tools

---

## Summary

This project demonstrates a comprehensive analysis of music collaboration networks using multiple complementary link prediction approaches. By decomposing the task into distinct musical eras and combining structural, genre-based, and community features, we achieve high-quality predictions that capture the evolving nature of artistic collaboration. The hybrid optimization framework provides a principled way to leverage multiple signals for improved accuracy, with clear interpretability across temporal domains.
