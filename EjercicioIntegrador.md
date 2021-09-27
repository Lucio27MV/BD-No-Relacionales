# Tarea Ejercicio Integrador

## Luciano Montes de Oca Villa - 173670


4. Cómo podemos saber si los tuiteros hispanohablantes interactúan más en las noches?

Primero sacamos el total de tweets hispanohablantes que serán 2385 tweets:
```javascript
db.tweets.aggregate([
{$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
{$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
{$match:{"user.lang":'es'}},
{$group: {_id:"$fulllocale.languages", "conteo": {$count:{}}}}
])
```

Luego contamos cuántos tweets se hicieron en la noche (considerando un horario de las 19:00:00 a las 23:59:59) y nos dirá que se publicaron 1407 tweets hispanohablantes:
```javascript
db.tweets.aggregate([
  {$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
  {$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
  {$match:{"user.lang":'es',"created_at":/^[A-Z]+[a-z]{1,2}\s+[A-Z]+[a-z]{1,2}\s+[0-9]{1,2}\s+([1]+[9]|[2]+[0-3])+:+[0-5]+[0-9]+:+[0-5]+[0-9].........../}},
  {$group: {_id:"$fulllocale.languages", "conteo": {$count:{}}}}
])
```

Por último contamos el número de tweets en el transcurso del día (fuera del horario anterior, es decir, que no se hicieron en la noche), resultando 978 tweets hispanohablantes y de esta manera sabremos que los tuiteros hispanohablantes interactúan más en las noches:
```javascript
db.tweets.aggregate([
  {$lookup: {from:"primarydialects","localField":"user.lang","foreignField":"lang","as":"language"}},
  {$lookup: {from:"languagenames","localField":"language.locale","foreignField":"locale","as":"fulllocale"}},
  {$match:{"user.lang":'es',"created_at":/^[A-Z]+[a-z]{1,2}\s+[A-Z]+[a-z]{1,2}\s+[0-9]{1,2}\s+([0]+[1-9]|[1]+[0-8])+:+[0-5]+[0-9]+:+[0-5]+[0-9].........../}},
  {$group: {_id:"$fulllocale.languages", "conteo": {$count:{}}}}
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

Intervalo de 7:00:00pm a 6:59:59am:
```javascript
db.tweets.aggregate([
  {$lookup: {from:"countries","localField":"user.time_zone","foreignField":"time_zone","as":"countryy"}},
  {$match:{"created_at":/^[A-Z]+[a-z]{1,2}\s+[A-Z]+[a-z]{1,2}\s+[0-9]{1,2}\s+([1]+[9]|[2]+[0-3]|[0]+[0-6])+:+[0-5]+[0-9]+:+[0-5]+[0-9].........../}},
  {$group: {_id:"$countryy.country", "conteo": {$count:{}}}},
  {$sort: {"conteo":-1}}
])
```

Intervalo de 7:00:00am a 6:59:59pm:
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
