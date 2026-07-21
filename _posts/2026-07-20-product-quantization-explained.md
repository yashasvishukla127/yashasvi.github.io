---
layout: post
title: "Product Quantization Explained"
date: 2026-07-20
---

# Product Quantization Explained
While building my agentic architecture, I had to choose between pgvector, ChromaDB, Qdrant, and Pinecone. I started studying where each one performs well and where it falls short.

One thing I kept coming across was that pgvector starts consuming a lot of memory once your dataset grows to millions of vectors. That's when people often switch to Qdrant.

I became curious: what is Qdrant doing under the hood that makes it so much more memory-efficient?

That question led me to Product Quantization (PQ).

pgvector works well for smaller datasets, but once you have millions of vectors, memory usage increases rapidly because every vector is stored in full precision. Qdrant handles the same scale much more efficiently using a compression technique called Product Quantization (PQ). Understanding how PQ works took me much longer than I expected.

![Product Quantization Overview]({{ site.baseurl }}/assets/images/001.png)

 
What an embedding actually is. One document → 1 vector of N numbers (e.g. 1536). Each number is a coordinate along a learned direction — not spatial like x/y/z, but the mathematical generalization of it.

Why compression is needed at all. we have now 1 million vectors × 1536 floats × 4 bytes = ~6GB just for one field, in memory its a large number . This is the "why should I care" bridge before you go into HOW.

step 2
---

![What an Embedding Looks Like]({{ site.baseurl }}/assets/images/02.png)
we pick 1st vector and slice it into 96 sub-vectors of 16 numbers
This split is arbitrary—it isn't intrinsic to the data. It's simply a design choice that makes clustering computationally tractable.


keep in mind
-
![What an Embedding Looks Like]({{ site.baseurl }}/assets/images/03.png)

keep in mind 1st vector's chunk 1 is in dimension 1-16,
similarly 2nd vector's chunk 1 is also in dimension 1-16,
it goes on 1MNth vector - chunk 1 is in dimension 1-16
i.e chunk 1 of each vector is in same dimension i.e (1-16)

similarly 1st vector's chunk 2 is in dimension 17-32
2nd vector's chunk 2 is also in dimension 17-32
till 1millionTH vector chunk 2 will be in dimension 17-32
.
.
.
similarly 1st vector's chunk 96 is in dimension 1521-1536
2nd vector's chunk 2 is also in dimension 1521-1536
till 1millionTH vector chunk 2 will be in dimension 1521-1536

idea here is each similar chunk from each vector lives in similar dimensional space


**3rd step**
--
![What an Embedding Looks Like]({{ site.baseurl }}/assets/images/05.png)

Next, we take Chunk 1 from every vector. Since each Chunk 1 represents dimensions 1–16, we now have 1 million 16-dimensional chunks. We then run k-means clustering on these chunks.

K-means groups together chunks that are close to each other in the vector space (i.e., the distance between them is small). For each group, it computes the average of all the chunks in that group. This average becomes the cluster centroid.

We repeat this process until 256 cluster centroids are created for Chunk 1.



![What an Embedding Looks Like]({{ site.baseurl }}/assets/images/06.png)
now we repeat the same for chunk 2 of ALL 1M vectors
then for chunk 3 ........till chunk 96

we have 256 cluster for each 96 chunks

so far we had 1 million vectors , each vector of 1536 dimension ,
we split each vector into 96chunks * 16spaces

we took each chunk 1 formed 256 clusters
we took each chunk 2 formed 256 clusters
.
.
we took chunk 96 formed 256 clusters

now

the compression begins


**4th step**
--
![What an Embedding Looks Like]({{ site.baseurl }}/assets/images/08.png)

remember at second step we have divided each vector into 96 chunks

now read carefully
**we pick up vector 1**
see step 2
we pick chunk 1 (from step 2) we compare it with chunk

1's 256 centroids (which we made in prev step)
and check **chunk 1** is closest to which centroid among these 256
centroids

say it was close to cluster 33 , we store this **id 33**

we are still at **vector 1**
now we pick **chunk **2 (from step 2 ) we we compare it with chunk

2's 256 centroids (which we made in prev step) not any other chunk's
centroids

we found it was close to say cluster 221 , we store this **id 221**

we repeat the same process till **96th chunk**

so we get a compressed pq 96 ids for vector 1
**[33,221,88.......144]**

**5th step** 
--

now we move on to vector 2
we repeat the same process
we pick up **chunk 1 (from step 2) we compare it with chunk

1's 256 centroids (which we made in step #3)
and check chunk 1 is closest to which centroid among these 256
centroids

say it was close to cluster 17 , we store this id 17

we are still at vector 2
now we pick chunk 2 (from step 2 ) we we compare it with chunk

2's 256 centroids (which we made in step 3) not any other chunk's
centroids

we found it was close to say cluster 4 , we store this id 4
we got ids for vector 2
**[17,4,190......,9]**

we repeat the same process till** 96th chunk**


![What an Embedding Looks Like]({{ site.baseurl }}/assets/images/09.png)


and we repeat this for vector 3 then vector 4 till..... 1M vectors

Compression = replace values with IDs. For a given vector, chunk 1's actual 16 numbers are replaced with the ID of the nearest centroid in chunk 1's codebook (using Euclidean distance). That ID is a number between 0 and 255, so it fits in a single byte instead of storing 16 float32 values (64 bytes).


![What an Embedding Looks Like]({{ site.baseurl }}/assets/images/10.png)

Repeat this for all 96 chunks, and the vector shrinks from 6144 bytes (1536 × 4) to 96 bytes—roughly 64× smaller.

> **6144 bytes → 96 bytes (~64× compression)**

Of course, something is lost. Product Quantization is lossy, so distance calculations become approximate rather than exact. The tradeoff is lower memory usage at the cost of some recall. Systems like Qdrant mitigate this by rescoring the top-K candidates using the original full-precision vectors, recovering much of the lost accuracy.

in next blog i would talk about after compressing , how we are going to fetch fast if any querry comes in

i have given my many hours understanding it don't expect you would understand in 1st go read it several times ,you would surely understand in 1/50th time i took to understand after reading this .





