# Installation des dépendances

```
npm install
```

# Création de la base de données

```
createdb "apicarto-aoc"
psql -d "apicarto-aoc" -c "CREATE EXTENSION postgis"
```

# Chargement des données

## Table appellation

```
ogr2ogr -overwrite -f "ESRI Shapefile" data/Appellation.shp data/Appellation.TAB
shp2pgsql -d -g geom -s 2154 -W CP1252 data/Appellation.shp appellation > data/Appellation.sql
psql --quiet -d apicarto-aoc -f data/Appellation.sql
```

TODO GEOM_TYPE=MultiPolygon?

Voir [http://www.gdal.org/drv_pgdump.html](PGDUMP) pour plus de détail

## Table communes

```
shp2pgsql -d -g geom -s 4326 data/communes-20150101-5m.shp communes > data/communes.sql
psql --quiet -d apicarto-aoc -f data/communes.sql
```


# Configurer le service


Voir config/default.json pour les paramètres de la BDD


# Lancer le service

```
node index.js
```

# "Tester" le service

```
curl -X POST -F "geom=@tests/geom1.geojson" http://localhost:8091/aoc/api/beta/aoc/in > features.geojson
```

(features.geojson est une FeatureCollection)


# Requête :

Les paramètres sont des Feature GeoJSON avec des géométries en coordonnées?


Note pour récupérer une géométrie de test :

```
SELECT ST_AsGeoJSON(
    ST_Transform(
        ST_SetSRID(
            ST_Buffer(ST_Centroid(geom),50.0),
            2154
        ),
        4326
    ))
FROM appellation LIMIT 1
```

=>


La feature associée :

{
    "type":"Feature",    "geometry":{"type":"Polygon","coordinates":[[[4.79628704723532,45.2245686201141],[4.79627198205696,45.2244810075761],[4.79623301978841,45.2243971554783],[4.7961716578197,45.2243202861855],[4.7960902543159,45.2242533537068],[4.79599193758517,45.2241989301793],[4.795880485857,45.2241591070292],[4.79576018209113,45.2241354146047],[4.79563564939616,45.2241287633715],[4.79551167338089,45.2241394089266],[4.79539301826325,45.224166942177],[4.79528424380096,45.2242103050595],[4.79518953007658,45.2242678311979],[4.79511251686827,45.2243373099351],[4.79505616377777,45.224416071282],[4.79502263649038,45.224501088517],[4.79501322353864,45.2245890944965],[4.79502828676983,45.2246767072062],[4.79506724742323,45.2247605597291],[4.7951286083547,45.2248374296355],[4.79521001155707,45.2249043628231],[4.79530832876809,45.2249587870468],[4.79541978168514,45.2249986107744],[4.79554008716747,45.2250223035697],[4.79566462184518,45.2250289549107],[4.79578859980762,45.2250183091839],[4.79590725654041,45.2249907755084],[4.79601603203994,45.2249474120122],[4.79611074606575,45.2248898851649],[4.79618775879377,45.2248204057315],[4.7962441106954,45.2247416438072],[4.79627763626632,45.2246566262013],[4.79628704723532,45.2245686201141]]]}
}


# TODO

* Monter les géométries en coordonnées EPSG:4326
* Vérifier les encodages des caractères (UTF-8 en sortie des services)
* config/default.json.dist (l'autre en .gitignore)
* Des tests
