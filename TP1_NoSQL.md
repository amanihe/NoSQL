
# TP – Bases de Données NoSQL  
## Redis • MongoDB • JSON

---

# 1. Introduction au NoSQL

## 1.1. Qu’est-ce qu’une base de données NoSQL ?
Les bases de données **NoSQL** sont des systèmes qui **ne reposent pas sur le modèle relationnel** traditionnel (pas de tables, lignes, colonnes, ni schéma strict).  
Elles sont conçues pour :
- gérer de grandes quantités de données,s
- accepter des données **non structurées ou semi-structurées**,
- être **hautement scalables** (scalabilité horizontale),
- offrir une **latence très faible**.

---

## 1.2. Les 4 grandes familles de NoSQL

| Famille | Description | Exemples |
|--------|-------------|----------|
| **Clé-valeur** | données sous forme `clé : valeur` ; très rapide | Redis, DynamoDB |
| **Document** | stockage de documents JSON/BSON | MongoDB, CouchDB |
| **Colonnes** | stockage par colonnes, efficace pour l’analyse | Cassandra, HBase |
| **Graphes** | gestion de relations complexes | Neo4j, ArangoDB |

---

## 1.3. Objectif du TP  
Ce TP permet de :
- découvrir NoSQL via **Redis** (clé-valeur) et **MongoDB** (document),
- manipuler les commandes de base,
- comprendre les modèles de données,
- travailler avec **JSON**.

---

# 2. Redis – Base de Données Clé-Valeur

## 2.1. Présentation de Redis
Redis est une base **clé-valeur en mémoire** :
- extrêmement rapide,
- utilisée comme **cache**, **gestion de sessions**, **file d’attente**, etc.
- supporte des structures avancées (listes, sets, hashes, sorted sets),
- propose un système Pub/Sub pour les messages en temps réel.

---

## 2.2. Installation (Docker)
```bash
docker run -d --name redis -p 6379:6379 redis
```
Connexion à Redis :
```bash
docker exec -it redis redis-cli
```

---

## 2.3. Commandes de base

###  Clés simples
Les clés simples permettent de stocker une valeur associée à un nom (clé). Elles sont la base de Redis.
```bash
SET demo "Bonjour"
GET demo
DEL demo
EXPIRE demo 60
TTL demo
```

###  Incrémentation (compteur)
Redis permet d’incrémenter ou décrémenter une clé numérique, utile pour des compteurs (visites, likes…).
```bash
SET 1mars 0
INCR 1mars
DECR 1mars
```

---

## 2.4. Listes
Les listes sont des séquences ordonnées d’éléments.
```bash
RPUSH mesCours "BDA"
RPUSH mesCours "Services Web"
LRANGE mesCours 0 -1
LPOP mesCours
RPOP mesCours
```
==> Les listes **acceptent les doublons**.

---

## 2.5. Ensembles (Sets)
Les sets sont des collections non ordonnées.
```bash
SADD utilisateurs "Samir"
SADD utilisateurs "Amani"
SMEMBERS utilisateurs
SREM utilisateurs "Amani"
```
==> Les sets **ne contiennent que des valeurs uniques**.

Union :
```bash
SUNION utilisateurs autresUtilisateurs
```

---

## 2.6. Ensembles ordonnés (Sorted Sets)
Les sorted sets associent un score numérique à chaque élément, idéalement utilisés pour les **classements / scores**.
```bash
ZADD score4 19 "Augustin"
ZADD score4 12 "Samir"
ZRANGE score4 0 -1
ZREVRANGE score4 0 -1
ZRANK score4 "Samir"
```

---

## 2.7. Hashes
Les hashes permettent de stocker des objets sous forme de paires champ → valeur.
```bash
HSET user:1 username "Amani"
HSET user:1 age 25
HGETALL user:1
HINCRBY user:1 age 1
```

---

## 2.8. Pub/Sub (temps réel)
Pub/Sub permet d’envoyer des messages en temps réel entre clients via des canaux.

Terminal 1 :
```bash
SUBSCRIBE mescours user:1
```
Terminal 2 :
```bash
PUBLISH mescours "Un nouveau cours sur MongoDB"
PUBLISH user:1 "Bonjour user 1!"
```

---

## 2.9. Bases disponibles
Redis propose **16 bases (0 à 15)** :

```bash
SELECT 1
KEYS *
```

---

# 3. MongoDB – Base de Données Orientée Documents

## 3.1. Présentation
MongoDB stocke des données sous forme de **documents JSON/BSON**, ce qui permet :
- flexibilité totale,
- pas de schéma fixe,
- insertion rapide,
- requêtes puissantes.

---
# Prise en main de MongoDB 


## 1. Importation de la base *sample_mflix*

### Téléchargeement de l'archive

``` bash
curl https://atlas-education.s3.amazonaws.com/sampledata.archive -o sampledata.archive
```

### On copier le fichier dans le container

``` bash
docker cp sampledata.archive mongodb:/sampledata.archive
```

### Importion des données

``` bash
docker exec -it mongodb mongorestore --archive=sampledata.archive
```

### Vérification de la base

``` bash
docker exec -it mongodb mongo
```

Dans mango:

``` javascript
show dbs
use sample_mflix
show collections
```

# 2. Requêtes du TP

## Partie 1 -- Filtrer et projeter les données

### 1. Films sortis depuis 2015

``` javascript
db.movies.find({ year: { $gte: 2015 } }).limit(5)
```

### 2. Films de genre "Comedy"

``` javascript
db.movies.find({ genres: "Comedy" })
```

### 3. Films entre 2000 et 2005

``` javascript
db.movies.find({ year: { $gte: 2000, $lte: 2005 } }, { title: 1, year: 1 }).pretty()
```

### 4. Drama ET Romance

``` javascript
db.movies.find({ genres: { $all: ["Drama", "Romance"] } }, { title: 1, genres: 1 })
```

### 5. Films sans champ rated

``` javascript
db.movies.find({ rated: { $exists: false } }, { title: 1 })
```

## Partie 2 -- Agrégations

### 6. Nombre de films par année

``` javascript
db.movies.aggregate([
  { $group: { _id: "$year", total: { $sum: 1 } } },
  { $sort: { _id: 1 } }
])
```

### 7. Moyenne IMDb par genre

``` javascript
db.movies.aggregate([
  { $unwind: "$genres" },
  { $group: { _id: "$genres", moyenne: { $avg: "$imdb.rating" } } },
  { $sort: { moyenne: -1 } }
])
```

### 8. Nombre de films par pays

``` javascript
db.movies.aggregate([
  { $unwind: "$countries" },
  { $group: { _id: "$countries", total: { $sum: 1 } } },
  { $sort: { total: -1 } }
])
```

### 9. Top 5 réalisateurs

``` javascript
db.movies.aggregate([
  { $unwind: "$directors" },
  { $group: { _id: "$directors", total: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 5 }
])
```

### 10. Films triés par note IMDb

``` javascript
db.movies.aggregate([
  { $sort: { "imdb.rating": -1 } },
  { $project: { title: 1, "imdb.rating": 1 } }
])
```

## Partie 3 -- Mises à jour

### 11. Ajouter un champ

``` javascript
db.movies.updateOne({ title: "Jaws" }, { $set: { etat: "culte" } })
```

### 12. Incrémenter les votes IMDb

``` javascript
db.movies.updateOne({ title: "Inception" }, { $inc: { "imdb.votes": 100 } })
```

### 13. Supprimer le champ poster

``` javascript
db.movies.updateMany({}, { $unset: { poster: "" } })
```

### 14. Modifier le réalisateur de Titanic

``` javascript
db.movies.updateOne(
  { title: "Titanic" },
  { $set: { directors: ["James Cameron"] } }
)
```

## Partie 4 -- Requêtes complexes

### 15. Films les mieux notés par décennie

``` javascript
db.movies.aggregate([
  { $match: { "imdb.rating": { $exists: true } } },
  {
    $project: {
      title: 1,
      decade: { $subtract: ["$year", { $mod: ["$year", 10] }] },
      "imdb.rating": 1
    }
  },
  { $group: { _id: "$decade", maxRating: { $max: "$imdb.rating" } } },
  { $sort: { _id: 1 } }
])
```

### 16. Films commençant par "Star"

``` javascript
db.movies.find({ title: /^Star/ }, { title: 1 })
```

### 17. Films avec plus de 2 genres

``` javascript
db.movies.find({ $where: "this.genres.length > 2" }, { title: 1, genres: 1 })
```

### 18. Films de Christopher Nolan

``` javascript
db.movies.find(
  { directors: "Christopher Nolan" },
  { title: 1, year: 1, "imdb.rating": 1 }
)
```

## Partie 5 -- Indexation

### 19. Créer un index sur year

``` javascript
db.movies.createIndex({ year: 1 })
```

### 20. Vérifier les index

``` javascript
db.movies.getIndexes()
```

### 21. Comparer une requête avec explain()

``` javascript
db.movies.find({ year: 1995 }).explain("executionStats")
```

### 22. Supprimer l'index

``` javascript
db.movies.dropIndex({ year: 1 })
```

### 23. Créer un index composé

``` javascript
db.movies.createIndex({ year: 1, "imdb.rating": -1 })
```

# . Partie JSON

## 4.1. Qu’est-ce que JSON ?
JSON signifie **JavaScript Object Notation**.  
C’est un format :
- lisible par l’humain,
- léger,
- utilisé pour échanger des données (API),
- compatible avec MongoDB.

---

## 4.2. Exemple JSON
```json
{
  "nom": "Amani",
  "age": 25,
  "cours": ["BDA", "Redis", "MongoDB"],
  "actif": true
}
```

---

## 4.3. Points importants
- les clés sont entre guillemets `" "`,
- types disponibles : string, number, boolean, object, array,
- pas de commentaires autorisés,
- utilisé partout dans les bases NoSQL.

---

# 5. Conclusion

Ce TP m’a permis de :
- comprendre les **4 familles NoSQL**,
- manipuler Redis et ses structures (listes, sets, hashes, Pub/Sub),
- utiliser MongoDB Atlas, insérer et interroger des documents,
- maîtriser la structure du format **JSON**.

Les NoSQL offrent **rapidité, flexibilité et scalabilité**, ce qui les rend essentiels dans les applications modernes.
