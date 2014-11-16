### Zadanie 1a

#####Przygotowanie pliku
Plik nalezy przeformatowac, tj. zmienic nazwe kolumny z "Id" na "_id" i zamienia znaki nowej lini. Ponizej jest skrypt ktorego uzylem. Skypt skopiowalem z repo prowadzacego.
```sh
#! /bin/bash

if [ -z "$1" ] ; then
echo "First replaces all the \\n with spaces then replaces all the \\r with \\n"
echo "Replace header line with: _id,title,body,tags (for mongoDB)"
echo "usage: $0 input.csv output.csv"
exit 1;
fi

tr '\n' ' ' < "$1" | tr '\r' '\n' > "$2"
sed -i '$ d' "$2"

sed -i '1 c "_id","title","body","tags"' "$2" 
```

#####Import pliku do bazy MongoDB
Aby zaimportowac plik csv do mongo wpisałem ponizszą komendę do konsoli:
```sh
time mongoimport -c Topics --type csv --file Train.csv --headerline
```
![mongo.png](https://raw.githubusercontent.com/alatoszewski/nosql-mongo/master/mongo.png)

#####Import pliku do bazy PSQL
Niestety PostresSQL nie jest tak "inteligentny" jak Mongo i nie jest wstanie odczytac struktury tabeli z pliku csv. Dlatego najpierw musialem stworzyc odpowiednia tabele.
```sh
CREATE TABLE Topics(
ID INT PRIMARY KEY NOT NULL,
TITLE CHAR(128) NOT NULL,
BODY CHAR(128) NOT NULL,
TAGS CHAR(128) NOT NULL,
);
```
Nastepnie mozna wypełnić tabele danymi z pliku:
```sh
COPY Topics FROM '/home/Adam/Desktop/NoSQL/Train.csv' DELIMITER ',' CSV;
```


![psql.png](https://raw.githubusercontent.com/alatoszewski/nosql-mongo/master/psql.png)

##Zestawiennie czasów

| Baza Danych |       Import I       |
|-------------|:--------------------:|
|   MongoDB   | 14m51s               |
| PostgreSQL  | 18m36s               |



### Zadanie 1b
Aby policzyc rozmiar kolekcji, w konsoli mongo wpisalem:
```sh
db.Topics.count()
```
Wynik:
```sh
6.034.195
```

### Zadanie 1c
```sh
var database = db.Train.find();

var tags = {};
var records = [];

var tagsCounter = 0;
var uniqueTagsCounter = 0;
var docsCounter = 0;

database.forEach(function(document) {
    tagsCounter++;
    var arrayOfTags = [];
    if(document.tags.constructor === String) {
        arrayOfTags = document.tags.split(" ");
    } 
    else { 
        arrayOfTags.push(document.tags);
    }
document.tags = arrayOfTags;
tagsCounter = arrayOfTags.length + tagsCounter;

for(var i=0; i<arrayOfTags.length; i++) {
    if(tags[arrayOfTags[i]] === undefined) {
        uniqueTagsCounter++;  
        tags[arrayOfTags[i]] = 1; 
    }
}

records.push(document);
if (docsCounter % 10000 === 0 || docsCounter === 6034195) {
    db.Train2.insert(records);
    records = [];
    }
});

print("Number of tags:" + tagsCounter);
print("Number of unique tags:" + uniqueTagsCounter);
```

### Zadanie 1d
Importuje plik json do bazy z wiekszymi Polskimi miastami: 
```sh
mongoimport -c places < polskiemiasta.json
```
Dodaje geoindex do kolekcji places:
```sh
db.places.ensureIndex({loc : "2dsphere"})
```

Zapytanie 1: Miasta oddalone od Warszawy o maksymalnie o 200 km:
```sh
db.polskiemiasta.find({loc: {$near: {$geometry: {type: "Point", coordinates: [21.000366210937496, 52.231163984032676]}, $maxDistance: 200000}}}).skip(1)
```

[Geojson1](https://github.com/alatoszewski/nosql-mongo/blob/master/Zapytanie1.geojson)

Zapytanie 2: Poniższe zapytanie prezentuje mniejwięcej kształt województwa zachodnioporskiego:
```sh
db.polskiemiasta.find({loc: {$geoWithin: {$geometry: {type: "Polygon", coordinates: [[[16.6717529296875,54.559322587438636],[16.858520507812496,54.23955053156179],[16.737670898437496,54.210648685175904],[16.8475341796875,53.97224342934289],[16.885986328125,53.6544055985474],[16.4410400390625,53.44226352500859],[16.710205078125,53.27178347923819],[16.06201171875,53.01147838269373],[15.875244140625,53.100620879214304],[15.3314208984375,52.948637884883205],[14.6337890625,52.626394589731326],[14.1558837890625,52.8757609818473],[14.403076171875,53.28163740806336],[14.2822265625,53.73896488496292],[14.4635009765625,53.9560855309879],[16.6717529296875,54.559322587438636]]]}}}})

[Geojson2](https://github.com/alatoszewski/nosql-mongo/blob/master/Zapytanie2.geojson)

Zapytanie 3. Szukamy miast będących na południku 15.733795166015623:
```sh
db.polskiemiasta.find({loc: {$geoIntersects: {$geometry: {type: "LineString", coordinates: [[15.733795166015623, -90],[15.733795166015623, 90]]}}}})
     
```

[Geojson3](https://github.com/alatoszewski/nosql-mongo/blob/master/Zapytanie3.geojson)

Zapytanie 4. Szuakmy trzech miast najbliższych Słupsku:
```sh
db.polskiemiasta.find({loc: {$near: {$geometry: {type: "Point", coordinates: [17.019195556640625,54.46804241740563]}}}}).limit(3)
```

[Geojson4](https://github.com/alatoszewski/nosql-mongo/blob/master/Zapytanie4.geojson)

Zapytanie 5. Reuzltat taki sam jak w zapytaniu 2. Użycie komendy $geoIntersect (co zachodzi w interakcję z polygonem)
```sh
db.polskiemiasta.find({loc: {$geoIntersects: {$geometry: {type: "Polygon", coordinates: [[[19.259033203125, 52.3923633970718], [18.1768798828125, 51.17589926990911], [19.7259521484375, 50.86144411058924], [20.5059814453125, 51.50532341149335], [20.23681640625, 52.1166256737882], [19.259033203125, 52.3923633970718]]]}}}})
```
[Geojson5](https://github.com/alatoszewski/nosql-mongo/blob/master/Zapytanie5.geojson)

Zapytanie 6. Miasta na drodze pomiędzy Gdańskiem a Zakopanem:
```sh
db.polskiemiasta.find({loc: {$geoIntersects: {$geometry: {type: "LineString", coordinates: [ [[18.655128479003906,54.34815256064472]], [19.948768615722656,49.29803885147804]]}}}})
```

[Geojson6](https://github.com/alatoszewski/nosql-mongo/blob/master/Zapytanie6.geojson)