# Die Spark-Session

Alle PySpark-Programme müssen eine `SparkSession` besitzen. Dies ist die Schnittstelle zum Spark-Cluster.

Wenn wir PySpark lokal von der Shell aus starten, wird die `SparkSession` automatisch erstellt und ist in der Variable `spark` zugänglich.

Ansonsten muss die Spark-Session manuell erstellt werden. Im einfachsten Fall sieht dies dann so aus:

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.getOrCreate()
```

In der Regel wird noch ein Applikationsnamen definiert sowie das zu verwendende Cluster explizit angegeben. Dies sieht dann in etwa so aus:

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName('SparkAnwendung').master('local[4]').getOrCreate()
```

## Spark-Session überprüfen

Um die Spark-Session zu überprüfen können wir eine CSV-Datei öffnen und einen Teil des Inhalts anzeigen. Dazu verwenden wir das New York City Trip Duration-Dataset, das auf dem SDS zur Verfügung steht:

```python
trips = spark.read.format('csv').option('header', True) \
    .load('../data/nyc-taxi-trip-duration/train.csv')

trips.limit(5).toPandas()
```
