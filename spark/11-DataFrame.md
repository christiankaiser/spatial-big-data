# DataFrames

## 1. Ziel der Übung

- Verstehen was ein DataFrame in Spark ist
- Einlesen von Daten mit Spark
- Einfache Datenmanipulationen mit Python durchführen


## 2. Datensätze in Spark

### 2.1 Resilient Distributed Datasets (RDD)

Eine Spark-Anwendung verarbeitet Daten verteilt in einem Cluster. Dabei werden die Daten in einer Art Sammlung von Element zusammengefasst und automatisch auf die verschiedenen Knoten des Clusters verteilt, um parallele und verteilte Operationen ausführen zu können. Diese Abstraktion der Daten wird Resilient Distributed Dataset, kurz RDD, genannt. Es handelt sich im Grunde um eine Datei, die auf einem Filesystem wie HDFS oder ähnlich gespeichert wird. Die Daten, die in einem RDD gespeichert sind, haben immer einen fixen Datentyp (man sagt sie sind *strongly typed*).

Es ist weiter wichtig anzumerken, dass ein RDD immer nur read-only ist, es können also nicht beliebige Daten verändert werden.

### 2.2 Datasets

Basierend auf den RDDs definiert Spark sogenannte *Datasets*. Ein Dataset ist dabei eine Datei mit Zeilen und Kolonnen, also eine Art Tabelle. Ein solches Dataset kann gefiltert werden, oder mit Funktionen wie `map` bearbeitet werden.

Datasets sind nur in Scala und Java direkt verfügbar, wird unter Python oder R aber auch nicht gebraucht.

### 2.3 DataFrames

Ein DataFrame ist ein Dataset mit benannten Spalten. Es ist konzeptuell identisch mit einer Datenbank-Tabelle, oder einem DataFrame in R oder Python. DataFrames werden meist durch das Einlesen von Dateien erstellt, können aber auch manuell erstellt werden.

### 2.4 Pandas-on-Spark-DataFrames

Spark bietet die Möglichkeit, DataFrames mit der bekannten Pandas API zu bearbeiten. Gerade für neue Benutzer, die sich Pandas DataFrames gewohnt sind, ist dies von erheblichem Nutzen.

Dabei können Spark DataFrames, Pandas-on-Spark-DataFrames, und Pandas DataFrames untereinander konvertiert werden.


## 3. Shakespeare erkunden

Als erstes Beispiel nehmen wir die gesammelten Werke von Shakespeare. Der gesammte Text ist verfügbar in der Datei `t8.shakespeare.txt` im `data`-Ordner auf dem SDS-Server.

Es ist möglich, die Datei in der Shell zu inspizieren. Sie enthält etwa 5.5 MB an Daten (ist also kein Fall für Big Data 😀). Die ersten paar Zeilen können einfach mit `head t8.shakespeare.txt` angezeigt werden.

In einem neuen Jupyter-Notebook erstellen wir erst die Spark-Session:

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder \
    .appName('Shakespeare') \
    .master('local[4]') \
    .getOrCreate()
```

Nun können wir die Text-Datei einfach einlesen:

```python
txt = spark.read.text('../data/t8.shakespeare.txt')
```

**Notiz:** Die verschiedenen von Spark unterstützten Datenquellen sind in der Dokumentation zu finden: [https://spark.apache.org/docs/latest/sql-data-sources.html](https://spark.apache.org/docs/latest/sql-data-sources.html)

Falls das DataFrame anschliessend für weitere Operationen verwendet wird, macht es unter Umständen Sinn, auf jedem Knoten eine Kopie zur Verfügung zu haben. Dies geschieht mit der Funktion `.cache()`, die beim Ladevorgang gleich angefügt werden kann:

```python
txt = spark.read.text('../data/t8.shakespeare.txt').cache()
```

`.cache` ist natürlich nur sinnvoll, wenn das DataFrame nicht allzu gross ist...

Wir können schauen was `txt` nun eigentlich ist. Im Notebook einfach eingeben:

```python
txt
```

Dabei ist `txt` ein `DataFrame[value: string]`. Das heisst, es handelt sich um ein Spark DataFrame, das genau eine Spalte mit Namen `value` hat. Diese Spalte hat den Datentyp `string`. Spark hat also jede Zeile der Textdatei in eine Zeile eines DataFrames umgewandelt. Wir können also einfach schauen wie viele Zeilen der liebe Shakespeare in seinem Leben geschrieben hat:

```python
txt.count()
```

Nun ja, nicht ganz Big Data.

Wir können auch Filter anwenden. Zum Beispiel können wir schauen, in wie vielen Zeilen das Wort *Hamlet* vorkommt. Dabei erstellt der Filter ein neues DataFrame, das wir sinngemäss `hamlet` nennen:

```python
hamlet = txt.filter(txt.value.contains('Hamlet'))
hamlet.count()
```

Wir können nun auch die Wörter zählen, die Shakepeare geschrieben hat. Dabei wird zuerst der Wert jeder Zeile in einen Array von Wörtern aufgeteilt (`split`), und anschliessend seine Grösse (`size`) abgefragt. Die Funktionen `split` und `size` müssen dafür erst importiert werden.

```python
from pyspark.sql.functions import size, split
wordCount = txt.select(
    size(split(txt.value, '\s+')).name('nwords')
)
```

Dieser Code ist auch deshalb interessant, weil er die Funktion `.select` benützt. Wir kennen ja das Schlüsselword `SELECT` im Wesentlichen vom SQL. Diese Funktion hat den gleichen Zweck, nämlich SQL-ähnliche Abfragen durchzuführen. Im obigen Fall wird dabei die Anzahl Wörter in einer Zeile abgefragt. Der Befehl `.name('nwords')` macht nichts anderes als der Spalte im zu erstellenden DataFrame einen Namen zu geben.

Die `split`-Funktion ist noch von einem gewissen Interesse, vor allem das zweite Argument `\s+`. Es handelt sich dabei um eine Regex-Ausdruck, der besagt, dass der Seperationsstring ein Leerzeichen (`\s`, also ein Leerschlag, Tab oder ähnlich) sein soll. `+` besagt einfach, dass ein oder mehrere Vorkommnisse des vorgehenden Buchstabens gleich behandelt werden sollen. Also ist `\s+` einfach der Ausdruck für *"ein oder mehrere Leerzeichen"*.

Wenn wir nun im Notebook den Wert von `wordCount` abfragen:

```python
wordCount
```

bekommen wir als Antwort einfach `DataFrame[nwords: int]`, und nicht etwa Anzahl Wörter pro Zeile. Dies kommt daher, dass die Anzahl Wörter zu diesem Zeitpunkt gar nicht berechnet wurden! In der Tat wird der Inhalt der DataFrames erst dann berechnet, wenn die Werte auch tatsächlich gebraucht werden. Man bezeichnet diese Eigenschaft als `lazily evaluated`, also ein faules DataFrame sozusagen. Diese Faulheit macht aber durchaus Sinn, da es beim verteilten Berechnen vergleichsweise hohe Latenzen gibt durch die Netzwerk-Operationen. Es lohnt sich also, mehrere Operationen in einer Berechnung zusammenzufassen. Die Berechnung kann durch die Funktion `collect()` in jedem Fall ausgelöst werden. Wir können aber auch nur den Inhalt des DataFrames explizit inspizieren, z.B. mit:

```python
wordCount.show()
```

was uns die ersten 20 Zeilen gibt. In einem Notebook sind diese Zeilen unter Umständen nicht sehr schön formatiert. Wir können jedoch einfach die ersten paar Zeilen manuell extrahieren und dann in eine Pandas-DataFrame umwandeln, wo die Tabelle besser lesbar ist:

```python
wordCount.limit(5).toPandas()
```

Bei nur einer Spalte ist der Unterschied natürlich nicht gross, aber bei Tabellen mit vielen Spalten ist eine lesbare Tabelle angenehmer.

Nun kennen wir aber immer noch nicht die Anzahl Wörter, die Shakespeare geschrieben hat. Dazu müssen wir ein Reduce-Schritt durchführen, der mittels der `agg()`-Funktion bewerkstelligt werden kann. Wir brauchen auch noch eine Funktion um die Summe zu berechnen, und wir brauchen einen Weg um die Spalte `nwords` zu bezeichnen:

```python
from pyspark.sql.functions import sum, col
n = wordCount.agg(
    sum(col('nwords'))
)
```

Wir können den Inhalt von `n` abrufen und erhalten `DataFrame[sum(nwords): bigint]`. `collect` kann uns da helfen:

```python
n.collect()
```

und wir stellen fest, dass Shakespeare Schreib-Millionär ist.

Natürlich können wir die ganze Operation in einem Befehl zusammenfassen:

```python
txt.select(
    size(split(txt.value, '\s+')).name('nwords')
).agg(
    sum(col('nwords'))
).collect()
```

Nebst dieser Funktions-basierten Art, Abfragen durchzuführen, stellt Spark auch SQL-Abfragen zur Verfügung. Dabei kann das DataFrame als temporäre Tabelle registriert werden:

```python
txt.createOrReplaceTempView('shakespeare')
```

und nun können SQL-Abfragen gemacht werden:

```python
extract = spark.sql("""
    SELECT value
    FROM shakespeare
    LIMIT 10
""")
```

Natürlich haben wir da wird ein faules DataFrame bekommen, das wir aber durch

```python
extract.show()
```

überprüfen können.

Somit können wir die Wort-Frequenzen relativ einfach berechnen:

```python
wordFreq = spark.sql("""
    SELECT word, COUNT(*) AS n
    FROM (
        SELECT EXPLODE(SPLIT(value, "\\\\s+")) AS word
        FROM shakespeare
    ) B
    GROUP BY word
    ORDER BY n DESC
""")
```


## 4. Die Bevölkerung der Schweiz erkunden

Nun sollten wir im Stande sein, die vom Bundesamt für Statistik publizierten Statistiken auf Hektarniveau zu erkunden. Die Daten werden im Detail auf folgender Seite beschrieben, und können auch heruntergeladen werden: [www.geostat.admin.ch](http://www.geostat.admin.ch).

Ein Geostat-Datensatz befindet sich auf dem SDS-Server im `data`-Ordner.

Berechnen Sie die Bevölkerung der Schweiz (Kolonne `B21BTOT`), sowie der Anteil der 0-20 jährigen Bevölkerung in Prozent.
