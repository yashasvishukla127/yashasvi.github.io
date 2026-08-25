---
title: "Product Quantization part 2- how querry gets fetched"
date: 2026-08-25
tags: [ai, vectordatabase, machinelearning]
---
In last blog i have talked about how document is stored via the process of product quantization - which makes querrying faster .
we saw how an approximately 6 GB vector dataset containing one million vectors was compressed to a much smaller size.

## step 6

**continuing from the last blog**

![user querry getting converted into embedding]({{ site.baseurl }}/assets/images/2.6.png)

when querry comes in first it's converted into embeddings - from that same model which have converted the doc into embedding each embedding is converted into 96 chunks of 16 dimensions
