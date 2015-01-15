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

Geo-wynik [miasta.geojson](https://github.com/khinz/MongoDB/blob/master/miasta.geojson)

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

# Zadanie 2 #


Zastosowany plik:

* getglue_sample.json

wybrałem 500 000 pierwszych linii i zapisałem je do pliku... 

* getglue_sample\_cut.json

...a użyłem do tego polecania: 

    head -n 500000 getglue_sample.json > getglue_sample_cut.json

Import do bazy:
    
    time ./bin/mongoimport.exe -d nosql -c bigdata --file ../../getglue_sample_cut.json

A trwało to tak:

    real 3m12.214s
    user 0m0.015s
    sys  0m0.016s


### agregacje ###

Wyświetlenie nazw modeli wraz z ilością wystąpień:

	db.bigdata.aggregate([
		{ $group: { _id: "$modelName", ilosc: {$sum: 1}} },
		{ $sort: {ilosc: -1 } },
		{ $project: { modelName: "$_id", ilosc: "$ilosc", _id: 0} }
	])
	
Rezultaty, mamy, taty:

	{
		"result" : [
			{
					"ilosc" : 1090418,
					"modelName" : "movies"
			},
			{
					"ilosc" : 909572,
					"modelName" : "tv_shows"
			},
			{
					"ilosc" : 5,
					"modelName" : "recording_artists"
			},
			{
					"ilosc" : 5,
					"modelName" : "topics"
			}
		],
		"ok" : 1
	}

Zliczenie użytkowników, którzy mają ponad 7600 rekordów:
	
	db.bigdata.aggregate([
		{ $group: { _id: "$userId", ilosc: {$sum: 1}} },
		{ $sort: {ilosc: -1 } },
		{ $match: { ilosc: {$gte : 7600 } } },
		{ $project: { userId: "$_id", ilosc: "$ilosc", _id: 0 }}
	])
	
Mama, tata, rezultata:

	{
        "result" : [
                {
                        "ilosc" : 13585,
                        "userId" : "jesusvarelaacosta"
                },
                {
                        "ilosc" : 13141,
                        "userId" : "LilMissCakeCups"
                },
                {
                        "ilosc" : 12946,
                        "userId" : "johnnym2001"
                },
                {
                        "ilosc" : 11764,
                        "userId" : "erwin_ali_perdana"
                },
                {
                        "ilosc" : 11513,
                        "userId" : "endika"
                },
                {
                        "ilosc" : 11352,
                        "userId" : "cathy_blessing_hughes"
                },
                {
                        "ilosc" : 9737,
                        "userId" : "khairulazmas"
                },
                {
                        "ilosc" : 8778,
                        "userId" : "maria_santana1"
                },
                {
                        "ilosc" : 8683,
                        "userId" : "samnaeev"
                },
                {
                        "ilosc" : 7668,
                        "userId" : "wididip"
                }
        ],
        "ok" : 1
	}
