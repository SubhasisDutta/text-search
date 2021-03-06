Adding this as an additional file to prevent overwriting someone else's changes to the docx file.

1. Questions:

    1. What you learned?

        1. The overall flow of a search engine as well as the reason why Google employs so many people.  I also didn't realize the amount of irrelevant data you have to find a way to work around.  On pretty much every stage of the process, data was introduced that was just not needed for the search engine.  We had to find ways to cut the amount of data down to a manageable amount.  Data relevant for searching (such as sales information or site specific information (Target Redcard, etc.) was useful for indexing, but completely unneeded or even hurtful to creating useful clusters.  In addition, even weka, which is widely seen as a standard way to do data analysis and clustering, struggled with the 100k+ pages we had, causing me to dive into the weka documentation and find the settings that allowed us to trim the data down more.  By the end of this trial and error process, I had found ways to trim down the clustering data set to just what was needed to provide additional relevant results.

    2. What was your experience?

        1. Lots of data flow experience.  Figuring out what was needed by whom and the best and most sensible way to get it to them.  In addition, we had to find ways to see everything whole picture because information not necessarily needed by one part may be important for others lated down the pipeline.  The amount of communication needed was more than other projects I've dealt with in an academic environment.

    3. What were difficulties you faced?

        1. Run time efficiencies.  Any time data was needed quickly, we were either unable or unwilling to trust the weka core modules to deliver the results in the time frame I needed.

    4. How did you resolve them?

        1. By building custom data structures that only contained the needed data in an easy to parse format, we were able to get our results as quickly as possible.  In addition, we utilized as much "offline" processing as we could, creating these custom data structures before the application started or overnight so they could be used when one of us wanted to actually test the full application.

5. Clustering

    1. Using the weka core modules, I prepared the data by removing stopwords and normalizing the data results.  I mostly used the java code included as loading the large data sets in weka GUI didn't work well (or at all).  In addition, word counts were transformed into IDF values.  A custom stop words list was used containing a large amount of stop words as well as commerce specific words that brought no relevance to the clusters.  This data formatting step was separate from what was needed for indexing and discovered as a result of using the clustering algorithms.

        a. I used the weka k-means model using Euclidean Distance with a max of 500 iterations.  Data was prepared as above.  I found with 100 clusters, I was able to use those as broad categories in order to introduce additional results to the final results as well as re-order the final results.

            i. 100 clusters were used.

    b. A relevance model was created to be used at runtime with the received search results.  I took an initial "rank" Score (result set size / rank) and added to that a Cluster Score (distance of document to its centroid / closest document distance to centroid).  The top result usually stayed the same (since the rank score was set to the set size / 1 = set size vs. 1/2 of the set size of the next result).  This was done with the assumption that the top returned result would probably be the most relevant.  In addition, every time a cluster appeared, the top results were either added or the rank score added to the new result's current score.  Additional results were added to the end of the document with just a cluster score.  This allowed good results that weren't near the top to be moved up significantly if it's a top result in a cluster that's the best cluster for the result set.  Even though I used distance to centroid as opposed to distance to original document, the clusters were small enough that the result was negligible on results, but much faster on implementing during searching.

    c. See 5.b.

    d. Both the new result rankings and k-means clusters used in the results were given to the front end.

    e. The hierarchical models were mostly used in a trial and error fashion to curate the data and choose the important attributes we used in the final search engine (or, rather, the least important attributes to include).  After building each cluster, I looked at what each level was being split on and determining attributes.  For the first few iterations, the words used for splitting weren't stop words exactly, but unimportant words for a commerce site (like "free shipping").  After adding to the stop words list with these terms, each clustering model (k-means and hierarchy) using this data set provided better results and more relevant clusters.   The way the hierarchical clusters split allowed me to determine a commerce specific stop word list.

    f. ~100 leaves per agglomerative clustering method.

    g. The improved cluster titles were generated in part by the hierarchical clusters and they were provided to the front end.  Without the addition of the improved stop words list, each title would be something like "Target Free Shipping".

    h. 500 randomly generated query were used along with 25 manual queries.  Test queries were generated using cluster.generateClusterQueries() and the results fed into our test suite.  Examining those results determined which attributes were left out of subsequent clustering runs.

    i. See 5.e.  The results obtained during testing were used to prune the data and improve the overall clustering during the next iteration.

    j. The random queries were used to test and 25 manual queries were based on what I think people would search for online.

    k1: Search Query "Pants"; Introduced and moved ot 4th place in results the page http://www.target.com/s/pink+mossimo+pants
    
    k2: Red Paper: Introduced and moved to 6th in results http://www.target.com/s/paper+towel+holder+red
    
    k3: Lotion: Moved around results and added http://www.walmart.com/ip/Johnson-s-Honey-Apple-Baby-Lotion-Choose-Your-Size/14675929?action=product_interest&action_type=title&beacon_version=1.0.1&bucket_id=irsbucketdefault&category=&client_guid=18de1fbc-599f-4370-aa21-224715065393&config_id=2&customer_id_enc=&findingMethod=p13n&guid=4481ab97-4022-4e43-be97-d46d9558654e&item_id=14675929&parent_anchor_item_id=10294057&parent_item_id=10294057&placement_id=irs-2-m3&strategy=PWVUB&visitor_id= as the second result.
    
    k4: Apple: Moved electronic components up and added iPhone accessories such as http://www.walmart.com/ip/OtterBox-Symmetry-Case-for-Apple-iPhone-5-5S-with-Clearly-Protected-Vibrant-Screen-Protector-Candy-Pink/39746447?action=product_interest&action_type=title&beacon_version=1.0.1&bucket_id=irsbucketdefault&category=&client_guid=bf9d9c77-f580-4336-8eb0-fbd770f92b19&config_id=2&customer_id_enc=&findingMethod=p13n&guid=ae2a49ba-7c2e-43bd-9632-754b41f090c6&item_id=39746447&parent_anchor_item_id=38078636&parent_item_id=38078636&placement_id=irs-2-m3&strategy=PWVUB&visitor_id=
    
        k4 Note: the results from k3 (Apple Scented Lotion) was moved down.  This is consistent with the clustering I saw, which had Apple more as electronics than fruit.  This matches what I would expect to buy online.
