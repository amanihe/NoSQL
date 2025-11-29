# TP – Prise en main de MongoDB (Requêtes mongo)

## 0. Importation des données

```
mongoimport --db lesfilms --collection films --file films.json --jsonArray
```

## 1. Vérification de structure

```
db.films.count()
db.films.findOne()
```

---

## 2. Afficher la liste des films d’action

```
db.films.find({ genre: "Action" })
```

---

## 3. Nombre de films d’action

```
db.films.count({ genre: "Action" })
```

---

## 4. Films d’action produits en France

```
db.films.find({ genre: "Action", country: "FR" })
```

---

## 5. Films d’action produits en France en 1963

```
db.films.find({ genre: "Action", country: "FR", year: 1963 })
```

---

## 6. Projection de champs

```
db.films.find({ genre: "Action", country: "FR" }, { title: 1, country: 1 })
```

---

## 7. Masquer _id

```
db.films.find({ genre: "Action", country: "FR" }, { _id: 0, title: 1, country: 1 })
```

---

## 8. Titres + grades des films d’action en France

```
db.films.find({ genre: "Action", country: "FR" }, { _id: 0, title: 1, grades: 1 })
```

---

## 9. Films d’action en France avec note > 10

```
db.films.find({ genre: "Action", country: "FR", "grades.note": { $gt: 10 } }, { _id: 0, title: 1, grades: 1 })
```

---

## 10. Toutes les notes > 10

```
db.films.find({ genre: "Action", country: "FR", grades: { $not: { $elemMatch: { note: { $lte: 10 } } } } }, { _id: 0, title: 1, grades: 1 })
```

---

## 11. Genres distincts

```
db.films.distinct("genre")
```

---

## 12. Grades distincts

```
db.films.distinct("grades.grade")
```

---

## 13. Films avec acteurs spécifiques (à adapter)

```
db.films.find({ actors: { $elemMatch: { last_name: { $in: ["Nom1", "Nom2", "Nom3"] } } } })
```

---

## 14. Films sans résumé

```
db.films.find({ summary: "" })
```

---

## 15. Films avec Leonardo DiCaprio en 1997

```
db.films.find({ year: 1997, actors: { $elemMatch: { first_name: "Leonardo", last_name: "DiCaprio" } } })
```

---

## 16. Films avec DiCaprio OU en 1997

```
db.films.find({ $or: [ { actors: { $elemMatch: { first_name: "Leonardo", last_name: "DiCaprio" } } }, { year: 1997 } ] })
```

---

