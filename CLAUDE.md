# Clean Code — Règles applicables à tous les projets

> Ces règles s'appliquent par défaut. Un `CLAUDE.md` projet peut **assouplir** un point précis avec une justification explicite, mais jamais en silence. En cas de doute, **clarté > concision > cleverness**.

## 1. Nommage (intent-revealing names)

- **Le nom doit révéler l'intention** : pourquoi il existe, ce qu'il fait, comment il est utilisé. Si un nom a besoin d'un commentaire pour être compris, le nom est mauvais.
- **Pas d'encodage de type dans le nom** : pas de hongroise (`strName`, `iCount`), pas de préfixe `I` sur les interfaces sauf convention de l'écosystème.
- **Distinctions significatives** : `getActiveUsers()` vs `getUsers()` est utile ; `getUserData()` vs `getUserInfo()` est du bruit.
- **Prononçables et recherchables** : `creationTimestamp` plutôt que `crtTs`. Évite les abréviations exotiques.
- **Une notion = un mot** : ne mélange pas `fetch`, `get`, `retrieve`, `load` pour la même action dans le même domaine.
- **Conventions par catégorie** :
  - Classes/types : noms (Customer, OrderRepository).
  - Fonctions/méthodes : verbes (`createOrder`, `isEligible`, `hasPermission`).
  - Booléens : préfixe `is/has/can/should`.
  - Constantes : `SCREAMING_SNAKE_CASE` ou convention du langage.
- **Pas de magic numbers / magic strings** : remplace par une constante nommée. `MAX_RETRIES = 3` est plus parlant que `3`.

## 2. Fonctions

- **Petites** : ≤ 20 lignes en règle, ≤ 10 lignes en idéal. Si une fonction ne tient pas dans un écran sans scroll, elle est trop grosse.
- **Une seule responsabilité (Do One Thing)** : si tu décris la fonction et que tu utilises "et", découpe.
- **Un seul niveau d'abstraction** : ne mélange pas dans la même fonction du métier haut niveau et de la manipulation de bytes/SQL/HTTP brute.
- **Arguments minimaux** : 0 idéal, 1 acceptable, 2 ok, 3 à justifier, 4+ presque jamais. Au-delà, **encapsule** dans un objet/struct paramètre.
- **Pas de flag boolean en argument** : un boolean signale qu'on fait deux choses → deux fonctions.
- **Pas d'effet de bord caché** : une fonction `validatePassword(user, password)` ne doit pas créer de session en douce. Le nom doit refléter **tous** les effets.
- **Command Query Separation** : une fonction qui modifie un état ne retourne rien d'utile (sauf erreur), une fonction qui retourne une valeur ne mute rien.
- **Sortir tôt (early return)** plutôt que pyramide de `if` imbriqués.

## 3. Commentaires

- **Règle par défaut : pas de commentaire**. Le code propre se commente lui-même via des noms et un découpage clairs.
- **Quand commenter (les exceptions légitimes)** :
  - **Le "pourquoi" non évident** : contrainte cachée, invariant subtil, contournement d'un bug spécifique, choix surprenant.
  - **API publique** : docstring/JSDoc/Javadoc pour les contrats exposés à l'extérieur.
  - **TODO/FIXME** brefs et actionnables, idéalement avec un ticket.
- **Quand ne PAS commenter** :
  - **Pas de paraphrase du code** (`// incrémente i` au-dessus de `i++`).
  - **Pas de référence au contexte de la tâche** (`// ajouté pour le ticket JIRA-123`, `// utilisé par le flux X`) — ça rote dans le temps. Ces infos vont dans la PR/commit.
  - **Pas de commentaires "morts"** (`// ancien code laissé au cas où`). Si c'est mort, supprime — git garde l'historique.
  - **Pas de commentaire pour rattraper un mauvais nom** : renomme la variable.
- **Commentaire faux > pas de commentaire** : un commentaire désynchronisé du code induit en erreur. Si tu touches le code, vérifie les commentaires alentour.

## 4. Structure & séparation des responsabilités

- **Séparation des concerns (SoC)** au niveau module : I/O réseau, persistance, logique métier, présentation ne doivent pas vivre dans le même fichier/classe.
- **Layered / Hexagonal / Clean Architecture** comme cadre par défaut pour les apps non triviales :
  - **Domaine** au centre, sans dépendance externe (pas de framework, pas d'ORM, pas de HTTP).
  - **Application** (use cases) orchestre le domaine et expose les cas d'usage.
  - **Adapters / Infrastructure** branchent le monde extérieur (DB, HTTP, queues, UI) via des **ports** (interfaces) définis par le domaine.
  - **Règle de dépendance** : les couches externes dépendent des couches internes, jamais l'inverse.
- **Domain-Driven Design — usage modéré et explicite** :
  - **Bounded contexts** : si "User" ne veut pas dire la même chose dans Billing et Shipping, sépare les modèles.
  - **Ubiquitous language** : le code utilise les termes métier réels, pas du jargon dev.
  - Ne pas sortir l'arsenal DDD complet (Aggregates, Value Objects, Domain Events, etc.) pour un CRUD : **DDD se mérite**.
- **Organisation par feature plutôt que par couche technique** quand c'est possible (`/order/...` plutôt que `/controllers /services /repositories` éclatés).

## 5. SOLID — utiliser comme grille de lecture, pas comme dogme

- **S — Single Responsibility** : une classe / un module change pour une seule raison. Si deux acteurs métier différents poussent à modifier la même classe, scinde.
- **O — Open/Closed** : ouvert à l'extension, fermé à la modification. Important quand un point de variation est **avéré**, pas anticipé.
- **L — Liskov Substitution** : un sous-type doit pouvoir remplacer son parent sans casser les invariants. Si tu surcharges pour neutraliser un comportement, ta hiérarchie est fausse.
- **I — Interface Segregation** : préfère plusieurs petites interfaces ciblées à une grosse interface fourre-tout. Pas de méthodes inutilisées imposées aux implémentations.
- **D — Dependency Inversion** : dépends d'abstractions, pas d'implémentations concrètes. Le domaine définit l'interface, l'infrastructure l'implémente.
- ⚠️ SOLID est un outil pour **discuter d'un design**, pas une checklist à valider à tout prix. YAGNI s'applique aussi à SOLID.

## 6. DRY / KISS / YAGNI — tensions à arbitrer

- **DRY (Don't Repeat Yourself)** : une connaissance métier doit avoir **une seule** représentation autoritaire. Attention : la duplication **apparente** n'est pas toujours de la duplication de connaissance — deux règles métier différentes qui se ressemblent aujourd'hui peuvent diverger demain.
- **KISS (Keep It Simple)** : la solution la plus simple qui répond au besoin actuel gagne. Une abstraction prématurée coûte plus que la duplication qu'elle évite.
- **YAGNI (You Aren't Gonna Need It)** : n'ajoute pas un point d'extension, un paramètre, un flag pour un besoin futur hypothétique. Ajoute quand le besoin est réel.
- **Règle de 3** : tolère une duplication, soupçonne à la deuxième, refactor à la troisième. Avant 3 occurrences, on ne sait pas encore quelle est la bonne abstraction.

## 7. Tests

- **Pyramide de tests** : majorité d'unitaires (rapides, ciblés), une couche d'intégration (composants ensemble), un peu d'end-to-end (parcours critiques).
- **F.I.R.S.T.** :
  - **F**ast : un unitaire < 100 ms. Pas de réseau, pas de FS, pas de DB réelle.
  - **I**ndependent : pas d'ordre imposé, pas d'état partagé entre tests.
  - **R**epeatable : même résultat dans tous les environnements (dev, CI, prod-like).
  - **S**elf-validating : pass/fail, pas d'inspection humaine de logs.
  - **T**imely : écrits au moment où ils servent à guider le code (idéalement juste avant ou pendant l'implémentation).
- **Structure d'un test : Arrange / Act / Assert** (ou Given / When / Then). Une seule logique d'assertion conceptuelle par test.
- **Le test est de la documentation exécutable** : nomme-le pour qu'il décrive le comportement (`should_reject_login_when_password_expired`).
- **TDD (red / green / refactor)** : laisse-toi guider par un test qui échoue, fais passer minimalement, refactore avec le filet de sécurité.
- **Mocker l'infrastructure, pas le domaine** : on mocke les ports externes (DB, HTTP, horloge), on teste le domaine pour de vrai.
- **Tests = code de production** : applique-leur les mêmes règles de propreté (nommage, fonctions courtes, pas de magie). Un test sale tue la confiance.

## 8. Typage

- **Strict par défaut** dans les langages qui le permettent. En TypeScript : `"strict": true` (active `strictNullChecks`, `noImplicitAny`, etc.).
- **Pas d'`any` / `unknown` non typé** : si tu ne sais pas, c'est `unknown` + narrowing ; jamais `any` pour faire taire le compilateur.
- **Modélise les invariants dans le type** : préfère `NonEmptyArray<User>` à `User[]` + assertion runtime. Préfère un union de variants discriminés à un objet aux champs optionnels qui se contredisent.
- **Pas de types primitifs là où une notion métier existe** (Primitive Obsession) : `Email`, `UserId`, `Money` plutôt que `string`, `string`, `number`.
- **Annoter les frontières** : signatures publiques, retours d'API, types des entrées externes — l'inférence va bien à l'intérieur.
- **`satisfies`** (TS) pour vérifier la conformité à un type sans en élargir la forme inférée.

## 9. Gestion d'erreurs

- **Préférer les exceptions aux codes de retour** pour la logique métier / applicative : ça sépare le chemin nominal du chemin d'erreur et évite les `if (rc != 0)` partout. Codes de retour acceptables pour systèmes bas niveau / temps réel.
- **Échouer vite, échouer fort** : valider à la frontière (entrée user, API externe), faire confiance à l'intérieur. Pas de validations défensives à chaque appel interne.
- **Ne jamais avaler une exception** : `catch` vide ou `catch (e) { /* ignore */ }` interdit sans commentaire justifiant pourquoi c'est correct. Préfère laisser remonter.
- **Wrapper les exceptions des libs tierces** au niveau de l'adapter — la couche métier ne doit pas connaître `psycopg2.OperationalError`.
- **Messages d'erreur informatifs** : opération qui a échoué + contexte utile (sans secrets).
- **Pas de fallbacks silencieux** : revenir à une valeur par défaut en absorbant une erreur masque les bugs en prod. Soit on traite explicitement, soit on remonte.
- **Erreurs métier vs erreurs techniques** : modélise les erreurs attendues (utilisateur introuvable, panier vide) comme des **résultats** typés (`Result<T, E>`, union discriminée). Réserve les exceptions aux situations vraiment exceptionnelles.

## 10. Code smells (déclencheurs de refactor)

- **Long Method / Long Class** : trop long → découper.
- **Long Parameter List** : encapsule en objet paramètre.
- **Duplicated Code** : extraire une fonction / une classe (après la règle de 3).
- **Feature Envy** : une méthode utilise plus une autre classe que la sienne → la déplacer.
- **Data Clump** : trois mêmes paramètres baladés ensemble → ils forment un objet implicite, donne-lui un nom.
- **Primitive Obsession** : usage abusif des types primitifs pour modéliser du métier.
- **Switch / chaîne d'`instanceof`** sur un type → polymorphisme ou pattern matching.
- **Shotgun Surgery** : une modif simple touche 10 fichiers → cohésion mal placée.
- **Divergent Change** : un fichier modifié pour des raisons sans rapport → SRP violé, scinder.
- **Comments to explain bad code** : refactor le code, pas le commentaire.
- **Dead Code** : supprimer (git garde l'historique).
- **Speculative Generality** : abstraction "au cas où" jamais utilisée → YAGNI, supprimer.

## 11. Refactor & maintenance

- **Boy Scout Rule** : laisse le code dans un état un peu meilleur que celui où tu l'as trouvé, dans le périmètre de ta tâche — pas de refactor géant non demandé.
- **Refactor sous filet de tests** : pas de refactor sans couverture suffisante de la zone touchée.
- **Petits pas** : commits atomiques, chaque étape laisse le code compilable et les tests verts.
- **Séparer refactor et changement de comportement** : ne fais jamais les deux dans le même commit/PR.

## 12. Ce qu'on ne fait PAS

- Pas d'over-engineering pour un besoin futur hypothétique.
- Pas de validation/handling pour des scénarios qui ne peuvent pas se produire (faire confiance aux garanties internes du langage/framework).
- Pas de feature flags ni de shims de compatibilité quand on peut juste changer le code.
- Pas d'abstractions pour 2 occurrences ; attendre la 3ᵉ.
- Pas de wrapper "au cas où" autour d'une lib bien typée.
- Pas de code commenté laissé en place.
