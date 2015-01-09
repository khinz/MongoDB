##MongoDB
=======
Pod maską:

Procesor: Intel(R) Core(TM) i7-3612QM CPU @ 2.10GHz
RAM: 12 GB DDR3

---

##Zadanie 1a

Zaimportowanie i uruchomienie danych z pliku Train.csv
---

Użyłem komendy do oczyszczenia bazy ze zbędnych pustych ciągów i znaków nowych linni:
```bash
	cat Train.csv | tr "\n" " " | tr "\r" "\n" | head -n 6034196 > train2.csv
```
następnie importujemy plik do bazy mongodb
```
>time mongoimport --collection train --type csv --file Train1.csv --headerline
```

czas:
```
real	7m43.229s
user	1m7.610s
sys	0m6.167s
```

![](https://github.com/khinz/MongoDB/blob/master/TrainLoad.png)

##Zadanie 1b 
Zliczyć liczbę zaimportowanych rekordów:
```
>db.Train.count()​;
>6034195
```

##Zadanie 1c

Zadanie 1c polega na zamianie formatu danych.
Należy zamienić string zawierający tagi na tablicę napisów z tagami, następnie zliczyć wszystkie tagi i wszystkie różne tagi.
Użyłem tutaj prostego skryptu dla mongo (znaleziony w internetach) :

```
db.train.find( { "tags" : { $type : 2 } } ).snapshot().forEach(
 function (x) {
  if (!Array.isArray(x.tags)){
    x.tags = x.tags.split(' ');
    db.train.save(x);
}});
```

## Zadanie 1d
Geograficzny Json :)

##I
4 miasta, które są ulokowane w pobliżu Gdańska
```
> db.Miasta.ensureIndex({"loc" : "2dsphere"})
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}
> db.Miasta.find({ loc: {$near: {$geometry: gdansk.loc}} }).skip(1).limit(4)
```
![](https://github.com/khinz/MongoDB/blob/master/PointNear.png)


##II
zliczanie miast w promieniu 100km od Gdańska - definicja dla Gdańska i polecenie

```
> var gdansk = db.Miasta.findOne({"miasto": "Gdańsk"})
> db.Miasta.find( { "loc" :{ $near : { $geometry : gdansk.loc , $minDistance : 50000, $maxDistance: 100000 } } }).skip(1).count()
```
Wynik: 11

![](https://github.com/khinz/MongoDB/blob/master/count.png)

##III
zliczanie miast w obszarze poligonu

Najpierw definiujemy wybrane miasta, a potem...
...tworzymy poligon:
```
var polygon = {type: "Polygon", coordinates: [[gdansk.loc.coordinates , olsztyn.loc.coordinates , warszawa.loc.coordinates, lublin.loc.coordinates, opole.loc.coordinates, poznan.loc.coordinates, gdansk.loc.coordinates]]}

```

Zapytanie:
```
> db.Miasta.find({"loc" : {"$geoWithin" : {"$geometry" : polygon}}}).count()

```
Wynik: 87

![](https://github.com/khinz/MongoDB/blob/master/poligon.png)
