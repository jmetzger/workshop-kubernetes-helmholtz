# Eigenes Helmchart verwenden 

  * helmchart liegt im git

## Schritt 1: Chart erstellen 

```
cd
mkdir helm-chart-test/
cd helm-chart-test
helm create final-chart
```

## Schritt 2: Repo einrichten 

  * Achtung: Repo muss leer sein
  * URL rauskopieren, z.B. https://gitlab.com/training-tn<nr>/final-chart-<dein-kuerzel>.git


## Schritt 3: lokal git initialisieren und pushen

```
# in git einchecken
git init
git remote add origin https://gitlab.com/training-tn<nr>/final-chart-<dein-kuerzel>.git
git add -A
git commit -am "Chart hochschicken"
git push
```
