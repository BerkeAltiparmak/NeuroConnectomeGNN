# FlyWire Connectome Analysis and Preliminary GNN Setup

## Overview
This project aims to explore the **FlyWire connectome** of *Drosophila melanogaster* (the fruit fly) by leveraging graph-based techniques and neural networks. The ultimate goal is to predict neuron function from connectivity data, basic cell size metrics, and classification annotations.

---

## Data Investigation

### 1. Connections Dataset
- **File:** `data/all/connections.csv`  
- **Purpose:** Contains synaptic connections between neurons (columns: `pre_root_id`, `post_root_id`, `syn_count`, `neuropil`, etc.).  
- **Findings:**
  - **Shape:** Several million rows, indicating the large scale of the connectome.  
  - **Synapse Count Distribution:** Filtered out outliers (1st–99th percentile) to get a clearer picture. Most connections have fewer than 10 synapses, but some outliers reach much higher counts.

<img width="726" alt="Screenshot 2025-03-05 at 01 10 39" src="https://github.com/user-attachments/assets/8ee8680a-bcdc-4fda-92ea-740fa463ec69" />


### 2. Cell Size Measurements
- **File:** `data/all/cell_stats.csv`  
- **Purpose:** Provides morphological metrics (`length_nm`, `area_nm`, `size_nm`) for each neuron.  
- **Observations:**
  - **Cell Size Distribution:** After excluding the top and bottom 1% outliers, the histogram shows a heavily skewed distribution, with many neurons having relatively small volumes and a tail extending to very large volumes.

<img width="719" alt="Screenshot 2025-03-05 at 01 11 02" src="https://github.com/user-attachments/assets/18057662-e4b2-414f-91da-41c17c5e6966" />


### 3. Classification Data
- **File:** `data/all/classification.csv`  
- **Purpose:** Maps each neuron (`root_id`) to a functional or anatomical class (e.g., “visual,” “Kenyon_Cell,” “mechanosensory”).  
- **Insights:**
  - **Class Distribution:** Highly imbalanced, with “visual” neurons and “Kenyon_Cell” types dominating.  
  - **Unique Classes:** Over 20 distinct labels, some quite rare (e.g., TPN, MBIN).

<img width="984" alt="Screenshot 2025-03-05 at 01 11 29" src="https://github.com/user-attachments/assets/a86ce69b-a156-49ef-a858-bb5efc4442cc" />

---

## Data Preparation

1. **Node and Edge Creation**  
   - Mapped each unique `root_id` to an integer index.  
   - Built a `PyTorch Geometric`-style `edge_index` tensor from the `connections` dataset.  

2. **Node Features**  
   - For each neuron, extracted `[length_nm, area_nm, size_nm]`.  
   - Assigned zero if morphological data was missing.

3. **Labeling and One-Hot Encoding**  
   - Created an integer mapping (`class2idx`) from string labels to numeric IDs.  
   - Assigned `-1` for neurons without valid labels.  
   - Converted valid integer labels to **one-hot vectors**, preparing for potential multi-class or multi-label training scenarios.

---

## Preliminary Graph Visualization

- **Subgraph Extraction:**  
  - Displayed the first 100 neurons (`sub_nodes = list(range(100))`) to keep the plot manageable.
- **Node Size Proportional to `size_nm`:**  
  - Normalized cell volume (3rd column in features) to a range suitable for visualization.
- **Node Color by Class:**  
  - Used a colormap (`viridis`), with gray for neurons lacking valid labels.
- **Layout and Background:**  
  - Employed a spring layout (`nx.spring_layout`) with a black background for clarity.  
  - Iterations and the “k” parameter were tuned to reduce node overlap.

**Result:**  
A partial network view (Figure: “Subgraph of FlyWire Connectome (First 100 Nodes)”) showing clusters of similarly labeled neurons. Some nodes are larger (indicating higher `size_nm`), while color differences reflect class distinctions.

---

## Preliminary Progress and Observations

1. **Data Download & Cleaning:**  
   - Successfully loaded the three main datasets (Connections, Cell Stats, Classification).  
   - Filtered out extreme outliers (1st–99th percentile) for histograms to better understand typical distributions.

2. **Graph Construction:**  
   - Created a large adjacency structure in `edge_index`, verifying that each neuron can serve as both pre- and post-synaptic partner.  
   - Mapped morphological features to each node.

3. **Label Handling & One-Hot Encoding:**  
   - Established a robust approach for dealing with missing labels (`-1`) and converting valid labels to a one-hot format.  
   - Identified an imbalanced class distribution (dominated by “visual” and “Kenyon_Cell”).

4. **Subgraph Visualization:**  
   - Confirmed the feasibility of plotting smaller sections of the connectome with node size and color keyed to morphological and classification data.

---

## Potential Challenges and Mitigations

1. I personally don't think the FlyWire connectome is extremely large (< 1GB), but in the case that it's too much for NEURO140 GPU, it can lead to memory constraints and slow computation. In that case, I'll start with subsets (e.g., specific brain regions or a limited number of neurons) for initial testing. I'll scale up incrementally, ensuring hardware resources are sufficient.

2. A big problem will be the imbalanced classes. Certain classes dominate the dataset, risking biased classification. My plan is to consider weighted loss functions, oversampling minor classes, or specialized metrics (e.g., F1-score) to address imbalance.

3. Finally, I observed that many neurons lack classification annotations or have incomplete morphological data. I will compare the model performance on hierarchical classification dataset with the community labeled one to decide on which one to use, as the community labeled one has more data but it is more prone to be mislabeled. This will also be an interesting experiment :)

---

## Next Immediate Steps

1. Experiment with Different Models 
   - Graph Convolutional Network (GCN), Graph Attention Network (GAT), and other GNN variants to identify which architecture best fits this large-scale problem.

2. Feature Variation  
   - Test how the model performs with and without certain features (e.g., morphological data vs. connectivity tags) to evaluate the importance of each input.

3. Training Set Size Variation  
   - Vary the proportion of labeled data (e.g., 10%, 25%, 50%, 75%) to see how many labeled neurons are necessary for robust performance.

4. Initial Model Training  
   - Implement a baseline GNN training loop (e.g., with PyTorch Geometric) on a smaller subset of neurons to confirm the pipeline works end-to-end.

5. Evaluate Performance Metrics  
   - Use accuracy, precision, recall, F1-score (or other relevant metrics) to gauge model success, especially given class imbalance.

---

## Conclusion

So far, this project has established a foundation by investigating the FlyWire dataset, constructing a node-feature-edge structure, and visualizing a small subgraph. Despite the dataset’s complexity, early results indicate it is feasible to proceed with GNN-based classification. The next steps involve scaling up experiments, testing various model architectures, and fine-tuning feature sets to achieve reliable neuron-function predictions from the Drosophila connectome.

**If you have any suggestions or resources to help tackle the large-scale nature of the dataset or the class imbalance, please let me know!**
