# Subdivide testen

Auf dem SDS-Server befindet sich im `data/naturalearth`-Ordner befinden sich zwei SQL-Dumps, einer für die Länder der Erde, und der zweite für die wichtigsten Seen.

Wir wollen eine Abfrage machen um zu wissen, in welchem Land (oder in welchen Ländern) sich jeder See befindet.

In einem ersten Schritt sollen wir die SQL-Abfrage basierend auf den existierenden Daten erstellen.

Anschliessend sollen die Geometrien der Länder mit `ST_Subdivide` aufgeteilt werden, und die gleiche Abfrage nochmals durchgeführt werden. Dabei soll jeweils die Zeit gemessen werden, um festzustellen, wie gross der allfällige Zeitgewinn ist.
