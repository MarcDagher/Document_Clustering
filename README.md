<h1 align="center">Document Clustering and Labeling</h1>

### Introduction
The task at hand involves clustering documents and assigning each cluster a title which represents the main idea of the documents within it. The dataset used contains the titles and the abstracts of articles. Data preprocessing included removing punctuation and non-letter characters to minimize noise in the data. After cleaning the data, I proceeded to represent the texts in more interpretable formats.

Word embeddings have revolutionized how we represent textual data for models. Instead of relying on keyword-based techniques, word embedding models transform texts into dense vectors, where vectors that are close together in space have similar meanings, and vectors that are farther apart have differing meanings. This numerical representation of words, which encapsulates semantic information, enriches NLP tasks by providing a deeper understanding of the real-world meanings of words.

To capture the semantics and word order in texts of varying lengths, I used Gensim’s Doc2Vec model, which implements the paragraph vector [1]. Doc2Vec transforms documents into embedding vectors using one of two approaches: Paragraph Vector - Distributed Memory (PV-DM) and Paragraph Vector - Distributed Bag of Words (PV-DBOW). PV-DM predicts a central word based on the average of both context word-vectors and the document-vector, while PV-DBOW predicts a target word solely from the document's doc-vector using classification. I chose the PV-DM approach, as it consistently delivers good results.

Since embedding vectors are high-dimensional, they can add noise and they will make the clustering process computationally intensive. To address this, I used a dimensionality reduction algorithm called Uniform Manifold Approximation and Projection (UMAP) [2] [3]. UMAP helps by reducing the complexity of the data while preserving its essential structure, making it more suitable for clustering. This technique is effective in maintaining both local and global data relationships, enabling more accurate clustering outcomes.

Hierarchical Clustering [4] is particularly advantageous for clustering documents and assigning meaningful titles. This algorithm does not require specifying the number of clusters in advance, allowing the exploration of different levels of clustering within the data. By building a graph hierarchy of nodes, it provides more flexibility with distance metrics and linkage criteria, enhancing its applicability to diverse data shapes and structures. This makes it well-suited for capturing the relationships between documents in our dataset. Using the clustered documents and the titles in our dataset, I used an LLM (ChatGPT) to generate meaningful and summarized titles for each cluster.

### Doc2Vec
To transform text into an interpretable format, various techniques can be used. Bag-of-Words converts a document into a vector representing the frequency of each word within the document. Bag-of-N-grams, on the other hand, represents a document as a vector of n-gram counts, where an n-gram is a sequence of a word followed by the next “n” words in the text. The main issue with these techniques is that Bag-of-Words does not capture the semantics of the words, results in very sparse vectors, and does not consider word location or context. Although n-grams can capture word location depending on the value of “n,” they still do not capture word semantics and also produce sparse vectors.

A more advanced technique is the Word2Vec model for word representation. When applied to a document, Word2Vec can generate a weighted average of all the words in the document. While this approach captures word semantics in dense vectors, it suffers from the same problem as Bag-of-Words: the loss of word order.

For our document clustering task, we used the paragraph vector model, proposed by Quoc Le and Tomas Mikolov [1], an unsupervised model that learns vector representations for texts, sentences, or documents. This model has two approaches for learning vectors from documents. The first is Distributed Memory of Paragraph Vectors (PV-DM), which uses a matrix representing the paragraph’s context and a matrix representing its words. For each paragraph, the context vector and word vectors are concatenated to predict the next word in the context. The context vector represents the missing information from the current context and can act as a memory of the topic of the paragraph, hence the name Distributed Memory. The second approach is the Distributed Bag of Words version of Paragraph Vector (PV-DBOW), which ignores the context words in the input. In PV-DBOW, during each iteration of stochastic gradient descent, the process involves sampling a text window, sampling a random word, and forming a classification task using the paragraph vector to predict the next word.

### Uniform Manifold Approximation and Projection (UMAP)
To make the high-dimensional embeddings generated by Doc2Vec more manageable and suitable for clustering, we used Uniform Manifold Approximation and Projection (UMAP) [2] [3], a dimensionality reduction technique. UMAP is a manifold learning technique that embeds a higher-dimensional space into a lower-dimensional one in its entirety, while preserving the data’s structure. UMAP seeks to preserve both local and global structure, making it more useful when preparing data for clustering [5].

UMAP works by constructing a graph of the data points in the high-dimensional space, where the edges represent the similarity between points. This graph is then projected into a lower-dimensional space in a way that attempts to preserve the local and global structures of the data. UMAP seeks to preserve both local and global structure, meaning it tries to maintain the distances between nearby points while also respecting the overall shape of the data distribution. A major added value for UMAP is its flexibility. The algorithm’s parameters can be tuned to emphasize either local or global structures, depending on the specific needs of the task.

One of the key advantages of UMAP is its ability to handle large datasets efficiently, making it ideal for tasks like document clustering. Unlike Principal Component Analysis (PCA), which assumes linear relationships in the data, UMAP captures non-linear structures, providing a more meaningful embedding space for complex data like text. After applying UMAP, the resulting lower-dimensional embeddings are more suitable for clustering, as they retain the essential relationships between documents while significantly reducing computational complexity.

### Hierarchical Clustering
Hierarchical clustering is a method of clustering that builds a hierarchal graph of clusters. There are two types of this method: Agglomerative and Divisive. Agglomerative method is a “bottom-up” approach where each data point is treated as its own cluster in the beginning and as we move from bottom to top, each data point is merged into pairs, and pairs are merged into clusters. The divisive method is a “top-down” approach. All observations start in one cluster, and splits are performed recursively as we move from top to bottom. In both approaches, nodes are merged based on the distance between them. For our task, we used SciPy’s linkage method which implements the agglomerative approach, where mutually closest nodes are turned into a new node until there is one final node left, representing the entire data set [4]. The measurement of distance between nodes can be done using single, complete, average, weighted, Ward, centroid and median (WPGMC) linkage. We used the centroid linkage method for agglomerative clustering, with a criterion of a maximum of 5 clusters. This means the algorithm groups the data into clusters by merging them based on the centroid of each cluster, and the final result is constrained to no more than 5 clusters.

### Cluster Labeling using ChatGPT
After preprocessing the documents and converting them into document vectors using Doc2Vec, we performed hierarchical clustering on these vectors. We then utilized the large language model, ChatGPT, to assign labels to each cluster. By sorting the data into clusters and providing ChatGPT with the title of each article, we prompted it to generate a general title that represents each cluster. To assess the accuracy and relevance of the cluster labels, we reviewed the results manually.

