# Dask-Installation

Idealerweise sollte die Dask-Installation in einem Virtualenv geschehen. Sie unten für die Instruktionen.

Falls Anaconda installiert ist, ist Dask wahrscheinlich auch schon installiert. Ansonsten kann einfach mit Conda oder Pip installiert werden:

```bash
conda install dask
```

oder

```bash
python -m pip install "dask[complete]"
```

Dazu ist es noch nützlich die Bibliothek `graphviz` zu installieren, damit wir die Ausführungsgraphen anschauen können:

```bash
pip install graphviz
```


## Virtualenv erstellen

Idealerweise geschieht dies natürlich in einem Virtualenv oder in einem Conda environment.

Im Falle eines Virtualenv geht das einfach so:

```bash
python3 -m venv venv
```

was das Virtualenv in den Ordner `venv` installiert.

Danach muss das Virtualenv mit `source venv/bin/activate` aktiviert werden.

Falls wir ein Programm wie Dotenv in der Shell installiert haben, können wir uns die manuelle Aktivierung sparen, indem wir folgende Instruktionen in das `.envrc`-File schreiben:

```bash
#!/usr/bin/env bash
export VIRTUAL_ENV="/<path>/<to>/<project>/venv"
PATH="$VIRTUAL_ENV/bin:$PATH"
```
