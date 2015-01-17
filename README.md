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

# Zadanie 3 #

# "Koń jaki jest, każdy widzi"


# Przygotować funkcje map i reduce, które: #


# wyszukają wszystkie anagramy w pliku word_list.txt #

"Anagram – nazwa wywodząca się od słów greckich: ana- (nad) oraz grámma (litera), oznacza wyraz, wyrażenie lub całe zdanie powstałe przez przestawienie liter bądź sylab innego wyrazu lub zdania, wykorzystujące wszystkie litery (głoski bądź sylaby) materiału wyjściowego"

Import pliku z nagłówkiem id i word, następnie map-reduce. 

```js
mongoimport -c words -d nosql--file K:\MongoDB\words.csv --type csv --headerline
```


Czas na mapreduce. Internety podsunęły mi poniższe rozwiązanie - js:
```
t = db.words;
db.letters.out.drop(); // drop collection if exists

//Map
m = function() {
var xx = this.word.split("").sort().toString(); //split word to single characters and sort them
emit(xx,this.word); //emit this array as key , and word as a value , so words which have anagrams will be emitted at least twice
};

//Reduce
r = function(key, value) {
return results.toString(); // return string (for some reason reduce method must return the same type as emited value)
};
var finalize = function(key, reducedValue) { // finalize for simpler recognition which word has anagram, we could also run map-reduce once more and extract only those which have
if(reducedValue.length > 6 )
return reducedValue; //every word has 6chars, so if our string contains more, that means whit have at least 2 anagrams
else
return reducedValue=null; //we set value filed to null to indicate that this words has no anagrams in database
//later db.letters.out.find({$type:2})
};

res = t.mapReduce( m, r, {out: 'letters.out',finalize: finalize});
//db.letters.find();
db.letters.out.find();
```

```js
{
    "result" : "letters.out",
    "timeMillis" : 396,
    "counts" : {
        "input" : 8199,
        "emit" : 8199,
        "reduce" : 914,
        "output" : 7011
    },
...

```

Liczba Anagramów: 914

A oto kilka z nich... 

```js
db['letters.out'].find({value:{$type:2}}).sort({id:-1})

```
```js


{
    "_id" : "a,a,b,l,s,t",
    "value" : "basalt,tablas"
}

{
    "_id" : "a,a,b,m,n,t",
    "value" : "bantam,batman"
}

{
    "_id" : "a,a,c,e,t,v",
    "value" : "caveat,vacate"
}

{
    "_id" : "a,a,c,i,m,n",
    "value" : "caiman,maniac"
}

{
    "_id" : "a,a,c,l,r,s",
    "value" : "rascal,scalar"
}


{
    "_id" : "a,b,i,n,r,s",
    "value" : "bairns,brains"
}

{
    "_id" : "a,b,i,n,r,y",
    "value" : "binary,brainy"
}

{
    "_id" : "a,b,m,r,s,u",
    "value" : "umbras,rumbas"
}


```

# wyszukają najczęściej występujące słowa z Wikipedia data PL aktualny plik z artykułami, ok. 1.3 GB #

* Pobrałem wiki dump Wikipedii

* Przekonwertowałem wiki dump do formatu .json pozostawiając pola:id, tytuł, url, tekst i przypsiy. 

Przykład otrzymanego jsona:
```
{
	"_id" : ObjectId("54afddb38009de6b59e6c876"),
	"url" : "http://en.wikipedia.org/wiki/Alergologia",
	"text" : "Alergologia.\nAlergologia - dziedzina medycyny zajmująca się rozpoznawaniem i leczeniem schorzeń alergicznych.",
	"id" : [
		4
	],
	"annotations" : [
		{
			"surface_form" : "medycyny",
			"uri" : "Medycyna",
			"offset" : 37
		},
		{
			"surface_form" : "leczeniem",
			"uri" : "Leczenie",
			"offset" : 77
		},
		{
			"surface_form" : "schorzeń alergicznych",
			"uri" : "Alergia",
			"offset" : 87
		}
	]
}
```
Ograniczyłem mój wiki dump  tylko do pierwszych 5 000 000 wierszy z wykorzystaniem polecenia:
```
head -n 5000000 wikidump.xml > obojetnie.xml
```
Okrojony wiki dump sparsowałem do formatu json poleceniem:
```
bzip2 -dc obojetnie.xml.tar.bz2 | python annotated_wikiextractor.py
```
Pozostało tylko:
- złączenie plików za pomocą polecenia:
```
cat $(find -name "*" | sort) > wiki.json

```
- zaimportowanie do mongo kolekcji:
```
mongoimport -d wiki -c wiki --file wiki.json
```

W mongo zdefiniowałem funkcje map i reduce:
```
var map = function() {  
    var tekst = this.text;
    if (tekst) { 
      
        tekst = tekst.toLowerCase().split(" "); 
        for (var i = tekst.length - 1; i >= 0; i--) {
           
            if (tekst[i])  {      
               emit(tekst[i], 1);
            }
        }
    }
};

var reduce = function( key, values ) {    
    var licz = 0;    
    values.forEach(function(v) {            
        licz +=v;    
    });
    return licz;
};

db.xml.mapReduce(map, reduce, {out: "zlicz"});
```
Mamy, taty, rezultaty...:
```
db.zlicz.find().sort({value:-1})
{ "_id" : "w", "value" : 129735 }
{ "_id" : "i", "value" : 79788 }
{ "_id" : "z", "value" : 55150 }
{ "_id" : "na", "value" : 53334 }
{ "_id" : "się", "value" : 47263 }
{ "_id" : "do", "value" : 40125 }
{ "_id" : "jest", "value" : 26801 }
{ "_id" : "–", "value" : 19938 }
{ "_id" : "przez", "value" : 17332 }
{ "_id" : "nie", "value" : 16535 }
{ "_id" : "a", "value" : 16352 }
{ "_id" : "od", "value" : 15168 }
{ "_id" : "oraz", "value" : 13862 }
{ "_id" : "o", "value" : 13183 }
{ "_id" : "to", "value" : 12712 }
{ "_id" : "że", "value" : 11653 }
{ "_id" : "po", "value" : 10809 }
{ "_id" : "roku", "value" : 10727 }
{ "_id" : "są", "value" : 9844 }
{ "_id" : "jako", "value" : 8769 }
```
