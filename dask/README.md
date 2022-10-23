# Dask

Dask ist eine Python-Bibliothek für die paralelle und verteilte Verarbeitung von Daten. Dabei baut Dask auf den bekannten Python-Bibliotheken wie Numpy or Pandas auf, und ist deshalb relative einfach zu lernen.

Dabei besteht Dask aus den folgenden High-Level-Bestandteilen:

- Dask **DataFrames**: DataFrames wie man sie aus Pandas kennt, unterstützt jedoch das Verteilung der Berechnungen auf einem Cluster.

- Dask **Arrays**: Arrays wie man sie aus Numpy kennt. Es ist jedoch möglich, verteilte Berechnungen durchzuführen, und auch Berechnungen, bei den der Array eigentlich zu gross ist um im Speicher gehalten werden zu können.

Daneben gibt es Bestandteile die fundamentalere Operationen erlauben und müssen nicht unbedingt direkt verwendet werden:

- **dask.delayed** erlaubt das Paralellisieren von irgendwelchem Python-Code.

- **Distributed** verteilt die Daten und Berechnungen auf einem Cluster.

- **Futures** erlauben die verteilten Berechnungen.


## Dask-GeoPandas

Dask-GeoPandas ist eine weitere Python-Bibliothek. Sie vereint, wie der Name es schon verrät, Dask mit GeoPandas.
