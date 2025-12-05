=====================================
0. Introduction
=====================================


Ce document présente la mise en place et l’analyse d’un Replica Set MongoDB.  
Il couvre la réplication, les mécanismes d’élection, la tolérance aux pannes et les commandes essentielles pour administrer un cluster MongoDB.

---

=====================================
2. Compréhension de base
=====================================

-------------------------------------
2.1 Qu’est-ce qu’un Replica Set ?
-------------------------------------
Un Replica Set est un groupe de serveurs MongoDB maintenant automatiquement des copies identiques des données afin d’assurer la haute disponibilité.

-------------------------------------
2.2 Rôle du Primary
-------------------------------------
Le Primary reçoit toutes les écritures et garantit une vue cohérente des données.

-------------------------------------
2.3 Rôle des Secondaries
-------------------------------------
Les Secondaries répliquent les données du Primary et peuvent gérer certaines lectures selon la configuration.

-------------------------------------
2.4 Pourquoi pas d’écriture sur un Secondary ?
-------------------------------------
Pour éviter les conflits et maintenir un flux de réplication cohérent.

-------------------------------------
2.5 Cohérence forte
-------------------------------------
Lecture toujours effectuée sur le Primary → données à jour.

-------------------------------------
2.6 Différence entre readPreference primary et secondary
-------------------------------------
- **primary** : cohérence maximale  
- **secondary** : charge réduite mais données potentiellement obsolètes

-------------------------------------
2.7 Pourquoi lire sur un Secondary ?
-------------------------------------
Pour délester le Primary lors de lectures lourdes.

---

=====================================
3. Commandes et configuration
=====================================

-------------------------------------
3.1 Initialiser un Replica Set
-------------------------------------
```js
rs.initiate()
```

-------------------------------------
3.2 Ajouter un nœud
-------------------------------------
```js
rs.add("host:port")
```

-------------------------------------
3.3 Afficher l’état du Replica Set
-------------------------------------
```js
rs.status()
```

-------------------------------------
3.4 Vérifier le rôle d’un nœud
-------------------------------------
```js
rs.isMaster()
```

-------------------------------------
3.5 Forcer la bascule du Primary
-------------------------------------
```js
rs.stepDown()
```

---

=====================================
4. Résilience et tolérance aux pannes
=====================================

-------------------------------------
4.1 Ajouter un arbitre
-------------------------------------
```js
rs.addArb("host:port")
```

-------------------------------------
4.2 Configurer un délai de réplication
-------------------------------------
```js
cfg = rs.conf()
cfg.members[i].slaveDelay = 120
rs.reconfig(cfg)
```

-------------------------------------
4.3 Absence de majorité
-------------------------------------
Aucun Primary ne peut être élu → cluster en lecture seule.

-------------------------------------
4.4 Critères d’élection
-------------------------------------
- Majorité disponible  
- Priorité  
- Fraîcheur des données  

-------------------------------------
4.5 Auto-dégradation
-------------------------------------
Un nœud perdant la majorité se rétrograde en Secondary.

-------------------------------------
4.6 Nombre impair de nœuds
-------------------------------------
Facilite l’obtention d’une majorité.

-------------------------------------
4.7 Partition réseau
-------------------------------------
Peut empêcher l’élection d’un Primary ou provoquer une dégradation du cluster.

-------------------------------------
4.8 Exemple 27017 / 27018 / 27019
-------------------------------------
Le Secondary devient Primary grâce à la majorité (Secondary + Arbitre).

-------------------------------------
4.9 Utilité d’un slaveDelay
-------------------------------------
Permet de revenir en arrière après une erreur humaine.

-------------------------------------
4.10 Lecture toujours à jour
-------------------------------------
```js
readConcern: "linearizable"
writeConcern: { w: "majority" }
```

-------------------------------------
4.11 Écriture confirmée par deux nœuds
-------------------------------------
```js
writeConcern: { w: 2 }
```

-------------------------------------
4.12 Lecture obsolète depuis un Secondary
-------------------------------------
Solution :
- readPreference "primary"  
- readConcern "majority"

-------------------------------------
4.13 Vérifier le Primary
-------------------------------------
```js
rs.status()
```

-------------------------------------
4.14 Forcer la bascule manuelle
-------------------------------------
```js
rs.stepDown()
```

-------------------------------------
4.15 Ajouter un Secondary
-------------------------------------
```js
rs.add("host:port")
```

-------------------------------------
4.16 Retirer un nœud
-------------------------------------
```js
rs.remove("host:port")
```

-------------------------------------
4.17 Secondary caché
-------------------------------------
```js
cfg.members[i].hidden = true
rs.reconfig(cfg)
```

-------------------------------------
4.18 Modifier la priorité d’un nœud
-------------------------------------
```js
cfg.members[i].priority = 2
rs.reconfig(cfg)
```

-------------------------------------
4.19 Vérifier le délai de réplication
-------------------------------------
```js
rs.printSecondaryReplicationInfo()
```

---

=====================================
5. Questions complémentaires
=====================================

-------------------------------------
5.1 rs.freeze()
-------------------------------------
Empêche un nœud de devenir Primary pendant un temps donné.

-------------------------------------
5.2 Redémarrage sans perte de configuration
-------------------------------------
La configuration est stockée dans la base locale.

-------------------------------------
5.3 Surveiller la réplication
-------------------------------------
```js
rs.printReplicationInfo()
rs.printSlaveReplicationInfo()
```

-------------------------------------
5.4 Arbitre
-------------------------------------
Participe aux élections, ne stocke pas de données.

-------------------------------------
5.5 Vérifier la latence
-------------------------------------
```js
rs.printSecondaryReplicationInfo()
```

-------------------------------------
5.6 Retard des Secondaries
-------------------------------------
```js
rs.printSlaveReplicationInfo()
```

-------------------------------------
5.7 Réplication asynchrone vs synchrone
-------------------------------------
MongoDB utilise une réplication asynchrone.

-------------------------------------
5.8 Modifier la configuration sans redémarrer
-------------------------------------
```js
rs.reconfig()
```

-------------------------------------
5.9 Secondary très en retard
-------------------------------------
Resynchronisation totale possible.

-------------------------------------
5.10 Conflits de données
-------------------------------------
Règle du dernier write appliquée.

-------------------------------------
5.11 Plusieurs Primary ?
-------------------------------------
Impossible grâce au vote majoritaire.

-------------------------------------
5.12 Écriture sur un Secondary ?
-------------------------------------
Déconseillée : les données seraient écrasées.

-------------------------------------
5.13 Réseau instable
-------------------------------------
Peut provoquer des basculements et pertes de majorité.

---

=====================================
6. Conclusion
=====================================

Ce travail permet de comprendre les mécanismes essentiels de réplication, d’élection et de tolérance aux pannes dans MongoDB.  
Ces principes sont indispensables pour concevoir des architectures distribuées fiables.

