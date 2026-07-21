# Product Quantization Explained

![Product Quantization Overview](/assets/images folder/01.png)

The pain, stated first. "pgvector chokes past ~1-5M vectors because it keeps full-precision float32 vectors in RAM. Qdrant survives that same scale — the difference is a compression trick called product quantization, and understanding it took me way longer than I expected."

---

![What an Embedding Looks Like](/assets/images folder/02.png)

What an embedding actually is. One document → one vector of N numbers (e.g. 1536). Each number is a coordinate along a learned direction — not spatial like x/y/z, but the mathematical generalization of it.

Why compression is needed at all. 1 million vectors × 1536 floats × 4 bytes = ~6GB just for one field, before indexes. This is the "why should I care" bridge before you go into HOW.

The chunking trick. Slice each 1536-dimensional vector into 96 sub-vectors of 16 numbers. This split is arbitrary—it isn't intrinsic to the data. It's simply a design choice that makes clustering computationally tractable.

Per-chunk k-means → codebooks. Take chunk 1 from all one million vectors, run k-means with **k = 256**, and you get 256 centroids. That's chunk 1's codebook. Repeat this independently for chunks 2 through 96. You now have **96 separate codebooks**, each with 256 centroids. This independence is the key idea that's easy to miss.

Compression = replace values with IDs. For a given vector, chunk 1's actual 16 numbers are replaced with the ID of the nearest centroid in chunk 1's codebook (using Euclidean distance). That ID is a number between 0 and 255, so it fits in a single byte instead of storing 16 float32 values (64 bytes). Repeat this for all 96 chunks, and the vector shrinks from **6144 bytes** (1536 × 4) to **96 bytes**—roughly **64× smaller**.

> **6144 bytes → 96 bytes (~64× compression)**

Of course, something is lost. Product Quantization is **lossy**, so distance calculations become approximate rather than exact. The tradeoff is lower memory usage at the cost of some recall. Systems like Qdrant mitigate this by rescoring the top-K candidates using the original full-precision vectors, recovering much of the lost accuracy.

For my P2 project, I chose **[your choice]** because it matches my current scale. Once my vector count reaches **[your threshold]**, I'd seriously reconsider and move to a Product Quantization-backed vector database like Qdrant.
