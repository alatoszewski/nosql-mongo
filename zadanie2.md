### Zadanie 2
W tym zadaniu wykorzystalem baze GetGlue i biblioteke PyMongo.

Aby zainicjalizowac baze wpisalem ponizsze linijki kodu:

```sh
from pymongo import MongoClient
db = MongoClient().getglue
```
###Zapytanie 1

Ponizsza agregacja zwraca 10 rezyserow ktorzy nakrecili najwiecej filmow:

#MongoDB
```sh
db.movies.aggregate(
    { $match: {"modelName": "movies" || "tv_shows"  } },
    { $group: {_id: {"dir": "$director", id: "$title"}, count: {$sum: 1}} },
    { $group: {_id: "$_id.dir" , count: {$sum: 1}} },
    { $sort: {count: -1} },
    { $limit : 10}
    );
```

#pymongo
```sh
db.movies.aggregate(
    { "$match": {"modelName": "movies" || "tv_shows"  } },
    { "$group": {"_id": {"dir": "$director", "id": "$title"}, "count": {"$sum": 1}} },
    { "$group": {"_id": "$_id.dir" , "count": {"$sum": 1}} },
    { "$sort": {"count": -1} },
    { "$limit" : 10}
    );
```

```sh
{ "_id" : "not available", "count" : 1474 }
{ "_id" : "various directors", "count" : 54 }
{ "_id" : "alfred hitchcock", "count" : 50 }
{ "_id" : "michael curtiz", "count" : 48 }
{ "_id" : "woody allen", "count" : 47 }
{ "_id" : "takashi miike", "count" : 43 }
{ "_id" : "jesus franco", "count" : 43 }
{ "_id" : "ingmar bergman", "count" : 42 }
{ "_id" : "john ford", "count" : 42 }
{ "_id" : "steven spielberg", "count" : 41 }
```

###Zapytanie 2

10 najpopularniejszych filmow i programow telewizyjnych.
#MongoDB
```sh
db.movies.aggregate(
    { $match: {"modelName": "movies" || "tv_shows"  } },
    { $group: {_id: "$title", count: {$sum: 1}} },
    { $sort: {count: -1} },
    { $limit : 10}
    );
  ```  
#pymongo
```sh
db.movies.aggregate(
    { "$match": {"modelName": "movies" || "tv_shows"  } },
    { "$group": {"_id": "$title", "count": {"$sum": 1}} },
    { "$sort": {"count": -1} },
    { "$limit" : 10}
    );
```

```sh
{ "_id" : "The Twilight Saga: Breaking Dawn Part 1", "count" : 87521 }
{ "_id" : "The Hunger Games", "count" : 79340 }
{ "_id" : "Marvel's The Avengers", "count" : 64356 }
{ "_id" : "Harry Potter and the Deathly Hallows: Part II", "count" : 33680 }
{ "_id" : "The Muppets", "count" : 29002 }
{ "_id" : "Captain America: The First Avenger", "count" : 28406 }
{ "_id" : "Avatar", "count" : 23238 }
{ "_id" : "Thor", "count" : 23207 }
{ "_id" : "The Hangover", "count" : 22709 }
{ "_id" : "Titanic", "count" : 20791 }
```


###Zapytanie 3
10 najbardziej aktywnych użytkowników
#MongoDB
```sh
db.movies.aggregate({$group:{_id: "$userId", count:{$sum: 1}}},{$sort:{count: -1}},{$limit: 10});
```
#pymongo
```sh

db.movies.aggregate({"$group":{"_id": "$userId", "count":{"$sum": 1}}},
{"$sort":{"count": -1}},{"$limit": 10});
```

```sh
{ "_id" : "LukeWilliamss", "count" : 696782 }
{ "_id" : "demi_konti", "count" : 68137 }
{ "_id" : "bangwid", "count" : 59261 }
{ "_id" : "zenofmac", "count" : 56233 }
{ "_id" : "agentdunham", "count" : 55740 }
{ "_id" : "cillax", "count" : 43161 }
{ "_id" : "tamtomo", "count" : 42378 }
{ "_id" : "hblackwood", "count" : 32832 }
{ "_id" : "ellen_turner", "count" : 32239 }
{ "_id" : "husainholic", "count" : 32135 }
```


###Zapytanie 4
Szukamy wpisów które zostały skomentowane oraz mają status Disliked.

#MongoDB
```sh
db.movies.aggregate(
  {$match: 
    {"action": "Disliked" }
  },
  {$match:
    {"comment": {$ne: ""}} 
  }, 
  {$group: 
    {_id: "$title", count: {$sum: 1}
  }
  }, 
  {$sort: 
    {count: -1 }
  }, 
  {$limit: 10}
);
```
#pymongo
```sh
db.movies.aggregate(
  {"$match": 
    {"action": "Disliked" }
  },
  {"$match": 
    { "comment": {"$ne": ""}
  }
  },
  {"$group": 
  {"_id": "$title", 
    "count": 
      {"$sum": 1}
  } 
  }, 
  {"$sort": 
    {"count": -1 }
  }, 
  {"$limit": 10} 
);
```	

```sh
{ "_id" : "Avatar", "count" : 24 }
{ "_id" : "Jersey Shore", "count" : 24 }
{ "_id" : "Glee", "count" : 21 }
{ "_id" : "Skyline", "count" : 17 }
{ "_id" : "Sucker Punch", "count" : 14 }
{ "_id" : "The Last Exorcism", "count" : 11 }
{ "_id" : "2012", "count" : 11 }
{ "_id" : "Twilight", "count" : 11 }
{ "_id" : "Clash of the Titans", "count" : 10 }
{ "_id" : "Spider-Man 3", "count" : 10 }
```