# Tarea Ejercicio Integrador

## Luciano Montes de Oca Villa - 173670


4. Cómo podemos saber si los tuiteros hispanohablantes interactúan más en las noches?
```javascript
db.tweets.aggregate([         
    {$project: {"user.followers_count":1, "user.lang":1}}, 
    {$sort: {"user.followers_count":-1}},   
    {$limit: 10},
    {$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
    {$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
    {$project: {_id:1,"user.followers_count":1, "fulllocale.languages":1}}
])
```


5. Cómo podemos saber de dónde son los tuiteros que más tiempo tienen en la plataforma?
```javascript
db.tweets.aggregate([
    {$addFields: { "user.created_at": { "$toDate": "$user.created_at" }}},
    {$lookup: {from:"countries","localField":"user.time_zone","foreignField":"time_zone","as":"countryy"}},
    {$group: {_id:{"user_created_at":"$user.created_at","location":"$user.location","country":"$countryy.country"}}},
    {$sort: {"_id.user_created_at":1}}
])
```


6. En intervalos de 7:00:00pm a 6:59:59am y de 7:00:00am a 6:59:59pm, de qué paises la mayoría de los tuits?

Intervalo de 7:00:00pm a 6:59:59am
```javascript
db.tweets.aggregate([
  {$lookup: {from:"countries","localField":"user.time_zone","foreignField":"time_zone","as":"countryy"}},
  {$match:{"created_at":/^[A-Z]+[a-z]{1,2}\s+[A-Z]+[a-z]{1,2}\s+[0-9]{1,2}\s+([1]+[9]|[2]+[0-3]|[0]+[0-6])+:+[0-5]+[0-9]+:+[0-5]+[0-9].........../}},
  {$group: {_id:"$countryy.country", "conteo": {$count:{}}}},
  {$sort: {"conteo":-1}}
])
```

Intervalo de 7:00:00am a 6:59:59pm
```javascript
db.tweets.aggregate([
  {$lookup: {from:"countries","localField":"user.time_zone","foreignField":"time_zone","as":"countryy"}},
  {$match:{"created_at":/^[A-Z]+[a-z]{1,2}\s+[A-Z]+[a-z]{1,2}\s+[0-9]{1,2}\s+([0]+[7-9]|[1]+[0-8])+:+[0-5]+[0-9]+:+[0-5]+[0-9].........../}},
  {$group: {_id:"$countryy.country", "conteo": {$count:{}}}},
  {$sort: {"conteo":-1}}
])
```


7. De qué país son los tuiteros más famosos de nuestra colección?
```javascript
db.tweets.aggregate([
  {$lookup: {from:"countries","localField":"user.time_zone","foreignField":"time_zone","as":"countryy"}},
  {$group: {_id:{"país":"$countryy.country","seguidores":"$user.friends_count"}}},
  {$sort: {"_id.seguidores":-1}},
  {$limit : 15}
])
```
