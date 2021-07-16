---
jupyter:
  jupytext:
    notebook_metadata_filter: all
    text_representation:
      extension: .md
      format_name: markdown
      format_version: "1.2"
      jupytext_version: 1.6.0
  kernelspec:
    display_name: Julia 1.6.0
    language: julia
    name: julia-1.6
  plotly:
    description:
      How to make a Mapbox Choropleth Map of US Counties in Julia with
      Plotly.
    display_as: maps
    language: julia
    layout: base
    name: Mapbox Choropleth Maps
    order: 1
    page_type: example_index
    permalink: julia/mapbox-county-choropleth/
    thumbnail: thumbnail/mapbox-choropleth.png
---

A [Choropleth Map](https://en.wikipedia.org/wiki/Choropleth_map) is a map composed of colored polygons. It is used to represent spatial variations of a quantity. This page documents how to build **tile-map** choropleth maps, but you can also build [**outline** choropleth maps using our non-Mapbox trace types](/julia/choropleth-maps).

Below we show how to create Choropleth Maps using either Plotly Express' `px.choropleth_mapbox` function or the lower-level `go.Choroplethmapbox` graph object.

#### Mapbox Access Tokens and Base Map Configuration

To plot on Mapbox maps with Plotly you _may_ need a Mapbox account and a public [Mapbox Access Token](https://www.mapbox.com/studio). See our [Mapbox Map Layers](/julia/mapbox-layers/) documentation for more information.

### Introduction: main parameters for choropleth tile maps

Making choropleth Mapbox maps requires two main types of input:

1. GeoJSON-formatted geometry information where each feature has either an `id` field or some identifying value in `properties`.
2. A list of values indexed by feature identifier.

The GeoJSON data is passed to the `geojson` argument, and the data is passed into the `color` argument of `px.choropleth_mapbox` (`z` if using `graph_objects`), in the same order as the IDs are passed into the `location` argument.

**Note** the `geojson` attribute can also be the URL to a GeoJSON file, which can speed up map rendering in certain cases.

#### GeoJSON with `feature.id`

Here we load a GeoJSON file containing the geometry information for US counties, where `feature.id` is a [FIPS code](https://en.wikipedia.org/wiki/FIPS_county_code).

<!-- ```python
from urllib.request import urlopen
import json
with urlopen('https://raw.githubusercontent.com/plotly/datasets/master/geojson-counties-fips.json') as response:
    counties = json.load(response)

counties["features"][0]
``` -->

```julia
using HTTP, JSON, DataFrames

url = "https://raw.githubusercontent.com/plotly/datasets/master/geojson-counties-fips.json"
resp = HTTP.get(url);
counties = JSON.parse(String(resp.body));

counties["features"][1]
```

#### Data indexed by `id`

Here we load unemployment data by county, also indexed by [FIPS code](https://en.wikipedia.org/wiki/FIPS_county_code).

<!-- ```python
import pandas as pd
df = pd.read_csv("https://raw.githubusercontent.com/plotly/datasets/master/fips-unemp-16.csv",
                   dtype={"fips": str})
df.head()
``` -->

```julia
using HTTP, CSV, DataFrames
df = CSV.File(
    HTTP.get("https://raw.githubusercontent.com/plotly/datasets/master/fips-unemp-16.csv").body
) |> DataFrame

```

### Choropleth map using plotly.express and carto base map (no token needed)

[Plotly Express](/python/plotly-express/) is the easy-to-use, high-level interface to Plotly, which [operates on a variety of types of data](/python/px-arguments/) and produces [easy-to-style figures](/python/styling-plotly-express/).

With `px.choropleth_mapbox`, each row of the DataFrame is represented as a region of the choropleth.

<!-- ```python
from urllib.request import urlopen
import json
with urlopen('https://raw.githubusercontent.com/plotly/datasets/master/geojson-counties-fips.json') as response:
    counties = json.load(response)

import pandas as pd
df = pd.read_csv("https://raw.githubusercontent.com/plotly/datasets/master/fips-unemp-16.csv",
                   dtype={"fips": str})

import plotly.express as px

fig = px.choropleth_mapbox(df, geojson=counties, locations='fips', color='unemp',
                           color_continuous_scale="Viridis",
                           range_color=(0, 12),
                           mapbox_style="carto-positron",
                           zoom=3, center = {"lat": 37.0902, "lon": -95.7129},
                           opacity=0.5,
                           labels={'unemp':'unemployment rate'}
                          )
fig.update_layout(margin={"r":0,"t":0,"l":0,"b":0})
fig.show()
``` -->

NOTE: Map just showing up empty

```julia
using PlotlyJS, CSV, JSON, HTTP, DataFrames

counties = JSON.parse(
    String(
        HTTP.get("https://raw.githubusercontent.com/plotly/datasets/master/geojson-counties-fips.json").body
    )
);

df = CSV.File(
    HTTP.get("https://raw.githubusercontent.com/plotly/datasets/master/fips-unemp-16.csv").body
) |> DataFrame

trace = choropleth(
    df,
    locations=:fips,
    color=:unemp,
    colorscale="Viridis",
    zoom=3,
    center=attr(lat=37.0902, lon=-95.7129),
    opacity=0.5,
    labels=attr(unemp="unemployment rate"),
    mapbox_style="carto-positron"
)
trace.fields[:geojson] = counties

plot(trace)
```

### Indexing by GeoJSON Properties

If the GeoJSON you are using either does not have an `id` field or you wish you use one of the keys in the `properties` field, you may use the `featureidkey` parameter to specify where to match the values of `locations`.

In the following GeoJSON object/data-file pairing, the values of `properties.district` match the values of the `district` column:

<!-- ```python
import plotly.express as px

df = px.data.election()
geojson = px.data.election_geojson()

print(df["district"][2])
print(geojson["features"][0]["properties"])
``` -->

<!-- NOTE: can't load either dataset -->

```julia
using PlotlyJS, CSV, DataFrames

df = dataset(DataFrame, "election")
geojson = dataset(JSON, "election_geojson")

df["district"][3]

geojson["features"][1]["properties"]
```

To use them together, we set `locations` to `district` and `featureidkey` to `"properties.district"`. The `color` is set to the number of votes by the candidate named Bergeron.

<!--
```python
import plotly.express as px

df = px.data.election()
geojson = px.data.election_geojson()

fig = px.choropleth_mapbox(df, geojson=geojson, color="Bergeron",
                           locations="district", featureidkey="properties.district",
                           center={"lat": 45.5517, "lon": -73.7073},
                           mapbox_style="carto-positron", zoom=9)
fig.update_layout(margin={"r":0,"t":0,"l":0,"b":0})
fig.show()
``` -->

<!-- NOTE: Can't load datasets -->

```julia
using PlotlyJS, CSV, DataFrames

df = dataset(DataFrame, "election")
geojson = dataset(JSON, "election_geojson")

layout = Layout(margin=attr(r=0, t=0, ;=0, b=0))

trace = choropleth(
    df,
    color=:Bergeron,
    locations=:district,
    featureidkey="properties.district",
    center=attr(
        lat=45.5517,
        lon=-73.7073,
        mapbox_style="carto-positron",
        zoom=9
    )
)
trace.fields[:geojson] = counties

plot(trace, layout)
```

### Discrete Colors

In addition to [continuous colors](/python/colorscales/), we can [discretely-color](/python/discrete-color/) our choropleth maps by setting `color` to a non-numerical column, like the name of the winner of an election.

<!-- ```python
import plotly.express as px

df = px.data.election()
geojson = px.data.election_geojson()

fig = px.choropleth_mapbox(df, geojson=geojson, color="winner",
                           locations="district", featureidkey="properties.district",
                           center={"lat": 45.5517, "lon": -73.7073},
                           mapbox_style="carto-positron", zoom=9)
fig.update_layout(margin={"r":0,"t":0,"l":0,"b":0})
fig.show()
``` -->

<!-- NOTE: Can't load datasets -->

```julia
using PlotlyJS, CSV, DataFrames

df = dataset(DataFrame, "election") # NOTE: Can't load this dataset
geojson = dataset(JSON, "election_geojson") # NOTE: Can't load this dataset

layout = Layout(margin=attr(r=0, t=0, ;=0, b=0))

trace = choropleth(
    df,
    color=:winner,
    locations=:district,
    featureidkey="properties.district",
    center=attr(
        lat=45.5517,
        lon=-73.7073,
        mapbox_style="carto-positron",
        zoom=9
    )
)
trace.fields[:geojson] = counties

plot(trace, layout)
```

<!-- NOTE: Not sure if this translates to julia... -->

#### Mapbox Light base map: free token needed

```python
token = open(".mapbox_token").read() # you will need your own token


from urllib.request import urlopen
import json
with urlopen('https://raw.githubusercontent.com/plotly/datasets/master/geojson-counties-fips.json') as response:
    counties = json.load(response)

import pandas as pd
df = pd.read_csv("https://raw.githubusercontent.com/plotly/datasets/master/fips-unemp-16.csv",
                   dtype={"fips": str})

import plotly.graph_objects as go

fig = go.Figure(go.Choroplethmapbox(geojson=counties, locations=df.fips, z=df.unemp,
                                    colorscale="Viridis", zmin=0, zmax=12, marker_line_width=0))
fig.update_layout(mapbox_style="light", mapbox_accesstoken=token,
                  mapbox_zoom=3, mapbox_center = {"lat": 37.0902, "lon": -95.7129})
fig.update_layout(margin={"r":0,"t":0,"l":0,"b":0})
fig.show()
```

#### Reference

See [function reference for `px.(choropleth_mapbox)`](https://plotly.com/python-api-reference/generated/plotly.express.choropleth_mapbox) or https://plotly.com/python/reference/choroplethmapbox/ for more information about mapbox and their attribute options.
