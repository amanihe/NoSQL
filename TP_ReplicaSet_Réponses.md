# TP MongoDB Replica Set  

---

#  Partie 1 — Compréhension de base   (detaillée plus dans "[ l'introduction ](Replication_MongoDB.md)" )
## 1. Qu’est-ce qu’un Replica Set dans MongoDB ?
   Un Replica Set est un groupe de serveurs MongoDB contenant les mêmes données afin d’assurer la haute disponibilité et la tolérance aux pannes.

## 2. Quel est le rôle du Primary dans un Replica Set ?  
   Le Primary reçoit toutes les écritures et synchronise les données vers les Secondaries.

## 3. Quel est le rôle essentiel des Secondaries ? 
   Ils répliquent les données du Primary et peuvent répondre aux lectures si configurés.

## 4. Pourquoi MongoDB n’autorise-t-il pas les écritures sur un Secondary ? 
   Pour éviter les conflits et garantir la cohérence des données.

## 5. Qu’est-ce que la cohérence forte dans le contexte MongoDB ? 
   Lire sur le Primary garantit toujours la donnée la plus récente.

## 6. Différence entre readPreference:"primary" et "secondary" ?  
   - primary → lecture toujours à jour  
   - secondary → lecture plus rapide mais parfois en retard  

## 7. Dans quel cas pourrait-on souhaiter lire sur un Secondary malgré les risques ?
   Pour soulager le Primary ou améliorer les performances.

---

#  Partie 2 — Commandes & configuration  

## 8. Commande pour initialiser un Replica Set :
   ```
   rs.initiate()
   ```

## 9. Ajouter un nœud  à un Replica Set après son initialisation : 
   ```
   rs.add("host:port")
   ```

## 10. Afficher l’état actuel du Replica Set : 
   ```
   rs.status()
   ```

## 11. Identifier le rôle d’un nœud :
   ```
   rs.isMaster()
   ```

## 12. Forcer le basculement du Primary : 
   ```
   rs.stepDown()
   ```
## 13. Désigner un nœud comme Arbitre :  
   ```
   rs.addArb("host:port")
   ```  

 On le fait pour obtenir une majorité sans ajouter un serveur complet.

14. **Configurer un Secondary avec un slaveDelay :**  
   ```
  cfg = rs.conf()
  cfg.members[1].slaveDelay = 80      
  rs.reconfig(cfg)  
   ```

---

#  Partie 3 — Résilience et tolérance aux pannes  

## 15. Si le Primary tombe et pas de majorité ?  
   Aucun nouveau Primary n’est élu → cluster en lecture seule.

## 16. Critères de choix du nouveau Primary :
   - priorité  
   - fraîcheur des données  
   - disponibilité  

## 17. Qu’est-ce qu’une élection ?
   Processus automatique choisissant le nouveau Primary après une panne.

## 18. Auto-dégradation :
   Le cluster passe en mode dégradé (lecture seule) sans Primary dès qu’il ne trouve plus la majorité des membres.

## 19. Pourquoi un nombre impair de nœuds ?
   Pour simplifier l’obtention d’une majorité.

## 20. Effet d’une partition réseau : 
   Une partie perd la majorité → y'aura plus de Primary → lecture seule.

## 21. Cluster : 27017 (Primary), 27018 (Secondary), 27019 (Arbitre). Si 27017 tombe ? 
   Si le Primary (27017) devient injoignable :
   - Le Secondary (27018) et l'Arbitre (27019) forment une majorité.
   - Une élection a lieu.
   - Le Secondary (27018) devient le nouveau Primary.
   - L’Arbitre ne peut pas devenir Primary.
   - L’ancien Primary, lorsqu'il revient, redevient Secondary.


## 22. Secondary avec slaveDelay de 120s : utilité ? 
   Un Secondary avec `slaveDelay: 120` réplique les données avec 2 min de retard.
   Utile pour :
   - récupérer des données après une erreur humaine,
   - éviter la propagation immédiate d’une corruption,
   - générer des rapports sur des données stables,
   - disposer d’un filet de sécurité avant un déploiement risqué.

## 23. Lecture toujours à jour, même après bascule :
   - readConcern: `"majority"`  
   - writeConcern: `{ w: "majority" }`

## 24. Écriture confirmée par au moins 2 nœuds : 
   ```
   { writeConcern: { w: 2 } }
   ```

## 25. Pourquoi donnée obsolète  lorsqu'un étudiant a lu depuis un Secondary ?  
   Les Secondaries répliquent avec un retard → données potentiellement obsolètes.

   **Comment éviter cela :**  
   - Lire sur le Primary (`readPreference: "primary"`).  
   - Ou "primaryPreferred".  
   - Réduire le replication lag.  
   - Ne pas lire sur un Secondary retardé (slaveDelay).


## 26. Vérifier quel nœud est Primary : 
   ```
   rs.isMaster()
   ```

## 27. Forcer une bascule manuelle :
   ```
   rs.stepDown()
   ```

## 28. Ajouter un Secondary à chaud : 
   ```
   rs.add("host:port")
   ```

## 29. Retirer un nœud défectueux :
   ```
   rs.remove("host:port")
   ```

## 30. **Nœud Secondary caché :
   ```
   cfg = rs.conf()
   cfg.members[1].hidden = true
   rs.reconfig(cfg)

   ```  
 → Un nœud caché ne reçoit pas de requêtes de lecture doc, c'est utile pour backups et analyses et reporting sans impacter les performances du cluster.

## 31. Modifier la priorité :
   ```
   cfg = rs.conf()
   cfg.members[1].priority = 2
   rs.reconfig(cfg)
   ```

## 32. Vérifier le délai de réplication :  
   ```
   rs.printSlaveReplicationInfo()
   ```  
→ Affiche le retard en secondes pour chaque Secondary.

## 33. Que fait la commande rs.freeze() : 
   Empêche un nœud de devenir Primary pendant un délai.
   
   Utile pour :
   - empêcher un nœud temporairement instable d'être élu,
   - forcer l’élection d’un autre nœud.
## 34. Redémarrer un Replica Set sans perdre la config : 
   La config est stockée dans `local.system.replset` → elle survit aux 
redémarrages.
  
Procédure :
   - Arrêter chaque mongod proprement.
   - Relancer avec les mêmes options --replSet.

## 35. Surveiller la réplication en temps réel : 
   ### Via commandes shell :
   ```
   rs.printSlaveReplicationInfo()
   rs.status()
   ```
   ### Via logs MongoDB
Chercher les messages contenant :
   - SYNC
   - rsSync
   - initial sync
   - oplog
 
Exemple: 
 ```
grep rsSync mongod.log
 ```
---

#  Questions complémentaires  
## 37. Qu’est-ce qu’un Arbitre (Arbiter) et pourquoi ne stocke-t-il pas de données ?
Un Arbiter est un membre d'un Replica Set qui ne contient pas de données.
Il sert uniquement à voter lors des élections pour atteindre la majorité.
Il ne stocke pas de données pour être léger, rapide et réduire l'utilisation des ressources.

## 38. Comment vérifier la latence de réplication entre le Primary et les Secondaries ?
Utiliser :
```
rs.printSlaveReplicationInfo()
```

## 39. Afficher le retard de réplication des membres secondaires :  
   ```
   rs.printSlaveReplicationInfo()
   rs.status()
   ```

## 40. Réplication asynchrone vs synchrone — quel type pour MongoDB ? 
   - synchrone : chaque écriture doit être confirmée par tous les nœuds → très cohérente mais lente. 
   - asynchrone : les Secondaries répliquent après coup → rapide mais pas toujours à jour. 
   → MongoDB utilise **asynchrone**

## 41. Peut-on modifier la configuration d’un Replica Set sans redémarrer les serveurs ? 
   Oui on modifier puis on applique:  `rs.reconfig()` → Aucun redémarrage nécessaire

## 42. Que se passe-t-il si un nœud Secondary est en retard de plusieurs minutes ?**  
- Il fournit des données obsolètes.
- Il peut perdre l’éligibilité au rôle de Primary.
- La réplication peut nécessiter une resynchronisation (initial sync) si l’oplog est trop ancien.

## 43. Comment MongoDB gère-t-il les conflits de données lors de la réplication ?
   Le Primary impose l’ordre → aucun conflit possible.
   c'est-à-dire:
   - Le Primary est la source de vérité.
   - Les Secondaries appliquent son oplog.
   -  S’il y a conflit → la version du Primary l’emporte (Primary Wins).

44. **Peut-on avoir plusieurs Primarys simultanément ? Pourquoi ?**  
   Impossible → règle de majorité.
   MongoDB impose un seul Primary à la fois pour assurer :

   - une cohérence d’écriture,

   - éviter les conflits,

   - garantir un ordre unique des opérations.

## 45. Pourquoi est-il déconseillé d’utiliser un Secondary pour des écritures ? 
Parce qu’un Secondary :
- ne supporte pas les écritures (sauf mode maintenance interne),
- réplique uniquement depuis le Primary,
- ne garantit pas la cohérence forte 
→ risque de perte ou divergence de données.

## 46. Conséquences d’un réseau instable sur un Replica Set 
- Perte temporaire de Primary → élections fréquentes.
- Risque de split-brain (si mauvaise config).
- Secondaries en retard → replication lag.

- Risque de réassignation fréquente des rôles.

- Dégradations de performance globales.

---