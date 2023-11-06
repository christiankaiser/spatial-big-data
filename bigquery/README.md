# Google BigQuery

BigQuery ist ein vollständig von Google verwaltetes Data Warehouse, das skalierbare Analysen über Petabytes von Daten ermöglicht. BigQuery unterstützt SQL-Abfragen und verfügt auch über integrierte Funktionen für maschinelles Lernen.


## Erste Schritte mit BigQuery

Wir können auf BigQuery einfach über die folgende URL zugreifen:
[https://console.cloud.google.com/bigquery](https://console.cloud.google.com/bigquery). Wir benötigen selbstverständlich ein Google-Konto. Dabei ist eine Gratisversion verfügbar, die BigQuery Sandbox. Dabei verfallen die Daten nach 60 Tagen. Mit einem Standard-Konto, bei dem eine Kreditkarte hinterlegt ist, entfallen die Begrenzungen der Sandbox.

In der BigQuery-Konsole muss mindestens ein Projekt definiert sein. Falls nicht, muss ein neues erstellt werden.

Es ist möglich, SQL-Abfragen direkt in der Web-Konsole auszuführen. Wir können auch auf über 200 öffentliche Datensätze zugreifen. So können wir zum Beispiel den `covi19_open_data`-Datensatz direkt mit SQL abfragen:

```sql
SELECT
    location_key,
    country_name,
    date,
    new_confirmed,
    new_deceased,
    cumulative_confirmed,
    cumulative_deceased,
    population
FROM `bigquery-public-data.covid19_open_data.covid19_open_data` c
WHERE date > '2021-10-01'
```

Anschliessend können wir die Anfrage oder die View abspeichern.


## Daten in BigQuery laden

Als erstes erstellen wir ein Dataset im Projekt. Dabei muss eine Dataset-ID angegeben werden, und der Datenstandort muss ausgewählt werden. `europe-west6` ist Zürich.

Daten können von verschiedenen Quellen hinzugefügt werden, unter anderem als Upload (z.B. einer CSV- oder JSON-Datei), von Google Cloud Storage, Google Drive, oder auch aus der Amazon- oder Azure-Cloud.

Wir können zum Beispiel eine CSV-Datei der Zürcher LoRa-Sensoren in BigQuery laden. Bei einer CSV-Datei ist das Komma das standardmässige Spaltentrennzeichen. In den erweiterten Optionen kann dies allenfalls geändert werden.

Bei der Verwendung eines Google Storage Buckets, müssen sich die BigQuery-Tabelle und der Bucket am gleichen Standort befinden. Buckets über mehrere Regionen stehen natürlich zur Verfügung.

Nach dem Laden kann die Tabelle in BigQuery-Abfragen verwendet werden.


## Räumliche Analysen

BigQuery unterstützt auch räumliche Daten, und Visualisierungen sind unter anderem mit BigQuery Geo Viz möglich. Details zu den geografischen Funktionen für BigQuery SQL sind in der Dokumentation verfügbar: [https://cloud.google.com/bigquery/docs/reference/standard-sql/geography_functions](https://cloud.google.com/bigquery/docs/reference/standard-sql/geography_functions).

Eine Einführung in räumliche Analysen steht in der BigQuery-Dokumentation zu Verfügung: [https://cloud.google.com/bigquery/docs/geospatial-data](https://cloud.google.com/bigquery/docs/geospatial-data).

Es ist wichtig zu wissen dass die räumlichen Funktionen in BigQuery mit WGS84-Koordinaten arbeiten. Die Geometrien sollten allenfalls vorgängig projeziert werden.


## Jupyter Notebooks und BigQuery

Jupyter Notebooks können direkt in der Google Cloud erstellt werden. Dies geschieht einfach in der Vertex AI Workbench. Dabei muss Die Vertex AI API zuerst aktiviert werden, und anschliessend die Notebook API.

BigQuery-Abfragen können einfach in einer Notebook-Zelle durchgeführt werden, indem am Anfang der Zelle `%%bigquery` angegeben wird.

Bei der Nutzung von Notebooks fallen Kosten an.
