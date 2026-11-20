# ARCHITECTURE — Journal de compréhension

À compléter jalon après jalon. Les règles sont dans [EXERCICE.md](EXERCICE.md).

**Critères d'une bonne réponse** : ancrée dans *ton* code (cite des classes/fichiers), explique le *pourquoi* et les *conséquences*, tient en quelques paragraphes. Pas de définition de cours recopiée.

| Jalon | Statut |
| --- | --- |
| 1 — Domain | ⬜ À faire |
| 2 — Application | ⬜ À faire |
| 3 — Infrastructure | ⬜ À faire |
| 4 — WebApi | ⬜ À faire |
| 5 — Garde-fous & évolution | ⬜ À faire |

---

## Jalon 1 — Domain

**J1-Q1.** Pourquoi le projet `Domain` n'a-t-il aucune dépendance (ni projet, ni package) ? Que perds-tu concrètement si tu y ajoutes EF Core « juste pour les attributs » ?

> Réponse :

**J1-Q2.** Classe chacun de tes types en *entité*, *value object* ou *agrégat* et justifie. Pourquoi `Isbn` n'est-il pas un simple `string` ?

> Réponse :

**J1-Q3.** Où as-tu placé chacun des invariants R1, R2, R3, et pourquoi à cet endroit ? Que se passerait-il si R1 vivait dans un contrôleur ?

> Réponse :

**J1-Q4.** Pourquoi le domaine reçoit-il la date courante en paramètre au lieu de la lire lui-même ?

> Réponse :

**J1-Q5.** Montre, avec une de tes classes, la différence entre un modèle riche et un modèle anémique. Qu'est-ce qui empêche quelqu'un de créer un emprunt invalide dans ton code ?

> Réponse :

**J1-Q6.** Exceptions ou `Result` pour signaler une règle violée : ton choix, ses avantages et son coût pour les couches supérieures.

> Réponse :

---

## Jalon 2 — Application

**J2-Q1.** Pourquoi l'interface de tes repositories est-elle dans `Application` (ou `Domain`) et pas dans `Infrastructure` ? Dessine le sens de la flèche de dépendance et nomme le principe en jeu. Si tu l'as mise dans `Domain`, justifie contre l'alternative.

> Réponse :

**J2-Q2.** Quelle différence entre une règle métier (Domain) et une règle applicative (Application) ? Donne un exemple de chaque tiré de ton projet. Où vit « membre introuvable » et pourquoi ?

> Réponse :

**J2-Q3.** Pourquoi tes use cases ne retournent-ils pas les entités du Domain ? Que casserait-on le jour où un client dépend de leur forme ?

> Réponse :

**J2-Q4.** À quoi sert ton unit of work ? Où commence et où finit la transaction, et qui en est responsable ?

> Réponse :

**J2-Q5.** Tu testes un use case sans base de données. Qu'est-ce que cela prouve sur ton architecture ? Quelle est la limite de ces tests (ce qu'ils ne garantissent pas) ?

> Réponse :

---

## Jalon 3 — Infrastructure

**J3-Q1.** Si on remplace SQLite par PostgreSQL, puis par une base documentaire, quels projets et quels fichiers changent ? Lesquels ne bougent pas, et pourquoi ?

> Réponse :

**J3-Q2.** Pourquoi la configuration EF en Fluent API dans `Infrastructure` plutôt que des attributs sur les entités du Domain ?

> Réponse :

**J3-Q3.** Pourquoi `Infrastructure` référence-t-il `Application` et non l'inverse, alors que c'est `Application` qui « utilise » la base ? Explique l'inversion de dépendance avec tes propres types.

> Réponse :

**J3-Q4.** EF t'a-t-il forcé à une concession sur le Domain (constructeur, champ privé, collection) ? Comment l'as-tu contenue côté mapping, et quel compromis reste ?

> Réponse :

**J3-Q5.** Deux membres empruntent le même livre exactement au même moment. Ton domaine respecte R2, pourtant la règle peut être violée. Où et comment, et quelle parade côté Infrastructure ?

> Réponse :

---

## Jalon 4 — WebApi

**J4-Q1.** `WebApi` référence `Infrastructure`, alors que la Dependency Rule interdit à l'intérieur de connaître l'extérieur. Pourquoi est-ce légitime ici ? Qu'est-ce qu'une racine de composition ?

> Réponse :

**J4-Q2.** Pourquoi des contrats HTTP distincts des DTO d'Application et surtout des entités du Domain ?

> Réponse :

**J4-Q3.** Comment une violation de R1 devient-elle un `409` ? Qui fait la traduction, et pourquoi ni le Domain ni les use cases ne connaissent HTTP ?

> Réponse :

**J4-Q4.** Demain on ajoute une CLI (ou gRPC). Qu'est-ce qui est réutilisé tel quel, qu'est-ce qui est réécrit ?

> Réponse :

**J4-Q5.** Trois niveaux de validation existent : forme de l'entrée, validité de la demande, invariants métier. Où vit chacun dans ton code, et pourquoi ne pas tout mettre au même endroit ?

> Réponse :

---

## Jalon 5 — Garde-fous & évolution

**J5-Q1.** Comment garantis-tu que la Dependency Rule ne sera pas violée dans six mois par quelqu'un qui n'a pas lu ce document ? Que ne détectent pas tes tests d'architecture ?

> Réponse :

**J5-Q2.** Pour R4 (prolongation), liste les fichiers modifiés ou créés couche par couche. Que dit cette répartition de la cohésion de ton architecture ?

> Réponse :

**J5-Q3.** Dessine (ASCII ou Mermaid) le parcours complet d'un `POST` d'emprunt, de la requête HTTP au SQL et retour. Distingue clairement le sens du **flux d'exécution** et le sens des **dépendances de code**.

> Réponse :

**J5-Q4.** Avec ta persistance en mémoire, qu'as-tu dû écrire, qu'as-tu dû modifier ? Que prouve le `git diff --stat` vide sur `Domain` et `Application` ?

> Réponse :

**J5-Q5.** Critique honnête : sur ce projet précis, où la Clean Architecture est-elle sur-dimensionnée ? Dans quel contexte ne l'utiliserais-tu pas ?

> Réponse :

---

## Bilan final

**Schéma des dépendances (à jour)** :

> 

**Trois erreurs que j'ai faites en cours de route, et ce que j'en retiens** :

> 
