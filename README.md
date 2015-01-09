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
