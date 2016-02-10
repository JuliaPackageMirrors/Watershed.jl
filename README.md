This repository is a fork from [Seung lab watershed repository(https://github.com/seung-lab/watershed/tree/master/src-julia).

Hierarchical watershed segmentation
=======
Given an *affinity graph* as input, return as output a segmentation into *watershed basins* and a hierarchy generated by single linkage clustering. The segmentation is an indexed image, with nonnegative integer values running from 0 to the number of basins. The hierarchy can be represented by a *dendrogram*,  a tree in which leaves represent watershed basins, and internal vertices represent mergings of clusters. The height of each vertex is the affinity at which the merging occurs. 

The vanilla version of the algorithm typically produces severe oversegmentation, basins that are excessively numerous and/or small. Options are provided to moderate this tendency in two ways:

1. Collapsing all subtrees above a height (*high threshold*) to single leaves. The resulting dendrogram defines a hierarchy on segments that are unions of watershed basins.
2. Collapsing subtrees to eliminate leaves below a *size threshold* (but only for vertices above some height).

After collapsing subtrees, the regions of the segmentation are all mergings of watershed basins, and the leaves of the dendrogram represent these regions. 

This implementation is for an affinity graph associated with a 3D image. Each vertex of the graph is an image voxel, and each edge between nearest neighbor pairs of voxels. A voxel has 6 nearest neighbors in the x,y,z,-x,-y, and -z directions, and the graph is said to have 6-connectivity.  The weight of an edge represents the “affinity” of two voxels for each other.  High affinity voxels tend to end up in the same segment, and low affinity voxels in different segments. 

Watershed basins are associated with local *maxima* of the graph, following the sign convention from graph partitioning in which edges between segments are minimized (and therefore edges within segments are maximized). This is potentially confusing, as the sign convention is the opposite from the original watershed definition in image processing.  In that field, watershed basins are associated with local minima, in accord with the metaphor of a "drop of water flowing downhill."

The affinity graph is represented by three images, because the number of edges in the affinity graph is three times the number of voxels in the image (neglecting boundary effects).  We use convolutional networks to generate the affinity graph from the image [Turaga et al. 2010], but other algorithms may be used.  The simplest case is an affinity graph in which the weight of each edge is the maximum of its two voxel values. Then the watershed basins are associated with local maxima of the image, and our watershed on graphs reduces to the conventional watershed on images (except for the flipped sign).

In image processing, many watershed implementations identify the ridgelines that separate basins, and assign the label "0" to voxels on ridgelines.  Our watershed does not do this, because the ridgeline between two basins is regarded as being located on edges rather than voxels. However, our watershed can be made to label some voxels as background as follows.  Edges of the graph with affinity below a *low threshold* are removed. After this operation, some voxels become singletons, completely disconnected from the rest of the graph. These background voxels are given a label of "0" to distinguish them from foreground regions.

Julia translation of Zlateski's C++ code.

See `watershed.jl` for example of how to use functions.
