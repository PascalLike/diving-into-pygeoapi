---
title: Metadata
---

# Metadata

## Sharing Records with pygeoapi

There are different options to share documents with OGC Record API. In this tutorial, we will create an index on ElasticSearch to store our documents and share that index as an OGC API Records collection.

First, we need to collect some data!

We can download some example ISO19139 xml from any CSW Catalog out there. For this workshop we will use the [GeoNetwork "Catalogo della Città Metropolitana di Firenze"](http://dati.cittametropolitana.fi.it/geonetwork/srv/ita/catalog.search#/home)

In the workshop `\docker\data\metadata\xml` folder, there are already some examples ready for use.

This data isn't completely ready to use because we have to convert it to GeoJSON first, a favoured format for pygeoapi. To convert this xml to geojson we can use [pygeometa](https://geopython.github.io/pygeometa/)

## Converting ISO19139 data to GeoJSON

In order not to have to download anything locally, we will proceed using a "disposable" docker container:

```bash
docker run -it --rm --name pygeometa-workspace -v "$PWD":/data ubuntu:latest
```
From the container shell we can setup the necessary tools

```bash
apt-get update
apt-get install python3 python3-pip git -y
python3 -m venv pygeometa
cd pygeometa
. bin/activate
pip install Cython pyproj
git clone https://github.com/geopython/pygeometa.git
cd pygeometa
python3 setup.py build
python3 setup.py install
```

And, finally, use python shell to do the conversion:

```bash
python


from pygeometa.schemas.iso19139 import ISO19139OutputSchema
import json

iso_os = ISO19139OutputSchema()
text_file = open("/data/xml/free-wifi-florence.xml", "r")
xml_string = text_file.read()
text_file.close()
json_data = iso_os.import_(xml_string)
out_file = open("/data/free-wifi-florence.geojson", "w")
out_file.write(json.dumps(json_data))
out_file.close()
```

## Push data to ElasticSearch

To push geojson to an index use

```bash
docker exec elastic /add_data.sh ./data/xml/free-wifi-florence.geojson geojson_id
```

## Configure record collection on pygeoapi

```
    example_catalog:
        type: collection
        title: FOSS4G Florence Record catalog
        description: FOSS4G Florence Record catalog (OGC API Records)
        keywords:
            - Services
            - Infrastructures
            - Florence
            - FOSS4G
        extents:
            spatial:
                bbox: [11.145, 43.718, 11.348, 43.84]
                crs: http://www.opengis.net/def/crs/OGC/1.3/CRS84
        providers:
            - type: record
              name: Elasticsearch
              data: http://elastic_search:9200/example_catalog
              id_field: metadata.identifier
              time_field: datetime
```