# Cours du 8 octobre — SQL Server, ADO.NET et pattern Factory

### 1. En-tête

| Champ | Valeur |
|---|---|
| Matière | Systèmes de gestion de bases de données (SGBD) : accès aux données en C# / .NET ⚠️ (intitulé exact à préciser) |
| Thème de la séance | Installation de SQL Server, tables, index et collation ; repository ADO.NET (CRUD) ; pattern Factory ; configuration par `appsettings.json` |
| Durée | 2 h 12 |
| Densité du cours | élevée |
| Fiabilité de la transcription | moyenne |

### 2. Résumé exécutif

La séance commence par l'installation d'une nouvelle instance de SQL Server Developer Edition. Deux réglages sont jugés décisifs : la **collation**, à passer en *French, Case Insensitive, Accent Insensitive*, et l'authentification en mode mixte (compte `sa` plus utilisateur Windows). Le professeur crée ensuite la base `sgbd2026`, une table `Products`, puis une table `Manager`. Il montre que le serveur vérifie lui-même les contraintes (NOT NULL, index unique) et qu'une colonne IDENTITY consomme ses valeurs même quand l'insertion échoue. Un exemple sous Excel distingue la clé primaire, qui fixe l'ordre physique des lignes, d'un index, qui est une table logique de pointeurs. La seconde partie revient au projet C# `DemoRepo`. Une classe `AdoNetProductRepository` implémente la même interface `IProductRepository` que le repository factice, ce qui permet de passer à une vraie base sans toucher à la logique du programme. Le professeur présente l'architecture ADO.NET comme le schéma à retenir : base de données, puis provider, puis ADO.NET, puis surcouches (Entity Framework, Dapper), puis application. Il implémente ensuite le CRUD complet avec `SqlConnection`, `SqlCommand` et les trois méthodes `ExecuteReader`, `ExecuteNonQuery` et `ExecuteScalar`. Le cours se termine sur le pattern **Factory** : le programme demande un repository via une énumération, et le type à créer est lu dans un fichier `appsettings.json`. Ce fichier doit être copié dans `bin/Debug`. Ces notions forment le socle des applications de gestion, décrites comme « le cœur du métier ».

### 3. Plan du cours

```
1. Installation et configuration de SQL Server [00:00]
   1.1 Setup Developer Edition, nouvelle instance, Database Engine [00:01]
   1.2 Collation CI_AI et authentification mixte [00:03]
   1.3 Services Windows : arrêt / démarrage de l'instance [00:05]
   1.4 SQL Server Management Studio (SSMS), versions [00:07]
2. Première base et table Products [00:09]
   2.1 Création de sgbd2026 et de la table par script [00:09]
   2.2 Vérification des contraintes par le serveur [00:14]
   2.3 Index unique sur le nom [00:20]
   2.4 Clé technique, IDENTITY et trous dans la numérotation [00:24]
3. Table Manager et index [00:30]
   3.1 Convention de préfixe des colonnes (MNG_) [00:30]
   3.2 Clé primaire et index unique composé [00:33]
   3.3 Clé primaire (ordre physique) vs index (logique) [00:35]
   3.4 Effets de la collation sur l'unicité et les recherches [00:40]
4. Repository ADO.NET [00:45]
   4.1 Issue, branche, Git Extensions [00:46]
   4.2 AdoNetProductRepository et package Microsoft.Data.SqlClient [00:49]
   4.3 Architecture ADO.NET [00:55]
   4.4 Connexion et connection string [01:00]
   4.5 GetAllProducts, GetProductById (ExecuteReader) [01:05]
   4.6 Delete, Add, Update (ExecuteNonQuery, ExecuteScalar) [01:23]
   4.7 Commit, pull request, CI [01:33]
5. Pattern Factory et configuration [01:35]
   5.1 Fusion de main dans la branche factory [01:36]
   5.2 Énumération RepositoryType et ProductRepositoryFactory [01:38]
   5.3 Passage de la connection string [01:43]
   5.4 appsettings.json et ConfigurationBuilder [01:48]
   5.5 Copy if newer, bin/Debug et Release [01:59]
6. Annonces et questions [02:06]
```

### 4. Points clés par thème

#### Installation et configuration de SQL Server

- On installe la **Developer Edition** en mode *Custom*. Une machine peut héberger plusieurs instances : une instance par défaut (`MSSQLSERVER`) et des instances nommées, par exemple `SQL2026` [00:02–00:03].
- Seule la fonctionnalité *Database Engine Services* est nécessaire. Il faut décocher l'extension Azure. Le service Database Engine démarre automatiquement, alors que SQL Server Agent n'est pas utile pour l'instant [00:02–00:03].
- La collation proposée par défaut est *French, Case Insensitive, Accent Sensitive* (`French_CI_AS`). Il faut choisir *Accent Insensitive* (`French_CI_AI`) pour que « frère » et « frere », ou « Vincent » et « vincent », soient équivalents dans les recherches [00:03–00:04].
- L'authentification doit passer en *Mixed Mode*. On définit un mot de passe pour l'administrateur `sa` et on ajoute l'utilisateur Windows courant. En authentification Windows seule, la connexion par utilisateur et mot de passe n'est plus possible [00:04–00:05].
- Chaque instance tourne comme un **service Windows** (Services, « SQL Server (nom d'instance) »). On peut l'arrêter et la redémarrer depuis SSMS ou depuis la console Services. Le professeur demande de savoir « jongler » avec ces opérations [00:05–00:07].
- SSMS (SQL Server Management Studio) s'installe séparément. Une ancienne version de SSMS ⚠️ (transcrit : "version 19") produit une erreur avec SQL Server 2025. Il faut donc utiliser la nouvelle version, qui offre les mêmes fonctions avec une autre interface [00:12–00:19].
- Le professeur travaille avec l'interface en anglais et refuse la version française [00:23–00:29].

> **En une phrase :** une instance correctement configurée suppose une collation CI_AI, une authentification mixte avec `sa` et un service Windows que l'on sait arrêter et démarrer.

#### Tables, contraintes et clés

- La base `sgbd2026` contient une table `Products` dans le schéma `dbo`, avec trois colonnes : `ID` (clé primaire), puis `Name` et `Price`, toutes deux `NOT NULL` [00:09–00:11].
- Les contraintes sont vérifiées **par le serveur**, pas par le programme. Un `INSERT` sans prix est rejeté et le moteur renvoie l'erreur [00:14–00:16].
- Le professeur crée un index unique `IX_Products_UniqueName` sur `Name` depuis le designer (*Indexes/Keys*, *Is Unique = Yes*). Un second produit « test » est alors refusé pour clé dupliquée [00:20–00:22].
- La clé primaire est un champ **technique** (`ID`, entier). Elle n'est pas une donnée business comme le nom ou le prix, même si le nom est unique [00:24–00:25].
- Avec *Identity Specification = Yes*, l'ID s'incrémente automatiquement. On ne le fournit donc pas dans l'`INSERT` [00:25].
- Une insertion qui échoue consomme quand même une valeur d'identité. Dans la démonstration, les ID obtenus sont 1, 4 et 8, et c'est le système qui gère ces trous [00:25–00:27].
- Convention de nommage : chaque colonne est préfixée par 3 ou 4 lettres de la table (`MNG_ID`, `MNG_FirstName`, `MNG_LastName`), avec des noms anglais, pour que les colonnes restent parlantes. La clé primaire s'appelle `PK_Manager` [00:30–00:34].
- Dans `Manager`, `MNG_FirstName` est un `varchar(50)` qui accepte NULL et `MNG_LastName` un `varchar(50)` NOT NULL. Le professeur précise qu'on prend `nvarchar` pour l'Unicode, sinon `varchar` [00:32–00:33].

> **En une phrase :** le serveur garantit seul l'intégrité (NOT NULL, unicité, identité), sur la base d'une clé primaire technique et auto-incrémentée.

#### Index et collation

- La clé primaire détermine l'**ordre physique** des enregistrements dans la table [00:35–00:36].
- Un index est une table logique, tenue à côté, qui trie une autre colonne et pointe vers les lignes physiques. Une recherche par index fait donc un « chemin plus long » qu'une recherche par clé primaire [00:37–00:39].
- Si l'on change de clé primaire (par exemple vers `plaque` dans l'exemple Excel), l'ordre physique des lignes change et les ID ne sont plus dans l'ordre [00:39–00:40].
- Un index unique composé sur `MNG_FirstName` + `MNG_LastName` interdit deux managers portant le même couple prénom/nom [00:33–00:34, 00:40].
- Avec la collation CI_AI, « Toto », « toto » et « TOTO » sont identiques pour l'index unique : la deuxième insertion échoue [00:41].
- La collation s'applique aussi aux requêtes. `SELECT * FROM Manager WHERE MNG_LastName = 'toto'` renvoie tous les « Toto », quelle que soit la casse. Avec une collation sensible à la casse, ce ne serait pas le cas [00:42–00:43].

> **En une phrase :** la clé primaire est physique, l'index est logique, et la collation décide de ce que l'unicité et les recherches considèrent comme « égal ».

#### Architecture et objets ADO.NET

- La couche d'accès s'ajoute avec le package NuGet `Microsoft.Data.SqlClient`. C'est la couche la plus basse proposée par Microsoft [00:51–00:52].
- Le schéma à retenir se lit de bas en haut [00:56–00:58] :
  - les bases de données ;
  - les providers (drivers) : OLE DB (générique), ODBC (ancienne technologie), SQL Server (le plus rapide pour SQL Server) ;
  - ADO.NET, la couche d'abstraction ;
  - les surcouches : Entity Framework, Dapper, DataSet ;
  - l'application .NET.
- Toutes les surcouches reposent sur ADO.NET. « Quand on a compris comment fonctionne ADO.NET, on a tout compris » [00:59].
- `SqlConnection` représente la connexion. L'interface générique correspondante est `IDbConnection` [01:00–01:01].
- Créer l'objet connexion n'ouvre rien. La connexion réelle, et donc les erreurs (serveur, base, utilisateur, mot de passe), n'arrivent qu'au `Open()` [01:09–01:10, 01:16].
- On ouvre la connexion le plus tard possible et on la ferme le plus tôt possible, dans chaque méthode [01:22–01:23].
- `SqlCommand` prépare, envoie et exécute un ordre SQL. On lui associe une connexion, un `CommandType.Text` et des paramètres [01:06–01:07].
- `SqlDataReader` est l'objet le plus rapide : il est en lecture seule et *forward only*. Il ne connaît pas à l'avance le nombre de lignes [01:07, 01:22].

> **En une phrase :** l'application passe par ADO.NET, qui passe par un provider pour atteindre la base ; maîtriser `SqlConnection`, `SqlCommand` et `SqlDataReader` revient à maîtriser tout l'accès aux données .NET.

#### CRUD dans AdoNetProductRepository

- `AdoNetProductRepository` est une copie de `FakeProductRepository` qui implémente `IProductRepository`. Grâce à cette abstraction, seule la ligne d'instanciation change dans le programme [00:50, 01:33].
- `GetAllProducts` : ouvrir la connexion, `ExecuteReader`, boucle `while (reader.Read())` qui construit un `Product` (ID, Name, Price) et l'ajoute à la liste, fermeture, retour de la liste [01:05–01:08].
- `GetProductById` : requête paramétrée `@Id` alimentée par `Parameters.AddWithValue`, puis `if (reader.Read())`. La méthode renvoie `null` si l'ID n'existe pas, par exemple 10 alors que les ID sont 1, 4 et 8 [01:18–01:21].
- Les requêtes paramétrées sont appelées « queries dynamiques ». Entity Framework et les autres outils les utilisent en permanence [01:18].
- `Delete` et `Update` utilisent `ExecuteNonQuery`, qui renvoie le nombre de lignes affectées. Le résultat `> 0` donne le booléen de retour [01:23–01:26, 01:31].
- `AddProduct` utilise `INSERT … OUTPUT INSERTED.ID` avec `ExecuteScalar`, qui ne renvoie qu'une valeur : le nouvel ID (9 dans la démonstration). L'ID passé dans l'objet n'est pas utilisé [01:27–01:29].
- Un second ajout du même nom provoque l'erreur *Cannot insert duplicate key* venue de l'index unique. La gestion des erreurs sera vue plus tard [01:29–01:30].
- Dans la connection string, le nom de la base n'est pas sensible à la casse, alors que le mot de passe l'est [01:14].

> **En une phrase :** le CRUD ADO.NET suit toujours le même schéma (commande paramétrée, ouverture, exécution adaptée, fermeture) ; seule la méthode `Execute…` change selon que l'on lit, modifie ou récupère une valeur unique.

#### Git, pattern Factory et configuration

- Le travail suit le flux suivant : une issue GitHub, une branche créée depuis l'issue, un *fetch* et un *checkout* dans Git Extensions, puis commit, push, pull request (tests CI exécutés) et merge [00:46–00:49, 01:33–01:35].
- Quand `main` est en avance, on fusionne `main` dans la branche de travail (*merge main into factory*). Il faut mettre ses branches à jour régulièrement quand on travaille à plusieurs [01:36–01:37].
- Pattern **Factory** : le programme client n'est pas responsable de la création de l'instance. Il demande à une factory, qui crée le bon objet [00:47–00:48].
- L'énumération `RepositoryType` vaut `Fake` ou `AdoNet`. La méthode `ProductRepositoryFactory.Create(RepositoryType, string? connectionString = null)` renvoie un `IProductRepository`, et le programme ignore la classe concrète [01:39–01:46].
- La connection string est passée au constructeur d'`AdoNetProductRepository`, qui lève une exception si elle est vide ou nulle. Changer de serveur ou d'utilisateur revient à changer cette chaîne [01:43–01:46].
- `appsettings.json` contient la section `Repository:Type` et la section `ConnectionStrings:DefaultConnection`. Le fichier est lu par `ConfigurationBuilder`, avec `optional: false` et `reloadOnChange: true` [01:49–01:55].
- La chaîne lue est convertie en énumération par `Enum.Parse` avec `ignoreCase: true`. En passant la valeur de `AdoNet` à `Fake` dans le fichier, le programme utilise le repository factice sans recompilation de la logique [01:57–02:01].
- Le fichier doit avoir la propriété *Copy to Output Directory = Copy if newer*. Sinon il n'est pas copié dans `bin/Debug/net10.0`, où l'exécutable cherche ses fichiers [01:59, 02:04–02:05].

> **En une phrase :** la factory et la configuration externe découplent le programme de l'implémentation : un mot dans `appsettings.json` choisit entre base réelle et données factices.

### 5. Fiches de révision

---

### 📌 FICHE 1 — Collation SQL Server
`Thème : Installation et configuration` · `Difficulté : ●○○` · `[00:03]`

**Définition**
> La collation est le paramètre qui fixe les règles de comparaison et de tri des chaînes : sensibilité à la casse (CS/CI) et aux accents (AS/AI). Elle détermine ce que le serveur considère comme deux valeurs égales.

**À retenir**
- La collation par défaut proposée, `French_CI_AS`, ne convient pas.
- Il faut choisir `French_CI_AI` : insensible à la casse et aux accents.
- Elle s'applique aux index uniques comme aux clauses `WHERE`.

**Formule / schéma**
$$ \text{CI\_AI} : \text{Toto} = \text{toto} = \text{TOTO}, \quad \text{frère} = \text{frere} $$

**Exemple donné en cours**
Dans `Manager`, l'insertion de « Toto » puis de « toto » échoue sur l'index unique `FirstName + LastName`. `WHERE MNG_LastName = 'toto'` renvoie tous les « Toto » [00:41–00:43].

**Piège fréquent** ⚡
Ne pas confondre CI (casse) et AI (accents). Avec `CI_AS`, « Vincent » et « vincent » sont égaux, mais « frère » et « frere » ne le sont pas. Il est difficile de changer la collation après l'installation : il faut la choisir dès le setup.

**Lien avec le reste du cours**
Elle conditionne le comportement des index uniques et des recherches, et donc les erreurs remontées par ADO.NET.

---

### 📌 FICHE 2 — Clé primaire technique et IDENTITY
`Thème : Tables et contraintes` · `Difficulté : ●○○` · `[00:24]`

**Définition**
> La clé primaire identifie chaque ligne de façon unique. On utilise de préférence une clé technique (un entier sans signification métier) plutôt qu'une donnée business. Avec la spécification IDENTITY, sa valeur est générée automatiquement par incrémentation.

**À retenir**
- On ne fournit pas l'ID dans l'`INSERT`.
- Le nom ou le prix, même uniques, sont des données business et ne servent pas de clé.
- Une insertion échouée consomme quand même une valeur d'identité.

**Formule / schéma**
```sql
-- Products : ID INT IDENTITY PRIMARY KEY, Name NOT NULL, Price NOT NULL
INSERT INTO Products (Name, Price) VALUES ('test', 10.2);
```

**Exemple donné en cours**
Après plusieurs erreurs volontaires (prix NULL, nom dupliqué), les ID valides obtenus sont 1, 4 puis 8. Plus tard, `AddProduct` renvoie l'ID 9 [00:26–00:27, 01:29].

**Piège fréquent** ⚡
Ne pas supposer que les ID sont continus : il y a des trous, et c'est le système qui gère la numérotation. Lors de la démonstration, la clé primaire avait été déplacée par erreur sur `Name` et a dû être remise sur `ID` [00:24].

**Lien avec le reste du cours**
Ces trous expliquent pourquoi `GetProductById(10)` renvoie `null`. `OUTPUT INSERTED.ID` récupère la valeur générée.

---

### 📌 FICHE 3 — Clé primaire (ordre physique) et index (ordre logique)
`Thème : Index` · `Difficulté : ●●○` · `[00:35]`

**Définition**
> La clé primaire impose l'ordre physique de stockage des enregistrements. Un index est une table logique, tenue à côté, qui trie une ou plusieurs colonnes et contient des pointeurs vers les lignes physiques. Un index peut être déclaré unique.

**À retenir**
- La recherche par clé primaire est la plus directe.
- La recherche par index passe par la table d'index puis suit le pointeur.
- Changer de clé primaire réordonne physiquement les lignes.
- Un index unique, simple ou composé, impose une contrainte d'unicité vérifiée par le serveur.

**Formule / schéma**
Démonstration Excel : une colonne ID de 1 à 10, stockée physiquement dans cet ordre, et une colonne `plaque` aux valeurs dans le désordre. L'index sur `plaque` est une liste triée qui renvoie vers les numéros de ligne correspondants. Si `plaque` devient la clé primaire, les lignes sont réordonnées selon `plaque` et les ID ne se suivent plus.

**Exemple donné en cours**
`IX_Products_UniqueName` sur `Name` ; un index unique sur `MNG_FirstName` + `MNG_LastName` dans `Manager` [00:21, 00:34].

**Piège fréquent** ⚡
La clé primaire et l'index unique sont deux objets distincts. Il ne faut pas les confondre même si tous deux interdisent les doublons. Le professeur précise que son exemple est volontairement imagé [00:38].

**Lien avec le reste du cours**
La violation d'un index unique provoque l'erreur *duplicate key* lors de `AddProduct` [01:30].

---

### 📌 FICHE 4 — Architecture ADO.NET
`Thème : Accès aux données` · `Difficulté : ●●○` · `[00:56]`

**Définition**
> ADO.NET est la couche d'abstraction de plus bas niveau de .NET pour accéder à une base de données. Elle fournit toutes les opérations d'interrogation et s'appuie sur un provider (driver) propre à chaque type de base.

**À retenir**
- Les providers sont OLE DB (générique), ODBC (ancien) et SQL Server (le plus rapide pour SQL Server).
- Les surcouches (Entity Framework, Dapper, DataSet) utilisent toutes ADO.NET.
- Côté code, on installe le package `Microsoft.Data.SqlClient`.

**Formule / schéma**
```
Application .NET (console)
        ↓
[Surcouches : Entity Framework, Dapper, DataSet]  (non utilisées pour l'instant)
        ↓
     ADO.NET
        ↓
Provider : SQL Server | OLE DB | ODBC
        ↓
   Base de données
```

**Exemple donné en cours**
L'application console `DemoRepo` passe directement par ADO.NET et le driver SQL Server pour atteindre `sgbd2026`. Le professeur a partagé une photo du schéma dans le chat [00:58].

**Piège fréquent** ⚡
Utiliser le bon provider : on ne passe pas par OLE DB ou ODBC pour SQL Server quand le provider natif existe. ADO.NET est plus lourd à écrire que les surcouches, mais il est indispensable pour les comprendre.

**Lien avec le reste du cours**
Dapper et d'autres surcouches seront vus plus tard [00:59].

---

### 📌 FICHE 5 — Connexion et connection string
`Thème : Accès aux données` · `Difficulté : ●●○` · `[01:00]`

**Définition**
> `SqlConnection` représente une connexion vers une instance SQL Server, décrite par une connection string (serveur, base, utilisateur, mot de passe, options). La connexion n'est établie qu'à l'appel de `Open()`.

**À retenir**
- Le constructeur ne fait rien sur le réseau : les erreurs n'apparaissent qu'au `Open()`.
- On ouvre le plus tard possible et on ferme le plus tôt possible, dans chaque méthode.
- Le nom de la base n'est pas sensible à la casse ; le mot de passe l'est.
- `IDbConnection` est l'interface générique des connexions.

**Formule / schéma**
```
Server=localhost;Database=sgbd2026;User Id=sa;Password=…;TrustServerCertificate=True
```

**Exemple donné en cours**
Une première exécution échoue avec une erreur de certificat. Le professeur ajoute l'option ⚠️ (transcrit : "Trusted certifié", vraisemblablement `TrustServerCertificate=True`). Un mauvais utilisateur ou une mauvaise base provoquent une erreur de login au `Open()` [01:10–01:11, 01:16].

**Piège fréquent** ⚡
Croire qu'une connexion est valide parce que le constructeur passe sans erreur. Ne pas laisser la connection string en dur dans le code : elle doit être passée au constructeur, puis lue dans la configuration.

**Lien avec le reste du cours**
La connection string est transmise par la factory [01:44], puis lue dans `ConnectionStrings:DefaultConnection` [01:55].

---

### 📌 FICHE 6 — SqlCommand et les trois méthodes Execute
`Thème : CRUD ADO.NET` · `Difficulté : ●●●` · `[01:06]`

**Définition**
> `SqlCommand` prépare, paramètre et exécute un ordre SQL sur une connexion. Trois méthodes d'exécution existent selon le résultat attendu : `ExecuteReader` (des lignes), `ExecuteNonQuery` (un nombre de lignes affectées) et `ExecuteScalar` (une seule valeur).

**À retenir**
- `ExecuteReader` sert aux `SELECT`. Il renvoie un `SqlDataReader` en lecture seule et *forward only*, le plus rapide.
- `ExecuteNonQuery` sert à `INSERT`, `UPDATE` et `DELETE`. Il renvoie le nombre de lignes affectées, et `> 0` donne `true`.
- `ExecuteScalar` renvoie une valeur unique, par exemple le nouvel ID via `OUTPUT INSERTED.ID`.
- Les paramètres s'écrivent `@Id`, `@Name`, `@Price` et s'alimentent par `Parameters.AddWithValue`.

**Formule / schéma**
```csharp
var command = new SqlCommand("SELECT ID, Name, Price FROM Products", _connection);
command.CommandType = CommandType.Text;
_connection.Open();
using var reader = command.ExecuteReader();
while (reader.Read()) { /* new Product {...}; list.Add(...) */ }
_connection.Close();

// Delete / Update
command.Parameters.AddWithValue("@Id", id);
return command.ExecuteNonQuery() > 0;

// Add
// INSERT INTO Products (Name, Price) OUTPUT INSERTED.ID VALUES (@Name, @Price)
var newId = command.ExecuteScalar();
```
*(code reconstitué d'après la démonstration orale)*

**Exemple donné en cours**
`GetAllProducts` renvoie les trois produits (ID 1, 4, 8). `GetProductById(10)` renvoie `null` et `GetProductById(4)` renvoie le produit. `AddProduct("new product", …)` renvoie l'ID 9 ; un second ajout identique lève *Cannot insert duplicate key* [01:16–01:30].

**Piège fréquent** ⚡
`Read()` renvoie `true` tant qu'il reste une ligne et `false` en fin de lecture. Il faut utiliser `while` pour une liste et `if` pour un élément unique. Le reader ne connaît pas le nombre d'enregistrements à l'avance : `FieldCount` donne le nombre de colonnes, pas de lignes [01:22].

**Lien avec le reste du cours**
Ces méthodes appliquent le CRUD vu en SQL. Les contraintes du serveur (fiches 2 et 3) remontent ici sous forme d'exceptions, dont la gestion sera vue plus tard.

---

### 📌 FICHE 7 — Repository et pattern Factory
`Thème : Architecture logicielle` · `Difficulté : ●●●` · `[00:47]`

**Définition**
> Le pattern Factory confie la création des instances à une classe dédiée plutôt qu'au programme client. Le client demande un objet typé par une interface (`IProductRepository`) et ignore l'implémentation concrète retournée.

**À retenir**
- `FakeProductRepository` et `AdoNetProductRepository` implémentent la même interface.
- L'énumération `RepositoryType` vaut `Fake` ou `AdoNet`.
- `ProductRepositoryFactory.Create(type, connectionString = null)` contient le seul `new` du dispositif.
- Le constructeur ADO.NET lève une exception si la connection string est vide ou nulle.

**Formule / schéma**
```csharp
public enum RepositoryType { Fake, AdoNet }

IProductRepository repo =
    ProductRepositoryFactory.Create(RepositoryType.AdoNet, connectionString);
```

**Exemple donné en cours**
Avec `RepositoryType.Fake`, aucune connection string n'est nécessaire. Avec `RepositoryType.AdoNet`, on passe la chaîne. Dans les deux cas, le reste du programme fonctionne sans modification [01:41–01:46].

**Piège fréquent** ⚡
La connection string doit être nullable (`string?`) puisque le repository factice n'en a pas besoin. Le contrôle de nullité se fait donc dans le repository ADO.NET.

**Lien avec le reste du cours**
Le pattern prolonge l'abstraction `IProductRepository` des séances précédentes. Il est complété par la lecture du type dans `appsettings.json` (fiche 8).

---

### 📌 FICHE 8 — Configuration par appsettings.json
`Thème : Configuration et déploiement` · `Difficulté : ●●○` · `[01:49]`

**Définition**
> `appsettings.json` est un fichier JSON de configuration lu au démarrage par `ConfigurationBuilder`. Il externalise les paramètres (type de repository, connection strings) sans modifier le code.

**À retenir**
- `GetConnectionString("DefaultConnection")` lit automatiquement la section `ConnectionStrings`.
- La valeur est lue par `configuration["Repository:Type"]`, puis convertie par `Enum.Parse` avec `ignoreCase: true`.
- Le programme lève une exception si une entrée manque.
- Le fichier doit être réglé sur *Copy to Output Directory = Copy if newer*.

**Formule / schéma**
```json
{
  "Repository": { "Type": "AdoNet" },
  "ConnectionStrings": { "DefaultConnection": "Server=localhost;Database=sgbd2026;..." }
}
```
```csharp
var configuration = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .Build();
```
L'exécutable et ses fichiers sont dans `bin/Debug/net10.0` (ou `bin/Release/…`).

**Exemple donné en cours**
Une première exécution échoue car le fichier est en *Do not copy*. Le professeur passe la propriété à *Copy if newer*. En remplaçant `AdoNet` par `Fake` dans le fichier, le débogueur entre dans `FakeProductRepository` [01:59–02:01].

**Piège fréquent** ⚡
Le fichier existe dans les sources mais pas dans `bin/Debug` : l'exécutable ne le trouve pas. Autre erreur rencontrée : une mauvaise clé de section, qui provoque « repository not found » avant correction [01:59–02:00].

**Lien avec le reste du cours**
Ce réglage alimente la factory (fiche 7) et prépare les tests avec une base SQL Server dans Docker la semaine prochaine.

### 6. Signaux examen 🎯

| Élément | Signal donné | Horodatage |
|---|---|---|
| Architecture ADO.NET (providers, ADO.NET, surcouches) | *« c'est vraiment l'image qu'il faut vraiment retenir »*, *« un des schémas qui est le plus important »* | [00:56–00:58] |
| ADO.NET comme base de toutes les surcouches | *« quand on a compris comment fonctionne ADO.NET, on a tout compris »* | [00:59] |
| Collation à l'installation | *« qu'est-ce qui est important ? C'est la collation »* | [00:03] |
| Vérification des contraintes par le serveur | *« il y a une chose importante dans SQL Server »*, *« c'est quelque chose qui est très important »* | [00:14] |
| Gestion de l'instance comme service Windows | *« vous devez absolument pouvoir jongler avec ça »* | [00:07] |
| Pattern Factory | *« cette notion qui est importante pour moi »* | [00:46] |
| Collation et index unique (Toto/toto) | *« la configuration pourrait être très importante »* | [00:41] |
| CRUD avec ADO.NET | *« ça va être le cœur de votre métier »* | [01:32] |
| Mise à jour des branches depuis `main` | *« c'est ce qu'il faut faire quand vous travaillez à plusieurs »* | [01:37] |
| Emplacement des binaires (`bin/Debug`) et Copy if newer | *« c'est quand même important »* | [02:04] |
| Interface en anglais | Un étudiant : *« à l'examen on a intérêt à passer en anglais »* ; le professeur : *« on travaille jamais avec des trucs en français »* | [00:28] |

### 7. Informations pratiques

- La transcription par IA de la séance est en principe interdite sans l'autorisation du professeur et des participants ; elle a été tolérée ici [00:00].
- Logiciels : Visual Studio, SQL Server Developer Edition, SSMS (nouvelle version, compatible SQL Server 2025), Git Extensions, dépôt GitHub `DemoRepo`.
- Packages NuGet : `Microsoft.Data.SqlClient` et les packages de configuration ⚠️ (noms exacts non cités, probablement `Microsoft.Extensions.Configuration.Json`).
- Le script de création de la table `Products` a été partagé dans le chat, ainsi qu'une photo du schéma ADO.NET [00:09, 00:58].
- À préparer : reproduire l'installation de SQL Server (collation `French_CI_AI`, mode mixte), la base `sgbd2026` et les tables `Products` et `Manager`.
- Semaine prochaine : tests du repository ADO.NET avec SQL Server dans Docker, exécuté sur GitHub (CI), et chargement des bases d'exemple de Microsoft pour travailler les jointures [02:08–02:11].
- La semaine suivante, le professeur est absent : deux semaines pour rattraper avant une accélération du rythme [02:10].
- La vidéo du cours reste disponible pour la revoir [02:06].
- Sans Windows (macOS) : SQL Server sous Linux, dans Docker, ou sur Azure [02:07–02:08].
- L'année comprend aussi le projet SGBD ⚠️ (transcrit : "projet GBD") et un projet web [02:06].

### 8. Lexique

| Terme | Définition courte |
|---|---|
| `AdoNetProductRepository` | Implémentation de `IProductRepository` qui accède à SQL Server via ADO.NET |
| ADO.NET | Couche d'accès aux données de plus bas niveau de .NET, au-dessus des providers |
| `appsettings.json` | Fichier JSON de configuration de l'application |
| Authentification mixte (*Mixed Mode*) | Connexion possible par compte Windows ou par compte SQL (utilisateur et mot de passe) |
| Clé business | Donnée métier (nom, prix), déconseillée comme clé primaire |
| Clé primaire (*Primary Key*) | Identifiant unique d'une ligne, qui fixe l'ordre physique de stockage |
| Clé technique | Clé sans signification métier, typiquement un entier IDENTITY |
| Collation | Règles de comparaison et de tri des chaînes (CI/CS, AI/AS) |
| Connection string | Chaîne décrivant le serveur, la base, l'utilisateur, le mot de passe et les options de connexion |
| Copy if newer | Propriété qui copie un fichier dans le répertoire de sortie s'il est plus récent |
| Dapper | Surcouche légère d'accès aux données qui repose sur ADO.NET |
| Entity Framework | Surcouche ORM de Microsoft qui repose sur ADO.NET |
| `ExecuteNonQuery` | Exécute un `INSERT`, `UPDATE` ou `DELETE` et renvoie le nombre de lignes affectées |
| `ExecuteReader` | Exécute un `SELECT` et renvoie un `SqlDataReader` |
| `ExecuteScalar` | Exécute une requête et renvoie une seule valeur |
| Factory (pattern) | Classe chargée de créer les instances à la place du programme client |
| Forward only | Curseur qui ne lit que vers l'avant, sans retour arrière |
| `IDbConnection` | Interface générique des connexions ADO.NET |
| IDENTITY | Spécification d'auto-incrémentation d'une colonne |
| Index | Table logique triée qui pointe vers les lignes physiques |
| Index unique | Index qui interdit les valeurs (ou combinaisons) en double |
| Instance SQL Server | Installation indépendante du moteur, par défaut ou nommée, exécutée comme service |
| `IProductRepository` | Interface qui définit les opérations CRUD sur les produits |
| Merge | Fusion d'une branche dans une autre (ici `main` dans `factory`) |
| `Microsoft.Data.SqlClient` | Package NuGet du provider SQL Server pour ADO.NET |
| ODBC | Ancienne technologie générique d'accès aux bases de données |
| OLE DB | Provider générique d'accès aux données |
| `OUTPUT INSERTED.ID` | Clause SQL qui renvoie l'ID généré par un `INSERT` |
| Provider (driver) | Composant propre à un type de base, utilisé par ADO.NET |
| Pull request | Demande de fusion d'une branche, qui déclenche les tests CI |
| Requête paramétrée | Requête SQL avec des paramètres `@nom` alimentés séparément |
| `sa` | Compte administrateur SQL Server du mode mixte |
| `SqlCommand` | Objet qui prépare et exécute un ordre SQL |
| `SqlConnection` | Objet de connexion à SQL Server |
| `SqlDataReader` | Lecteur de résultats en lecture seule et *forward only*, le plus rapide |
| SSMS | SQL Server Management Studio, outil graphique d'administration |

### 9. Auto-évaluation

1. **Q :** Quelle collation faut-il choisir à l'installation et que signifie-t-elle ?
   <details><summary>Réponse</summary>`French_CI_AI` : French, Case Insensitive, Accent Insensitive. Les majuscules et minuscules d'une part, les lettres accentuées et non accentuées d'autre part, sont considérées comme égales.</details>

2. **Q :** Pourquoi passer l'authentification en mode mixte ?
   <details><summary>Réponse</summary>Pour se connecter avec un utilisateur et un mot de passe SQL (`sa`), en plus du compte Windows. C'est ce qu'utilise la connection string de l'application.</details>

3. **Q :** Pourquoi utiliser une clé primaire technique plutôt que le nom du produit ?
   <details><summary>Réponse</summary>Le nom est une donnée business. Un `ID` entier auto-incrémenté (IDENTITY) est un champ technique stable, généré par le serveur.</details>

4. **Q :** Pourquoi les ID de la table `Products` valent-ils 1, 4 et 8 ?
   <details><summary>Réponse</summary>Chaque tentative d'insertion consomme une valeur IDENTITY, même si elle échoue (prix NULL, nom dupliqué). Les valeurs consommées par les échecs ne sont pas réutilisées.</details>

5. **Q :** Quelle différence y a-t-il entre la clé primaire et un index ?
   <details><summary>Réponse</summary>La clé primaire fixe l'ordre physique des lignes. Un index est une table logique, tenue à côté, qui trie d'autres colonnes et pointe vers les lignes physiques, ce qui allonge le chemin d'accès.</details>

6. **Q :** Avec une collation CI_AI et un index unique sur prénom + nom, que se passe-t-il si l'on insère « Toto » puis « toto » avec le même prénom ?
   <details><summary>Réponse</summary>La seconde insertion échoue (clé dupliquée), car les deux valeurs sont égales pour la collation.</details>

7. **Q :** Décrivez de bas en haut l'architecture d'accès aux données en .NET.
   <details><summary>Réponse</summary>On trouve d'abord la base de données, puis un provider (SQL Server, OLE DB ou ODBC), puis ADO.NET. Au-dessus viennent les surcouches facultatives (Entity Framework, Dapper, DataSet), puis l'application .NET.</details>

8. **Q :** Associez chaque méthode `Execute…` à son usage et à son retour.
   <details><summary>Réponse</summary>`ExecuteReader` sert aux `SELECT` et renvoie un `SqlDataReader`. `ExecuteNonQuery` sert à `INSERT`, `UPDATE` et `DELETE` et renvoie le nombre de lignes affectées. `ExecuteScalar` renvoie une valeur unique, par exemple l'ID inséré via `OUTPUT INSERTED.ID`.</details>

9. **Q :** Pourquoi une erreur de mot de passe n'apparaît-elle pas à la création de la `SqlConnection` ?
   <details><summary>Réponse</summary>Le constructeur se contente de définir la connexion. La connexion réelle n'est tentée qu'à `Open()`, d'où la règle : ouvrir le plus tard possible et fermer le plus tôt possible.</details>

10. **Q :** Expliquez comment la factory et `appsettings.json` permettent de basculer entre données factices et base réelle, et pourquoi le programme a d'abord échoué à lire le fichier.
    <details><summary>Réponse</summary>Le programme lit `Repository:Type` et le convertit en `RepositoryType` avec `Enum.Parse` (`ignoreCase: true`). Il lit aussi `ConnectionStrings:DefaultConnection`, puis appelle `ProductRepositoryFactory.Create(type, connectionString)`, qui renvoie un `IProductRepository` fake ou ADO.NET. Le reste du code ne change pas. Le premier échec venait du fichier réglé sur *Do not copy* : absent de `bin/Debug/net10.0`, il était introuvable pour l'exécutable. Il fallait le régler sur *Copy if newer*.</details>

### 10. Zones d'ombre

- Pourquoi `GetAllProducts` a-t-il d'abord renvoyé une liste vide alors que le reader contenait trois lignes [01:12–01:15] ? La cause n'a pas été expliquée : le code a fonctionné après un nouveau débogage.
- Quelle est l'option exacte ajoutée à la connection string après l'erreur de certificat : `TrustServerCertificate=True` ⚠️ (transcrit : "Trusted certifié") [01:10–01:11] ?
- Quelles versions de SSMS sont concernées ? Le cours cite une « version 19 », une « nouvelle version 2022 » et SQL Server 2025 ⚠️ [00:12–00:18]. Quelle version de SSMS faut-il installer précisément ?
- La collation par défaut proposée est-elle bien `French_CI_AS`, et comment changer la collation d'une instance déjà installée ?
- À [00:40], le professeur parle d'une « primary key sur first name, last name » alors qu'il a créé un index unique. Confirmer qu'il s'agit bien de l'index unique.
- Dans l'exemple Excel, une recherche par index est-elle plus lente ou plus rapide qu'un parcours complet de la table ? L'échange avec l'étudiant reste ambigu sur ce point [00:37–00:38].
- Quels sont les types exacts des colonnes de `Products`, notamment `Price` (`decimal` ?) et `Name` (`varchar` ou `nvarchar`, quelle longueur) ?
- Quels packages NuGet exacts faut-il pour `ConfigurationBuilder` et `AddJsonFile` [01:53–01:54] ?
- Quelle est la syntaxe exacte retenue pour lire le type de repository (`GetSection("Repository:Type").Value` ou `configuration["Repository:Type"]`) après l'erreur « repository not found » [02:00] ?
- L'interface `IDbConnection` sera-t-elle utilisée à la place de `SqlConnection` dans la suite du cours [01:01] ?
- Pourquoi la création visuelle de table (*New Table*) a-t-elle échoué en début de séance [00:10] ? Est-ce un problème d'installation en cours ou de version de SSMS ?
- Comment gérer proprement les exceptions (connexion, clé dupliquée) ? Le point est annoncé pour plus tard [01:17, 01:30].
- Quels sont les noms exacts des projets mentionnés (« projet GBD », « projet la web la caque » ⚠️) [02:06] ?
- La question sur les stages et le TFE a été coupée en fin d'enregistrement [02:12] : à reposer.
