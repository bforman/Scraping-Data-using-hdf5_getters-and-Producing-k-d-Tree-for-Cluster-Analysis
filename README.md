# Building-on-getTrackData.py- Producing-k-d-Tree-for-Cluster-Analysis

#Problem/Approach
This research module builds off of the getTrackData.py script explained in my previous research module. Up to that point I had used glob in concordance with other modules such as walk(), path(), and join() to link the thousands of files in the EchoNest MSD subset directory together. I then used helper methods from the hdf5_getters.py script in order to scrape the metadata from the .h5 files within the directory, which I then placed as vectors (consisting of artist, song_name, latitude, and longitude) into a list that would hold all the information. Once I had all the information I needed from the files in one place, I then needed to think of a structure I could use to organize the data in a way in which each item would be stored as a node. I wanted to produce nodes based on latitude/longitude pairs, so that I could then compute euclidean distance calculations to identify nearest neighbors within a geographical range and cluster those nodes together. These lat/long pairs could also later be used as a form of hash key back into the tracks list to determine which artist/song_name pair matched the lat/long coordinates, which in consideration of the World Music map would serve useful for displaying the artist and song name for whatever song may be playing. The structure I found that suited the desired functionability was a k-d tree. A kd-tree is a binary space-partitioning data tree in which every node serves as a dimensional point. Every non-leaf node generates a split in the dimensional space called a half-space. K-d trees are highly efficient in applications involving multidimensional searches, such as a range search or nearest neighbour search, which is why the k-d tree serves as a great data structure to use for this specific data set. Once I create a matrix with vectors at each index holding the lat/long pairs, I can produce the k-d tree with a specific number of leaves at the lowest level and then compute nearest neighbour caluclations to perform cluster analysis on the dataset.

#Process
The Scipy library of Python provides a spatial module that includes a function called KDTree, that given a list of data points will produce a k-d tree. There are many different modifiers that you can specify to customize the layout of the k-d tree to your advantage in a given application. I created a map (2D array), where each row (2 columns) would hold the longitude in the first index and latitude in the second index. I chose this approach as opposed to having a 1D array where each index was a 2 item vector holding the two values, because with each value in its respective index I can choose whether I want to cluster between latitude or longitude ranges. I went through the tracks list, retreiving the lat and long values from each 4 item vector and placing those values in their respective index in the map. Once the map was fully populated with all the lat/long values from the tracks list I was able to call the spatial.KDTree() function to produce the k-d tree. Then how I wanted to query the tree to compute nearest neighbour calculations was up for tests.

#Dependencies
To use the KDTree function along with the functions associated with it you will need to import
- scipy.spatial

#Example of Producing k-d tree
```
def produce_KDTree():
        W = np.zeros((200, 2))
        tracks = getTrackData('MillionSongSubset')
        for i in range(0, 200):
                if math.isnan(tracks[i][2]) != True or math.isnan(tracks[i][3]) != True:
                        W[i][0] = tracks[i][2]
                        W[i][1] = tracks[i][3]
                else:
                        W[i][0] = 0
                        W[i][1] = 0
        a = np.array(W.tolist())
        tree = spatial.KDTree(a, 10)
        nearest = tree.query([1,20], k=5)
```

#Code Explanation
Although there are 10,000 songs in the MSD subset, and therefore 10,000 vectors in the tracks list, I was only able to populate the map W with lat/long pairs from 200 tracks because of the recursion depth being exceeded when trying to populate W with all 10,000 songs. This serves as the reason for why W is a 200 x 2 matrix as opposed to a 10,000 x 2 matrix. For several of the songs within the 10,000 song subset of the MSD there were "nan" values for the latitude and/or longitude, so before placing a value in an index of W I had to first check to see whether each value was "nan" or not. A value that is "nan" would produce an error in creating the k-d tree. If the value was not "nan" then it would be placed in the respective index of W, otherwise a 0 would be filled in its place. Once W was populated, it needed to be converted into a list to be passed to the KDTree function of spatial. Then the K-d tree was successfully produced, and was enabled to be queried for nearest neighbor searches. My nearest neighbor search is still being fine-tuned to efficiently peform the task I desire to accomplish but this example displays the general syntax of how you would query the tree. There are other specifiers that can be included in the query call to calibrate the search. 
