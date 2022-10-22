# Jupyter Notebook für Apache Spark konfigurieren

Dieser Schritt ist optional. Einzig wenn mit dem mitgelieferten `pyspark`-Programm gearbeitet werden soll, kann es nützlich sein, es mit den Jupyter Notebooks zu integrieren. Der Punkt 2 unter erläutert wie dies geschieht.

Im Punkt 1 wird nochmals kurz erklärt wie die Jupyter Notebooks an sich installiert werden können.


## 1. Jupyter Notebook installieren

Der wohl einfachste Weg um Jupyter Notebook zu installieren ist mit Anaconda ([https://www.anaconda.com](https://www.anaconda.com/)). Jupyter Notebook ist Teil des Anaconda-Pakets.

Jupyter Notebook kann auch mit `pip` installiert werden:

```bash
pip install notebook
```

Anschliessend kann das Notebook ganz normal gestartet werden mit

```bash
jupyter notebook
```

Falls nun auch PySpark installiert ist, kann ganz normal PySpark-Code von den Notebooks aus ausgeführt werden. In diesem Fall muss die Spark-Session allerding manuell erstellt werden.


## 2. PySpark für Jupyter aktivieren

Das mit der Spark-Installierte mitgelieferte `pyspark`-Programm startet eine Python-Konsole wo die Spark-Session schon erstellt ist. Das manuelle Erstellen der Session entfällt in diesem Fall.

Falls wir nun diese automatische PySpark-Session auch mit Jupyter Notebooks verwenden wollen, müssen wir die folgenden Umgebungsvariablen setzen:

```bash
export PYSPARK_DRIVER_PYTHON="jupyter"
export PYSPARK_DRIVER_PYTHON_OPTS="notebook"
```

Dabei muss auch die Umgebungsvariable `SPARK_HOME` korrekt gesetzt sein.

Dies kann unter Umständen auch direkt in einem Environment-File geschehen, also z.B. im `.envrc` im Falle von Dotenv.

Anschliessend kann Jupyter Notebook enifach von der Shell gestartet werden mit

```bash
pyspark
```

Der Python-Kernel ist automatisch eine Pyspark-Umgebung mit lokalem Cluster. Die Spark-Session steht dabei in der `spark`-Variable zur Verfügung.
