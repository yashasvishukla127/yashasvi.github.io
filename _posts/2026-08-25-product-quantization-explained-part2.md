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

## Step 7
![splitting querry embedding into 96 chunks ]({{ site.baseurl }}/assets/images/2.7.png)

we pick querry's chunk 1st/96 - measure the distance against each ofchunk 1's256 centroids which we made atstep 3( check prev blog )
we store all the distance we have measured for querry chunk 1 against all 256 centroids in a LOOKUP table_1 (256 entries).

 ## Step 8
 
 distance measuring against each centroid for chunk 2 ,chunk 3 .....chunk 96 
![distance measuring against each chunk ]({{ site.baseurl }}/assets/images/2.8.png)
  we repeat the same process for
querry's chunk 2nd/96 - measured against chunks 2's 256 centroids ( made at step 3)- LOOKUP_table_2 with 256 entries is created for this chunk 2 also
then qeurry's chunk 3/96 - LOOKUP_table_3 created
...
till querry's chunk 96/96 - LOOKUP_table_96 created
we do it once for a querry we recieve every time

  ## Step 9 
 important step here !! every thing combines and gives us the result 
  ![using lookup table to get distances for vector #1  ]({{ site.baseurl }}/assets/images/2.9.png)

   things are bit complex here read carefully 2-3 times -
remember these 2 points
we have compression of 96 array reference (MADE in 5th step ), during compression stage .
we have a 96 lookup_table created at prev step(step 8) .
we pick 1st vector array from 1M vectors (see step 5)
it have 96 array reference
at array reference 1/96 the code was e.g 17
we check lookup table_1 for no.17 - we have a calculated distance already in prev. step we store that distance
then array reference 2/96 the code was lets say 4 ,
we check with lookUP_table_2 in lookup table for id 4 we save the distance
then compressed array reference 3/96 we check with lookup_table_3 we store the distance
..
..
we continue this for array reference 96/96 we check till lookup_table_96 we store the distance
 


