## Annoy

![Recursive split](https://raw.github.com/erikbern/annoy/master/ann.png)

## What is this?

Annoy ([Approximate Nearest Neighbors](http://en.wikipedia.org/wiki/Nearest_neighbor_search#Approximate_nearest_neighbor) Something Something) is a C++ library with Python bindings to search for points in space that are close to a given query point. It also creates large read-only file-based data structures that are mmapped into memory so that many processes may share the same data.

There's a couple of other libraries to do approximate nearest neighbor search, including [FLANN](https://github.com/mariusmuja/flann), etc. Other libraries may be both faster and more accurate, but there are one major difference that sets Annoy apart: it has the ability to **use static files as indexes**. In particular, this means you can **share index across processes**. Annoy also decouples creating indexes from loading them, so you can pass around indexes as files and map them into memory quickly. Another nice thing of Annoy is that it tries to minimize memory footprint so the indexes are quite small.

Why is this useful? If you want to find nearest neighbors and you have many CPU's, you only need the RAM to fit the index once. You can also pass around and distribute static files to use in production environment, in Hadoop jobs, etc. Any process will be able to load (mmap) the index into memory and will be able to do lookups immediately.

We use it at [Spotify](http://www.spotify.com/) for music recommendations. After running matrix factorization algorithms, every user/item can be represented as a vector in f-dimensional space. This library helps us search for similar users/items. We have many millions of tracks in a high-dimensional space, so memory usage is a prime concern.

Annoy was built by [Erik Bernhardsson](http://www.erikbern.com) in a couple of afternoons during [Hack Week](http://labs.spotify.com/2013/02/15/organizing-a-hack-week/).

## Summary of features

* Euclidean distance (squared) or cosine similarity (using the squared distance of the normalized vectors)
* Works better if you don't have too many dimensions (like <100)
* Small memory usage
* Lets you share memory between multiple processes
* Index creation is separate from lookup (in particular you can not add more items once the tree has been created)
* Native Python support

## Code example

```python

f = 40
t = AnnoyIndex(f)
for i in xrange(n):
    v = []
    for z in xrange(f):
        v.append(random.gauss(0, 1))
    t.add_item(i, v)

t.build(50) # 50 trees
t.save('test.tree')
    
# …

u = AnnoyIndex(f)
u.load('test.tree') # super fast, will just mmap the file
print u.get_nns_by_item(0, 1000) # will find the 1000 nearest neighbors
```

Right now it only accepts integers as identifiers for items. Note that it will allocate memory for max(id)+1 items because it generally assumes you will have items 0 … n.

## How does it work

Using [random projections](http://en.wikipedia.org/wiki/Locality-sensitive_hashing#Random_projection) and by building up a tree. At every intermediate node in the tree, a random hyperplane is chosen, which divides the space into two subspaces.

We do this k times so that we get a forest of trees. k has to be tuned to your need, by looking at what tradeoff you have between precision and performance. In practice k should probably be on the order of dimensionality.

## Source

It's all written in C++ with a handful of ugly optimizations for performance and memory usage. You have been warned :)

## Future stuff

* Better support for other languages
* More performance tweaks
* Figure what O and Y stand for in the backronym :)
