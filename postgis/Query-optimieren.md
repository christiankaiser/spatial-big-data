## PostGIS optimieren

Auf dem SDS-Server befindet sich im `data/lavaux`-Ordner ein SQL-Dump. Wir können den Dump in eine PostGIS-Datenbank lesen.

Dies kann nach einem SSH-Zugriff über die Shell geschehen mit dem `psql`-Tool, z.B.

```bash
psql db0000 < lavaux.sql
```

Dies kann auch durch ein Python-Skript geschehen, z.B. in einem Jupyter Notebook. Die `psycopg`-Bibliothek wird dafür zur Verbindung mit PostgreSQL verwendet.

```python

```

Nach dem Laden des SQL-Dumps haben wir zwei Tabellen in der Datenbank:

1. **statpop**: diese Tabelle enthält Bevölkerungsdaten für die ganze Schweiz. Eine Zeile entstpricht einem Hektar. Die Koordinaten befinden sich in den Spalten `x` und `y` im Koordinatensystem `CH1903 / LV03` (EPSG-Code 21781).

2. **lpm**: diese Tabelle enthält die durch die UNESCO geschützte Region in der Lavaux im Kanton Waadt. Dabei gibt es eine Kernzone, und eine Puffer-Zone. Diese Tabelle enthält eine PostGIS-Geometrie im Koordinatensystem `CH1903+ / LV95` (EPSG-Code 2056).


Wir wollen nun die Bevölkerungsdaten für die Kernzone der Lavaux herausfiltern. Dies kann anhand dieser Query geschehen:

```sql
SELECT reli, btot
FROM (
    SELECT
        reli,
        btot,
        ST_SetSrid(ST_MakePoint(x, y), 21781) AS geom
    FROM statpop
) s,
(
    SELECT ST_Transform(geom, 21781) AS geom
    FROM lpm
) b
WHERE ST_Intersects(b.geom, s.geom)
```

Versuchen Sie diese Query zu optimieren. Sie dürfen dabei auch neue Spalten zu den Tabellen hinzufügen, Indizes hinzufügen usw. Allerdings darf keine Spalte der Tabelle die Information enthalten, ob der Datenpunkt innerhalb des UNESCO-Perimeters liegt oder nicht.

Wieviel Zeit können Sie bei der Query gewinnen?
