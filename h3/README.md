# H3

H3 ist ein System für die räumliche Indizierung. Dabei wird die Welt in ein **regelmässiges hierarchisches Hexagon-Gitter** unterteilt.

H3 enthält Funktionen zur Konvertierung von Breiten- und Längenkoordinaten in H3-Zellen, sowie auch zur Berechnung des Zentrums jeder Zelle, um die Nachbarn zu ermitteln, und mehr.

Dabei ist das Gitternetz **hierarchisch** aufgebaut. Eine hexagonale Zelle enthält ungefähr sieben Zellen der daruntliegenden Hierarchie. Man spricht hier von *Parents* und *Children*, also Eltern und deren Kinder.

<img src="https://h3geo.org/images/parent-child.png" style="max-width: 500px" /><br />
<small>Quelle: https://h3geo.org/images/parent-child.png</small>

Die obige Abbildung zeigt hexagonale Gitternetz-Struktur mit einem Parent und der darunterliegenden Hierarchie. Die Hierarchie-Stufen sind ähnlich zu den Raster-Tiles in einer Webkarte.

Das Gitternetz kann dabei den Raum auch durch variable Hierarchien abdecken:

<img src="https://h3geo.org/images/ca_compact_6_901.png" style="max-width: 500px" /><br />
<small>Quelle: https://h3geo.org/images/ca_compact_6_901.png</small>

H3 ist natürlich nicht das einzige räumliche Indizierungssystem. Ein anderes solches System ist **[S2](https://s2geometry.io/)**, das die Welt in ein hierarchisches Gitternetz aus quadratischen Zellen. Ein hexagonales Gitter hat jedoch interessante Eigenschaften z.B. für das Buffering, da die Nachbarn einer einzelnen Zelle ungefähr einen Kreis bilden. Auch **[Geohash](https://en.wikipedia.org/wiki/Geohash)** ist ein alternatives Indizierungssystem.

Solche Gitternetze können interessant sein gerade im Zusammenhang mit Big Data, da die ursprüngliche Datenmenge relativ einfach reduziert werden kann, ohne dass man die Option auf Vergleiche mit anderen Layers verliert.

## H3 mit GeoPandas

Schauen wir uns hier konkret an, wie H3 zusammen mit GeoPandas verwendet werden kann. Dabei verwenden wir die Python-Bibliothek [H3-Pandas](http://h3-pandas.readthedocs.io/).

Die Installation geschieht falls nötig ganz normal mit `pip`:

```bash
pip install h3 h3pandas
```

Die nötigen Python-Bibliotheken können nun geladen werden:

```python
import pandas as pd
import matplotlib.pyplot as plt
import h3pandas
```

## Daten laden und vorbereiten

Irgendwelche Punktdaten sind am Besten geeignet. Wir brauchen einfach geographische Koordinaten (im WGS84).

Wir nehmen hier das NYC Taxi Trip Duration-Dataset mit Pandas:

```python
taxi_trips = pd.read_csv('../data/nyc-taxi-trip-duration/train.csv')
```

Wir können den Inhalt einfach inspizieren:

```python
taxi_trips.head()
```

Wir benennen die Koordinaten-Spalten in `lat` und `lng` um, und eliminieren die Spalten, die wir nicht brauchen:


```python
df = taxi_trips.rename({ 'pickup_longitude': 'lng', 'pickup_latitude': 'lat' }, axis = 1)
df = df[['lng', 'lat', 'passenger_count']]
```

Es ist immer eine gute Idee, die Daten zu überprüfen. Schauen wir uns die Maximum- und Minimum-Koordinaten an:

```python
{
    'xmin': df['lng'].min(),
    'xmax': df['lng'].max(),
    'ymin': df['lat'].min(),
    'ymax': df['lat'].max()
}
```

Dabei gibt es  ganz offensichtlich einige Werte, die nicht stimmen können. Diese können wir ungefähr aufgrund der Koordinatenwerte herausfiltern:

```python
df = df.loc[
    (df['lng'] > -74.3) & (df['lng'] < -73.7) & 
    (df['lat'] > 40.4) & (df['lat'] < 41.0)
]
```

Wir wandeln die räumlichen Koordinaten in H3-Koordinaten um. Dabei geben wir die Auflösung an:


```python
taxis_h3 = df.h3.geo_to_h3(10)
taxis_h3.head()
```

Wir haben immer noch ein einfaches Pandas-DataFrame. Zum Überprüfen:

```python
type(taxis_h3)
```

Die obige Operation geht natürlich auch mit GeoPandas-DataFrames, also auf der Basis der Geometrie.

Nun können wir einfach die Anzahl Passagiere pro hexagonale Zelle berechnen. Dazu können wir einfach die Koordinaten eliminieren:


```python
taxis_h3 = taxis_h3.drop(columns = ['lat', 'lng']).groupby('h3_10').sum()
taxis_h3.head()
```

Um eine sinnvolle Visualisierung zu bekommen, brauchen wir natürlich noch eine Geometrie. Die bekommen wir mit `h3_to_geo_boundary`:


```python
taxis_h3 = taxis_h3.h3.h3_to_geo_boundary()
taxis_h3.head()
```

Nun haben wir ein GeoPandas-DataFrame, und wir können eine einfache Karte machen:

```python
taxis_h3.plot(column = 'passenger_count', figsize = (10, 10))
```

Wir können auch die Hierarchiestufe nachträglich noch ändern, mit `h3_to_parent`:


Wir müssen den Index auf die Hierarchiestufe 9 ändern und gleichzeitig die Summe der Passagierzahlen neu berechnen:


```python
taxis_h3_9 = taxis_h3_9.set_index('h3_09') \
                       .groupby('h3_09') \
                       .sum(numeric_only = True)

taxis_h3_9.head()
```


Danach können wir die Hexagon-Polygone wieder hinzufügen und eine Karte erstellen:


```python
taxis_h3_9 = taxis_h3_9.h3.h3_to_geo_boundary()
taxis_h3_9.plot(column = 'passenger_count', figsize = (10, 10))
```


## Räumliche Glättung

Das hexagonale Gitter hat Vorteile bei gewissen räumlichen Operationen. So kann zum Beispiel eine räumliche Glättung einfach umgesetzt werden. Es genügt, die Anzahl Nachbaren zu definieren, die für die Glättung miteinbezogen werden sollen:

```python
taxis_h3_9_s2 = taxis_h3_9.h3.k_ring_smoothing(2)
taxis_h3_9_s2.head()
```

Eine kleine Karte der geglätteten Werte kann dann so erstellt werden:

```python
taxis_h3_9_s2.plot(
    figsize = (10, 10),
    column = 'passenger_count',
    cmap = 'plasma',
    edgecolor = 'none',
    legend = True
)
```
