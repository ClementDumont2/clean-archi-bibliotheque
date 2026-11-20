# EXERCICE — Clean Architecture en .NET : « Bibliothèque de quartier »

Fil rouge de la session. Tu codes, je challenge. Je ne fournis **aucun code C#** : tu peux me demander des indices, pas des solutions.

## 0. Règles du jeu

- **.NET 8 minimum** (on utilisera `TimeProvider`, apparu en .NET 8). xUnit pour les tests.
- **Un jalon à la fois.** Tu me dis « Jalon N terminé » → j'audite → je valide ou je refuse. Pas de jalon N+1 avant validation.
- **Un commit (ou plus) par jalon**, et un tag `jalon-N` sur le commit soumis à l'audit.
- **Commentaires dans le code** : uniquement le *pourquoi* (invariant, contrainte, arbitrage), jamais le *quoi* que le code dit déjà.
- **Les réponses de `ARCHITECTURE.md` sont personnelles** : elles doivent citer *ton* code (noms de classes, de fichiers), pas de la théorie générique. Une réponse qui pourrait être copiée d'un blog est refusée.

---

## 1. Contexte métier

### L'API : gestion des prêts d'une bibliothèque de quartier

Une petite bibliothèque veut gérer ses prêts. Pas un CRUD : les règles dépendent de **l'état** (combien d'emprunts en cours, le livre est-il dehors) et du **temps** (retards), et elles touchent **plusieurs objets à la fois**. Elles doivent rester vraies quelle que soit l'interface (HTTP aujourd'hui, CLI demain) et la base de données.

### Langage commun

| Terme | Définition |
| --- | --- |
| **Membre** | Personne inscrite à la bibliothèque. |
| **Livre** | Un **exemplaire physique unique**, identifié par un ISBN. Pas de gestion de stock multi-exemplaires. |
| **Emprunt** | Prêt d'un livre à un membre, daté. Actif tant que le livre n'est pas rendu. |
| **Retard** | Un emprunt actif depuis **plus de 21 jours** (21 jours pile = pas encore en retard). |

### Invariants (règles métier strictes)

- **R1 — Quota** : un membre ne peut pas avoir plus de **3 emprunts actifs** simultanément.
- **R2 — Exemplaire unique** : un livre ne peut être emprunté que s'il est **disponible**. Il redevient disponible uniquement à son retour.
- **R3 — Retard bloquant** : un membre ayant **au moins un emprunt en retard** ne peut plus emprunter tant qu'il n'a pas rendu ce livre.

Cas limites, tranchés d'avance pour que les tests soient déterministes :

- Rendre un livre qui n'est pas emprunté (ou emprunté par un autre membre) est une erreur.
- Un ISBN invalide (ni 10 ni 13 chiffres, checksum ignorée) est refusé à la création.
- Rendre un livre en retard est toujours possible (et débloque le membre s'il n'a plus d'autre retard).

### Cas d'usage

1. Inscrire un membre
2. Ajouter un livre au catalogue
3. Emprunter un livre
4. Rendre un livre
5. Lister les emprunts en retard *(lecture seule)*

---

## 2. Architecture cible

### Les couches

| Couche | Rôle | Contient | N'a le droit de connaître que |
| --- | --- | --- | --- |
| **Domain** | Le métier pur : entités, value objects, invariants. | Entités, VO, exceptions/erreurs du domaine, éventuels services de domaine. | **Rien.** |
| **Application** | L'orchestration des cas d'usage. | Use cases, commandes/requêtes, DTO de sortie, **ports** (interfaces vers l'extérieur). | Domain |
| **Infrastructure** | Les détails techniques qui implémentent les ports. | EF Core, SQLite, repositories concrets, mapping. | Application (et donc Domain) |
| **WebApi** | Le point d'entrée HTTP **et** la racine de composition. | Endpoints, contrats HTTP, DI, gestion d'erreurs HTTP. | Application, Infrastructure (uniquement pour le câblage DI) |

### La Dependency Rule

> Les dépendances du **code source** pointent toujours **vers l'intérieur**. Le Domain ne dépend de rien ; rien d'interne ne connaît quoi que ce soit d'externe.

Le **flux d'exécution** (HTTP → use case → base) et le **sens des dépendances** de code ne sont pas le même sens : comprendre pourquoi est l'objet du fil rouge. Elle se vérifie dans les `.csproj`, pas dans un schéma.

### Structure de solution attendue

```
Bibliotheque.sln
src/
  Bibliotheque.Domain
  Bibliotheque.Application
  Bibliotheque.Infrastructure
  Bibliotheque.WebApi
tests/
  Bibliotheque.Domain.Tests
  Bibliotheque.Application.Tests
  Bibliotheque.Infrastructure.Tests
  Bibliotheque.WebApi.Tests
  Bibliotheque.Architecture.Tests      (jalon 5)
```

### Références autorisées (`ProjectReference`)

| Projet | Peut référencer |
| --- | --- |
| Domain | *aucun projet, aucun package* (hors BCL) |
| Application | Domain |
| Infrastructure | Application, Domain |
| WebApi | Application, Infrastructure |
| *.Tests | le projet testé ; **jamais** une couche plus externe |

Toute autre référence est un défaut d'architecture, pas un détail.

---

## 3. Jalons

### Jalon 1 — Solution & couche Domain

**Objectif** : un modèle métier qui protège lui-même ses invariants, sans framework.

**Rendu technique**

- `Bibliotheque.sln` + projets `Domain` et `Domain.Tests` (les autres projets peuvent attendre).
- Dans `Domain` : `Membre`, `Livre`, `Emprunt`, `Isbn` (value object), identifiants typés (`MembreId`, `LivreId`, ou équivalent justifié), et le mécanisme d'erreur métier de ton choix (exceptions dédiées **ou** type `Result`, à justifier).
- Tu décides quel objet porte quel invariant (R1/R2/R3) — c'est une question du jalon.
- **Contrainte** : le domaine ne lit jamais l'horloge. La date « maintenant » lui est fournie.
- **Contrainte** : état modifiable uniquement par des méthodes métier (pas de setters publics).
- Tests unitaires attendus (xUnit, sans bibliothèque de mock) :
  - emprunt nominal ; 3ᵉ emprunt accepté, 4ᵉ refusé (R1) ;
  - livre déjà emprunté refusé (R2) ; livre redevenu disponible après retour ;
  - retard à 21 j accepté, à 22 j bloquant (R3) ; déblocage après retour du livre en retard ;
  - retour d'un livre non emprunté / emprunté par un autre membre ;
  - `Isbn` invalide refusé.

**Définition de terminé**

- `Domain.csproj` : zéro `ProjectReference`, zéro `PackageReference`.
- Aucun `DateTime.Now` / `UtcNow` dans le Domain.
- `dotnet build` et `dotnet test` verts.

**Rendu documentaire** → section *Jalon 1* de `ARCHITECTURE.md` (6 questions).

---

### Jalon 2 — Cas d'usage & couche Application

**Objectif** : orchestrer le domaine, définir les ports, tester sans base de données.

**Rendu technique**

- Projets `Application` et `Application.Tests`.
- **Ports** : un repository pour les livres, un pour les membres, une abstraction de validation de transaction (unit of work). Aucun type EF ou HTTP dans leurs signatures.
- **Use cases** : un par cas d'usage de la section 1 (5 au total), chacun avec sa commande/requête et son résultat. Sans MediatR obligatoire : si tu l'utilises, justifie le coût.
- Le temps via `TimeProvider` injecté (pas d'interface maison).
- Les use cases ne renvoient **pas** d'entités du Domain : des DTO/read models.
- Une méthode d'extension d'enregistrement DI propre à la couche (`AddApplication`).
- Tests (repositories **faux écrits à la main** dans le projet de tests, `FakeTimeProvider` autorisé) :
  - emprunt nominal ; membre inconnu ; livre inconnu ;
  - règle métier violée → erreur propagée **et rien n'est persisté** ;
  - retour nominal ; requête des retards avec une horloge figée.

**Définition de terminé**

- `Application.csproj` référence **uniquement** Domain.
- `Application.Tests` ne référence ni Infrastructure ni WebApi.
- Aucune entité du Domain n'apparaît dans une signature publique de use case.

**Rendu documentaire** → section *Jalon 2* (5 questions).

---

### Jalon 3 — Persistance & couche Infrastructure

**Objectif** : brancher une vraie base sans polluer le centre.

**Rendu technique**

- Projets `Infrastructure` et `Infrastructure.Tests`.
- EF Core + SQLite : `DbContext`, **configuration Fluent** (`IEntityTypeConfiguration`) par entité, implémentations des repositories et du unit of work, méthode d'enregistrement DI (`AddInfrastructure`).
- Le Domain n'a **aucun** attribut de persistance. Si EF t'impose une concession (constructeur, champ privé), tu la traites côté mapping et tu la documentes.
- Tests d'intégration (SQLite en mémoire) :
  - aller-retour d'un `Membre` avec ses emprunts ;
  - rechargement d'un agrégat puis violation d'invariant toujours détectée (le mapping n'a pas contourné le domaine) ;
  - les repositories respectent le contrat attendu par les use cases (introuvable, ajout, mise à jour).

**Définition de terminé**

- `Infrastructure.csproj` référence Application (et Domain), pas WebApi.
- Aucun type EF dans une signature publique de `Application` ou `Domain`.
- Domain et Application **inchangés**, ou chaque modification justifiée par écrit.

**Rendu documentaire** → section *Jalon 3* (5 questions).

---

### Jalon 4 — Endpoints API & injection de dépendances

**Objectif** : exposer l'application en HTTP, câbler le tout au seul endroit autorisé.

**Rendu technique**

- Projets `WebApi` et `WebApi.Tests`.
- `Program.cs` = **racine de composition** : seul endroit où les types concrets d'Infrastructure sont connus.
- Endpoints (Minimal API ou controllers, ton choix justifié) pour les 5 cas d'usage.
- Contrats de requête/réponse HTTP **distincts** des DTO d'Application.
- Traduction centralisée des erreurs en `ProblemDetails` : inexistant → 404, règle métier violée → 409, entrée invalide → 400.
- Tests d'intégration (`WebApplicationFactory`, horloge figée injectée) : parcours nominal + un test par règle R1, R2, R3 avec le code HTTP attendu.

**Définition de terminé**

- Un endpoint contient quelques lignes : mapper → appeler le use case → mapper. Aucune règle métier.
- Aucune entité du Domain sérialisée en JSON.
- `dotnet run` démarre, les 5 routes répondent.

**Rendu documentaire** → section *Jalon 4* (5 questions).

---

### Jalon 5 — Garde-fous & preuve de réversibilité

**Objectif** : prouver que l'architecture tient, par des tests et par une évolution réelle.

**Rendu technique**

- `Bibliotheque.Architecture.Tests` : tests automatiques qui échouent si la Dependency Rule est violée (références d'assemblies, via réflexion ou NetArchTest ; minimum : Domain sans dépendance, Application sans EF/ASP.NET, Infrastructure ne référence pas WebApi).
- **Preuve de changement de base** : ajouter une persistance **en mémoire** (repositories + unit of work) sélectionnable par configuration, sans toucher une ligne de `Domain` ni `Application`. La preuve est `git diff --stat` entre `jalon-4` et `jalon-5` sur ces deux projets : vide.
- **Évolution métier R4** : *« un emprunt peut être prolongé une seule fois de 7 jours, et pas s'il est déjà en retard »*. Nouveau use case + endpoint + tests à chaque niveau. Tu listes, couche par couche, les fichiers touchés.

**Définition de terminé**

- Tests d'architecture verts **et** vérifiés rouges quand tu casses volontairement une référence (une capture ou un commit de preuve suffit).
- Toute la suite passe avec les deux persistances.

**Rendu documentaire** → section *Jalon 5* (5 questions) + bilan final.

---

## 4. Protocole d'audit (ce que je vérifie)

Quand tu annonces la fin d'un jalon, j'inspecte, dans cet ordre :

1. **Dépendances** : chaque `.csproj` contre la matrice de la section 2 (`ProjectReference` *et* `PackageReference`), plus les `using` suspects.
2. **Isolation du domaine** : pas de fuite d'infrastructure, pas d'état public modifiable, invariants non contournables.
3. **Build & tests** : `dotnet build`, `dotnet test`, pertinence des tests (pas seulement leur couleur).
4. **Cohérence jalon / code** : ce que dit `ARCHITECTURE.md` correspond à ce que fait le code.
5. **Réponses écrites** : exactes, précises, ancrées dans ton code, avec les conséquences expliquées (le *pourquoi*, pas la définition).

Verdict : ✅ **Validé** (tu passes au jalon suivant) · 🟡 **Validé avec réserves** (points à corriger, à reprendre au jalon suivant) · ❌ **Refusé** (liste des défauts bloquants, tu corriges et tu me re-soumets).
