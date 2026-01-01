# K-Means Clustering

## 1. Overview

K-Means is an unsupervised machine learning algorithm that partitions $n$ data points into $k$ clusters by minimizing the within-cluster variance. Introduced by Stuart Lloyd in 1957 (though published in 1982), it's one of the most widely used clustering algorithms due to its simplicity and efficiency.

The algorithm iteratively assigns points to clusters and updates cluster centers until convergence, making it effective for pattern recognition, data compression, and exploratory data analysis.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a set of $n$ data points $X = \{x_1, x_2, ..., x_n\}$ in $d$-dimensional space and a positive integer $k$, partition the points into $k$ clusters $C = \{C_1, C_2, ..., C_k\}$ such that the within-cluster sum of squares (WCSS) is minimized:

$$\text{WCSS} = \sum_{i=1}^{k} \sum_{x \in C_i} \|x - \mu_i\|^2$$

where $\mu_i$ is the centroid (mean) of cluster $C_i$.

### 2.2 Mathematical Model

**Input**:
- Dataset $X = \{x_1, x_2, ..., x_n\}$ where $x_i \in \mathbb{R}^d$
- Number of clusters $k$ where $1 \leq k \leq n$

**Output**:
- Cluster assignments $c: X \rightarrow \{1, 2, ..., k\}$
- Cluster centroids $M = \{\mu_1, \mu_2, ..., \mu_k\}$ where $\mu_i = \frac{1}{|C_i|} \sum_{x \in C_i} x$

**Objective Function**:
$$\min_{C, M} \sum_{i=1}^{k} \sum_{x \in C_i} \|x - \mu_i\|^2$$

**Properties**:
1. Each point belongs to exactly one cluster
2. Centroids are the means of their respective clusters
3. Algorithm minimizes intra-cluster distance
4. Non-convex optimization (local minima possible)

### 2.3 Correctness Proof

**Theorem**: K-Means algorithm monotonically decreases the objective function and converges.

**Proof Sketch**:

1. **Assignment Step**: Assigning each point to the nearest centroid minimizes $\sum_{x \in C_i} \|x - \mu_i\|^2$ for fixed centroids.

2. **Update Step**: Setting $\mu_i = \frac{1}{|C_i|} \sum_{x \in C_i} x$ minimizes $\sum_{x \in C_i} \|x - \mu_i\|^2$ for fixed assignments (this is the mean minimizing squared distances).

3. **Monotonic Decrease**: Each step strictly decreases or maintains the objective function value.

4. **Finite States**: Only finitely many possible assignments exist.

5. **Convergence**: Since the objective decreases monotonically and is bounded below by 0, and there are finitely many states, the algorithm must converge.

**Note**: Convergence is to a local minimum, not necessarily global. Multiple runs with different initializations are recommended.

## 3. Algorithm Description

### 3.1 Intuition

K-Means works through an iterative process:

1. **Initialize**: Randomly select $k$ points as initial cluster centers
2. **Assignment**: Assign each point to the nearest center (forming clusters)
3. **Update**: Recalculate centers as the mean of all points in each cluster
4. **Repeat**: Steps 2-3 until centers stop moving (or change is minimal)

**Why it works**: 
- Assignment step minimizes distance from points to centers
- Update step finds the optimal center for current clusters
- These two steps alternately optimize, gradually improving the clustering

### 3.2 Pseudocode

```
function KMeans(data, k, max_iterations=100, tolerance=1e-4):
    n = number of data points
    d = dimensionality of data
    
    // Step 1: Initialize centroids
    centroids = selectRandomCentroids(data, k)
    
    previous_centroids = null
    
    for iteration from 1 to max_iterations:
        // Step 2: Assignment - assign each point to nearest centroid
        clusters = [[] for i in 1 to k]
        
        for each point in data:
            distances = [distance(point, centroids[i]) for i in 1 to k]
            nearest_cluster = argmin(distances)
            clusters[nearest_cluster].append(point)
        
        // Step 3: Update - recalculate centroids
        previous_centroids = centroids
        
        for i from 1 to k:
            if clusters[i] is not empty:
                centroids[i] = mean(clusters[i])
            else:
                // Handle empty cluster: reinitialize or remove
                centroids[i] = selectRandomPoint(data)
        
        // Step 4: Check convergence
        if centroidShift(centroids, previous_centroids) < tolerance:
            break
    
    return (centroids, clusters)

function distance(point1, point2):
    // Euclidean distance
    return sqrt(sum((point1[i] - point2[i])^2 for i in dimensions))

function centroidShift(new_centroids, old_centroids):
    // Maximum movement of any centroid
    return max(distance(new_centroids[i], old_centroids[i]) for i in 1 to k)

// Common initialization methods
function selectRandomCentroids(data, k):
    // K-Means++: smarter initialization
    centroids = []
    centroids[0] = random choice from data
    
    for i from 1 to k-1:
        distances = []
        for point in data:
            min_dist = min(distance(point, c) for c in centroids)
            distances.append(min_dist^2)
        
        // Select next centroid with probability proportional to distance^2
        centroids[i] = weighted random choice from data using distances
    
    return centroids
```

### 3.3 Step-by-Step Example

Cluster 6 points into k=2 clusters:

**Data**: `A(1,1), B(1.5,2), C(3,4), D(5,7), E(3.5,5), F(4.5,5)`

```
Iteration 0: Initialize
  Random centroids: μ₁=(1,1), μ₂=(4.5,5)

Iteration 1:
  Assignment:
    A(1,1): dist to μ₁=0, dist to μ₂=5.7 → Cluster 1
    B(1.5,2): dist to μ₁=1.1, dist to μ₂=4.3 → Cluster 1
    C(3,4): dist to μ₁=3.6, dist to μ₂=1.8 → Cluster 2
    D(5,7): dist to μ₁=7.2, dist to μ₂=2.1 → Cluster 2
    E(3.5,5): dist to μ₁=5.0, dist to μ₂=1.0 → Cluster 2
    F(4.5,5): dist to μ₂=0 → Cluster 2
  
  Clusters:
    C₁ = {A, B}
    C₂ = {C, D, E, F}
  
  Update centroids:
    μ₁ = mean(A, B) = ((1+1.5)/2, (1+2)/2) = (1.25, 1.5)
    μ₂ = mean(C, D, E, F) = ((3+5+3.5+4.5)/4, (4+7+5+5)/4) = (4.0, 5.25)

Iteration 2:
  Assignment with new centroids μ₁=(1.25,1.5), μ₂=(4.0,5.25):
    A(1,1): dist to μ₁=0.56, dist to μ₂=5.5 → Cluster 1
    B(1.5,2): dist to μ₁=0.56, dist to μ₂=4.0 → Cluster 1
    C(3,4): dist to μ₁=3.1, dist to μ₂=1.6 → Cluster 2
    D(5,7): dist to μ₁=6.7, dist to μ₂=2.0 → Cluster 2
    E(3.5,5): dist to μ₁=4.5, dist to μ₂=0.66 → Cluster 2
    F(4.5,5): dist to μ₂=0.56 → Cluster 2
  
  Same assignments → Converged!
  
Final clusters:
  C₁ = {A(1,1), B(1.5,2)}
  C₂ = {C(3,4), D(5,7), E(3.5,5), F(4.5,5)}
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Per iteration**:
  - Assignment: $O(nkd)$ - compare $n$ points to $k$ centroids in $d$ dimensions
  - Update: $O(nd)$ - calculate mean of points
  - Total per iteration: $O(nkd)$

- **Number of iterations**: $I$ (typically 10-100, depends on data)
  - Worst case: exponential in $n$ (rare in practice)
  - Typical case: converges in $O(1)$ to $O(\log n)$ iterations

- **Total**: $O(Inkd)$ where $I$ is number of iterations

**Optimizations**:
- K-Means++: Better initialization, $O(nk \log k)$ extra cost but fewer iterations
- Mini-batch K-Means: $O(bkd)$ per iteration where $b << n$
- Accelerated K-Means: Techniques to skip distance calculations

### 4.2 Space Complexity

- **Centroids**: $O(kd)$
- **Cluster assignments**: $O(n)$
- **Data storage**: $O(nd)$ (input, not counted as auxiliary)
- **Total auxiliary space**: $O(n + kd)$

In-place assignment tracking can reduce to $O(kd)$ if assignments aren't stored.

## 4. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Customer Segmentation**
- E-commerce: grouping customers by behavior
- Marketing: identifying target demographics
- Personalization: content recommendation groups

**2. Image Processing**
- Image compression: reducing color palette
- Image segmentation: separating regions
- Background removal
- Quantization

**3. Document Classification**
- Topic modeling
- News article clustering
- Email categorization
- Search result grouping

**4. Anomaly Detection**
- Network intrusion detection
- Fraud detection: identifying unusual patterns
- System monitoring: grouping normal vs abnormal behavior

**5. Data Compression**
- Vector quantization
- Feature engineering
- Dimensionality reduction preprocessing

**6. Bioinformatics**
- Gene expression analysis
- Protein sequence clustering
- Medical image analysis

### 6.2 Related Algorithms

**Variations**:
- **K-Means++**: Better centroid initialization
- **Mini-Batch K-Means**: Faster, uses random samples
- **K-Medoids (PAM)**: Uses actual data points as centers
- **Fuzzy C-Means**: Soft clustering (points belong to multiple clusters)
- **X-Means**: Automatically determines k

**Related Clustering**:
- **Hierarchical Clustering**: Builds tree of clusters
- **DBSCAN**: Density-based, finds arbitrary shapes
- **Mean Shift**: Mode-seeking algorithm
- **Gaussian Mixture Models**: Probabilistic clustering
- **Spectral Clustering**: Uses graph theory

**Preprocessing**:
- **PCA**: Dimensionality reduction before clustering
- **Normalization**: Scale features appropriately
- **Elbow Method**: Determine optimal k

**When to Use**:
- **K-Means**: Spherical clusters, known k, large datasets
- **DBSCAN**: Arbitrary shapes, noise handling
- **Hierarchical**: Need dendrogram, small datasets
- **GMM**: Probabilistic assignments, overlapping clusters

## 7. References

### Academic Papers
1. Lloyd, S.P. (1982). "Least Squares Quantization in PCM". *IEEE Transactions on Information Theory*, 28(2), 129-137.
2. Arthur, D., & Vassilvitskii, S. (2007). "k-means++: The Advantages of Careful Seeding". *SODA '07*, 1027-1035.
3. MacQueen, J. (1967). "Some Methods for Classification and Analysis of Multivariate Observations". *Proceedings of 5th Berkeley Symposium on Mathematical Statistics and Probability*, 281-297.

### Books
1. Bishop, C.M. (2006). *Pattern Recognition and Machine Learning*. Springer. Chapter 9: Mixture Models and EM.
2. Hastie, T., et al. (2009). *The Elements of Statistical Learning* (2nd ed.). Springer. Chapter 14: Unsupervised Learning.
3. Murphy, K.P. (2012). *Machine Learning: A Probabilistic Perspective*. MIT Press. Chapter 25: Clustering.

### Online Resources
1. [K-Means Clustering - Wikipedia](https://en.wikipedia.org/wiki/K-means_clustering)
2. [K-Means - Scikit-learn Documentation](https://scikit-learn.org/stable/modules/clustering.html#k-means)
3. [Visualizing K-Means](https://www.naftaliharris.com/blog/visualizing-k-means-clustering/)
4. [Stanford CS229 - K-Means](http://cs229.stanford.edu/notes/cs229-notes7a.pdf)

### Implementation
- Source: `src/general/kmeans.rs`
- Tests: Included in source file under `#[cfg(test)] mod tests`
