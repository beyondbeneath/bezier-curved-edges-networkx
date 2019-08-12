# Curved Bezier edges in NetworkX

| <img src="https://github.com/beyondbeneath/bezier-curved-edges-networkx/blob/master/example-atlassian.png" height=200px> | <img src="https://github.com/beyondbeneath/bezier-curved-edges-networkx/blob/master/example-fb.png" height=200px> | <img src="https://github.com/beyondbeneath/bezier-curved-edges-networkx/blob/master/example-got.png" height=200px> |
|---|---|---|

NB: code was originally written in July 2017, and first published here in April 2019.

This function (`curved_edges` in `curved_edges.py`) creates some curved Bezier edges for a NetworkX graph. The original motivation was to mimic the types of edge curves found in [Gephi](https://gephi.org/) when I was producing an [animation](https://www.atlassian.com/blog/inside-atlassian/teamwork-data-visualization) showing the ForceAtlas2 algorithm converging. While it now appears NetworkX offers some kind of curved edges (though only for directed graphs, via the `connectionstyle` property of `FancyArrowPatch`) this is substantially faster and more versatile.

This defaults to using Gephi's Bezier curve defintion, in which you travel `0.2` distance units (of the length of the line connecting the two nodes) from each node toward each other (magenta markers); then travel that same distance again but perpendicular to the connecting line (the red 'control points'). The nodes along with these control points, now define the curve by four `(x,y)` coordinates. 

<img src="https://github.com/beyondbeneath/bezier-curved-edges-networkx/blob/master/bezier.png">

## Dependencies

* [NetworkX](https://networkx.github.io/): `networkx==2.2`
* [Bezier](https://pypi.org/project/bezier/): `bezier==0.9.0`
* [ForceAtlas2](https://github.com/bhargavchippada/forceatlas2): `fa2==0.3.5` (for producing the layouts in the examples)

## Sample usage

### Facebook

The [Stanford SNAP](https://snap.stanford.edu/index.html) project hosts many network datasets, one of which is the [Facebook social network](https://snap.stanford.edu/data/egonets-Facebook.html) from 10 individuals.

```python
# Imports
import networkx as nx
import matplotlib.pyplot as plt
from matplotlib.collections import LineCollection
from fa2 import ForceAtlas2
from curved_edges import curved_edges

# Load the graph edges and compute the node positions using ForceAtlas2
G = nx.read_edgelist('facebook_combined.txt')
forceatlas2 = ForceAtlas2()
positions = forceatlas2.forceatlas2_networkx_layout(G, pos=None, iterations=50)

# Produce the curves
curves = curved_edges(G, positions)
lc = LineCollection(curves, color='w', alpha=0.05)

# Plot
plt.figure(figsize=(20,20))
nx.draw_networkx_nodes(G, positions, node_size=5, node_color='w', alpha=0.4)
plt.gca().add_collection(lc)
plt.tick_params(axis='both',which='both',bottom=False,left=False,labelbottom=False,labelleft=False)
plt.show()
```

<img src="https://github.com/beyondbeneath/bezier-curved-edges-networkx/blob/master/example-fb.png" width=600px>

### Game of Thrones

[This repository](https://github.com/mathbeveridge/gameofthrones) hosts character interaction data for Game of Thrones, with one edge file per season. We can combine all the files and plot the results, and note we can modify the width (as Gephi does) according to the weight of the edges too:

```python
# Imports
import networkx as nx
import matplotlib.pyplot as plt
from matplotlib.collections import LineCollection
from fa2 import ForceAtlas2
from curved_edges import curved_edges

# Concatenate seasons 1-7
got_data = 'got-s{}-edges.csv'
dfg = pd.DataFrame()
for i in range(7):
	df_current = pd.read_csv(got_data.format(i+1),
                           names=['source','target','weight','season'],
                           skiprows=1)
	dfg = pd.concat([dfg, df_current])
dfg.drop(['season'], axis=1, inplace=True)

# Group by the edges so they are not duplicated
dfgt = dfg.groupby(['source','target'], as_index=False).agg({'weight':'sum'})

# Remove some outliers
outliers = ['BLACK_JACK','KEGS','MULLY']
dfgt = dfgt[~dfgt.source.isin(outliers)&~dfgt.target.isin(outliers)]

# Load graph from pandas and calculate positions
G = nx.from_pandas_edgelist(dfgt, source='source', target='target', edge_attr='weight')
forceatlas2 = ForceAtlas2(edgeWeightInfluence=0)
positions = forceatlas2.forceatlas2_networkx_layout(G, pos=None, iterations=1000)

# Get curves
curves = curved_edges(G, positions)

# Make a matplotlib LineCollection - styled as you wish
weights = np.array([x[2]['weight'] for x in G.edges(data=True)])
widths = 0.5 * np.log(weights)
lc = LineCollection(curves, color='w', alpha=0.25, linewidths=widths)

# Plot
plt.figure(figsize=(10,10))
plt.gca().set_facecolor('k')
nx.draw_networkx_nodes(G, positions, node_size=10, node_color='w', alpha=0.5)
plt.gca().add_collection(lc)
plt.tick_params(axis='both',which='both',bottom=False,left=False,labelbottom=False,labelleft=False)
plt.show()
```

<img src="https://github.com/beyondbeneath/bezier-curved-edges-networkx/blob/master/example-got.png" width=600px>
