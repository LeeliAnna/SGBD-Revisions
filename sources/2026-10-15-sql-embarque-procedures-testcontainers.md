| Champ | Valeur |
|---|---|
| Matière | Développement .NET / C# : accès aux données avec ADO.NET (intitulé exact à préciser) |
| Thème de la séance | Sortir les ordres SQL du code (fichiers `.sql` incorporés, procédures stockées), puis tests d'intégration sur un vrai SQL Server avec Docker et Testcontainers |
| Durée | 2 h 50 d'enregistrement, avec du contenu de [00:17] à [02:50]. Le cours est interrompu de [01:52] à [02:04] par une panne du PC. |
| Densité du cours | moyenne |
| Fiabilité de la transcription | moyenne |

---

## 2. Résumé exécutif

La séance prolonge le CRUD ADO.NET (insert, update, delete, select) de la semaine précédente. Elle poursuit deux objectifs : retirer les ordres SQL du code C# et tester réellement l'accès aux données. Dans un premier temps, le professeur range chaque requête dans un fichier `.sql` placé dans un dossier `SQL` du projet repository. Il règle l'action de génération de ce fichier sur **Embedded Resource** : le fichier est alors incorporé dans la DLL et n'apparaît plus dans le dossier de sortie. Une fonction `LoadFromAssembly` relit ensuite le manifeste de l'assembly, trouve la ressource dont le nom se termine par `AddProduct.sql` et en renvoie le contenu. L'intérêt est la lisibilité des requêtes longues, et le client ne voit plus le SQL. Dans un deuxième temps, il remplace la requête du delete par une **procédure stockée** `Product_Delete`, appelée en `CommandType.StoredProcedure`. Il la compare aux requêtes dynamiques : elle est compilée, réutilisable, modifiable sans redéploiement, et elle forme une couche d'abstraction devant les tables. La dernière partie, la plus longue, traite des tests. Une base de test partagée pose problème : accès depuis GitHub, données polluées, structure à recréer. La solution est de démarrer SQL Server dans un conteneur Docker avec le package Testcontainers et l'interface `IAsyncLifetime`. Chaque test démarre un conteneur, crée une base dédiée (jamais `master`) avec ses tables et ses procédures, puis vérifie le comportement, notamment l'unicité du nom. Un de ces tests révèle un vrai bug : à cause de `SET NOCOUNT ON`, le delete renvoie toujours faux. La séance se termine par une pull request générée par Claude, que GitHub valide avec 23 tests, et par un devoir sur une entité `Manager`.

---

## 3. Plan du cours

```
1. Introduction et objectifs de la séance [00:17]
   1.1 Rappel : CRUD ADO.NET de la semaine passée [00:19]
   1.2 Usage annoncé de Claude comme assistant de code [00:19]
2. Externaliser les ordres SQL dans des fichiers .sql [00:20]
   2.1 Dossier SQL et fichier AddProduct.sql [00:21]
   2.2 Action de génération « Embedded Resource » [00:24]
   2.3 Fonction LoadFromAssembly [00:25]
   2.4 Débogage et inspection du manifeste [00:34]
   2.5 Généralisation (Delete, GetAll, GetById) et intérêt de la méthode [00:41]
3. Procédures stockées [00:52]
   3.1 Création de Product_Delete et appel en CommandType.StoredProcedure [00:53]
   3.2 Procédures stockées vs requêtes dynamiques [00:57]
   3.3 Couche d'abstraction : l'exemple de l'étudiant [01:00]
4. Tester l'accès aux bases de données [01:03]
   4.1 Les problèmes d'une base de test partagée [01:04]
   4.2 Docker et images SQL Server [01:06]
   4.3 Démonstration Docker Desktop, application console en conteneur [01:11]
5. Tests d'intégration avec Testcontainers [01:18]
   5.1 Package Testcontainers et interface IAsyncLifetime [01:18]
   5.2 Exécution : un conteneur SQL Server par test [01:42]
   5.3 Interruption technique (panne du PC) [01:46]
   5.4 Reprise et tests de contraintes (unicité, null) [02:05]
   5.5 Créer une base dédiée plutôt que master [02:15]
   5.6 Tester la procédure stockée : le piège SET NOCOUNT ON [02:21]
6. Pull request et intégration continue sur GitHub [02:31]
7. Devoir et perspectives [02:35]
```

---

## 4. Points clés par thème

#### Externaliser les requêtes SQL dans des fichiers incorporés

- Le professeur n'aime pas les requêtes paramétrées écrites en dur dans le repository. Il les place dans un dossier `SQL` du projet repository, à raison d'un fichier par requête, par exemple `AddProduct.sql` [00:20].
- Le fichier se crée comme un fichier texte (`Add New Item` → Text File), puis on le renomme en `.sql`. L'extension `.txt` fonctionnerait aussi, mais `.sql` est préférable [00:22].
- Comme pour `appsettings.json`, un fichier non configuré reste dans le dossier des sources lors de la compilation en Debug ou en Release. Il faut donc régler sa propriété d'action [00:23 – 00:24].
- Avec l'option **Embedded Resource**, le fichier est copié *dans* la DLL (l'assembly). On ne trouve aucun fichier `.sql` dans `bin/Debug` : le SQL est contenu dans `DemoRepo.exe` [00:24, 00:49].
- Avantage annoncé : quand on livre le programme à un client, celui-ci ne voit pas les ordres SQL. À l'inverse, un fichier de configuration doit rester modifiable [00:50].
- Avantage principal : la lisibilité. Les requêtes réelles sont longues, parfois plusieurs ordres dans un même fichier, et deviennent illisibles dans une chaîne C#. Le professeur le montre sur un projet de production [00:35, 00:51].
- Un copier-coller d'un fichier `.sql` conserve l'action Embedded Resource. Après création de `DeleteProduct.sql`, `GetAllProduct.sql` et `GetProductById.sql`, le manifeste contient 4 ressources [00:41 – 00:49].
- L'update n'a pas été converti pendant la séance [00:48].

> **En une phrase :** chaque requête vit dans son propre fichier `.sql` incorporé à la DLL : le code est plus lisible et le SQL reste invisible pour le client.

#### Chargement d'une ressource depuis l'assembly

- La fonction `LoadFromAssembly(string fileName)` récupère l'assembly en cours, liste ses ressources avec `GetManifestResourceNames()` et choisit celle qui correspond au nom demandé. Elle l'ouvre en flux (stream) et renvoie son contenu sous forme de `string` [00:26 – 00:30].
- Le nom réel d'une ressource est complètement préfixé : `DemoRepo.Repositories.SQL.AddProduct.sql`. Le débogueur l'a montré [00:38].
- La première version cherchait les noms qui *contiennent* le nom donné (`Contains`, à la manière d'un `LIKE '%name%'`). Le professeur la remplace par un *se termine par* (`EndsWith`) appliqué au nom complet avec son extension : `AddProduct.sql` [00:31 – 00:34].
- Le `.sql` dans le nom recherché est « important ». Il faut éviter deux fichiers aux noms ambigus. Il revient au développeur d'être plus précis s'il le faut [00:33 – 00:34].
- Convention adoptée : le fichier porte le même nom que la méthode du repository (`AddProduct` → `AddProduct.sql`), « comme ça on ne se perd pas dans les noms » [00:32].
- Le code a été généré par Claude puis corrigé et expliqué en classe. Le professeur insiste pour qu'on comprenne ce que l'outil produit [00:25, 00:30].
- Le breakpoint n'était pas atteint parce que l'application tournait en Release, puis parce que `appsettings.json` pointait sur l'implémentation *fake*. Il a fallu repasser sur ADO.NET [00:34 – 00:37].

> **En une phrase :** on retrouve une ressource incorporée en cherchant dans le manifeste de l'assembly le nom qui se termine par `NomFichier.sql`, puis on la lit en flux.

#### Procédures stockées et requêtes dynamiques

- Les requêtes construites et envoyées par le code s'appellent des **requêtes dynamiques**. La plupart des frameworks les utilisent [00:52].
- Démonstration : dans la base locale, le professeur crée la procédure `Product_Delete`, qui reçoit un paramètre `@ProductId` typé et exécute le `DELETE` [00:53 – 00:56].
- Côté C#, le code du delete se simplifie : `CommandType = StoredProcedure` à la place de `Text`, et le texte de la commande devient le nom de la procédure, `Product_Delete` [00:56 – 00:57].
- Avantage de performance : une procédure est compilée lors de sa première exécution et connaît d'avance « le chemin le plus rapide ». Une requête dynamique recalcule ce chemin à chaque fois. L'écart n'est visible que sur de gros volumes [00:58].
- Avantages de maintenance : la procédure est réutilisable. On la modifie dans la base sans redistribuer ni redéployer l'application, et la modification vaut « pour tout le monde ». Plusieurs projets ou personnes peuvent la partager [00:58 – 00:59].
- Inconvénients : la gestion devient lourde au-delà de quelques milliers de procédures (« 3000 ou 4000 »). Les approches modernes **Code First**, qui créent la base à la volée depuis les entités, s'accordent mal avec les procédures stockées [00:59 – 01:02].
- Principe défendu par le professeur : une application n'accède jamais directement aux tables. Elle passe par des procédures stockées protégées, qui forment une couche d'abstraction [01:00].
- Exemple de l'étudiant : une table *Personne* (nom, prénom, date de naissance), une table *Étudiant* (matricule) et des tables spécifiques à la promotion sociale et à la haute école. L'application appelle une seule procédure, qui fait les trois inserts. Si le schéma passe à deux tables, seule la procédure change [01:00 – 01:02].

> **En une phrase :** la procédure stockée déplace le SQL dans la base : elle y est compilée, partagée et modifiable sans redéploiement, et elle isole l'application de la structure réelle des tables.

#### Problématique des tests avec base de données

- Le professeur présente le test des accès aux bases de données comme « la partie la plus compliquée ». Les fakes de la séance précédente ne suffisent plus [01:04].
- Premier problème d'une base de test en ligne : les tests lancés sur GitHub devraient pouvoir y accéder [01:04].
- Deuxième problème : sur une base partagée, les inserts de chacun s'accumulent et la base n'est plus dans l'état attendu par les tests suivants. Il faut donc la réinitialiser à chaque test [01:05].
- Troisième problème : la base de test doit avoir la même structure que la production (tables, index, vues, procédures stockées). Si le code appelle une procédure absente, rien ne fonctionne [01:05 – 01:06].
- D'où deux étapes : un **init database** charge toute la structure, puis un **init data** insère uniquement les données nécessaires au test [01:06, 01:22].
- La structure doit être complète, mais les données doivent rester minimales, sans « des cent mille données ». Quand des tables sont liées, il faut parfois remplir d'autres tables pour que la table testée fonctionne [02:20 – 02:21].

> **En une phrase :** tester l'accès aux données exige une base isolée, recréée à l'identique pour sa structure et remplie au strict nécessaire pour chaque test.

#### Docker et conteneurs SQL Server

- Docker fait tourner des programmes sans les installer sur la machine, dans des environnements parallèles réduits au strict minimum nécessaire au programme [01:06 – 01:07].
- Il existe des images Docker de SQL Server. Ce sont elles qui servent pour les tests [01:07].
- Démarrer un SQL Server est l'opération qui coûte le plus cher, en temps comme en mémoire, « comme un moteur qui doit chauffer ». Le paradigme visé est donc *un seul SQL Server pour tous les tests, chaque test créant sa propre base* [01:09 – 01:10].
- GitHub et GitLab ont accès aux images Docker. Ils peuvent charger l'image SQL Server dans leur propre environnement et y exécuter les tests [01:10].
- Docker Desktop permet de lancer et de déboguer des conteneurs en local. Le professeur montre des images prêtes : PostgreSQL, .NET, SQL Server, Node, RabbitMQ [01:11, 01:14].
- Les images sont de petites machines Linux minimales. On les déploie aussi en production sur un serveur Docker : Azure, OVH ou infrastructure interne [01:13 – 01:14].
- Le projet a été créé avec l'option Docker, qui génère un Dockerfile : runtime, construction, publication. L'application console peut tourner dans un conteneur (`demorepo`), qui disparaît à l'arrêt [01:15 – 01:18, 01:38 – 01:39].

> **En une phrase :** Docker fournit à la demande un SQL Server jetable et identique partout, en local comme sur GitHub, mais son démarrage est l'opération la plus coûteuse.

#### Tests d'intégration avec Testcontainers

- Le package NuGet à installer est **Testcontainers.MsSql**. La première tentative a échoué à cause d'un mauvais package (« c'est un problème de package ») [01:18 – 01:19, 01:37, 02:05].
- La classe de test implémente obligatoirement `IAsyncLifetime`, donc deux méthodes : `InitializeAsync` pour la construction et `DisposeAsync` pour la destruction du conteneur [01:23 – 01:24, 02:06].
- `InitializeAsync` démarre le conteneur (`await StartAsync`) et récupère la chaîne de connexion générée par le conteneur, sans gérer soi-même les mots de passe. Elle crée ensuite la base et la table `Product`, puis instancie le repository via la factory de la semaine passée (type ADO.NET + chaîne de connexion) [01:21, 01:24 – 01:25, 02:06].
- `await` est indispensable : sans lui, `StartAsync` rend la main tout de suite et le code continuerait avant que le serveur ait démarré [01:41].
- Convention C# : un champ privé de classe est préfixé par un underscore (`_sqlContainer`) [01:30].
- Docker doit être lancé sur la machine. Sinon les tests restent grisés et ne s'exécutent pas [02:13 – 02:14].
- Avec cette recette, *chaque test* démarre puis arrête sa propre instance SQL Server (noms automatiques comme `practical_margulis`). Cela fonctionne, mais c'est coûteux. Une amélioration est annoncée [01:43 – 01:45, 02:06 – 02:07].
- Le professeur présente le tout comme « de la recette » : on l'applique une fois, comme une recette de gaufres, puis on la réutilise [01:20, 01:37].

> **En une phrase :** Testcontainers démarre un SQL Server Docker dans `InitializeAsync` et le détruit dans `DisposeAsync`, ce qui permet de tester le vrai repository ADO.NET contre une vraie base.

#### Ce que les tests révèlent : contraintes et SET NOCOUNT ON

- Quatre tests découlent directement de la table `Product`. Premier test : un ajout suivi d'un `GetProductById` qui retrouve le produit. Deuxième test : deux produits de même nom, le second doit échouer à cause de l'index unique sur `Name`. Troisième et quatrième tests : `Name` nul, puis `Price` nul, qui doivent échouer à cause des colonnes non nulles [02:07 – 02:09].
- Le test d'unicité attend une exception SQL sur le second ajout et vérifie le numéro et le message d'erreur renvoyés [02:11 – 02:12].
- En C#, `Name` et `Price` ne sont pas déclarés nullables (pas de `?`). Le cas « nul » est donc déjà empêché par le typage, et ce test est implicite [02:14 – 02:15].
- Il ne faut jamais créer les tables de test sur `master`, qui est une base système. On crée d'abord une base dédiée, nommée `SGBD2026` ⚠️ (transcrit : "l'HGBD 2026", "EGBD 2026"), avec la chaîne de connexion du conteneur, qui pointe sur `master`. Puis on modifie l'**Initial Catalog** de cette chaîne pour pointer sur la nouvelle base [02:15 – 02:20].
- Pour tester le delete par procédure stockée, il faut créer `Product_Delete` dans `InitializeAsync`. Le script s'obtient depuis SQL Server, mais il est difficile à lire dans une chaîne C# (guillemets triplés). Le professeur suggère de passer là aussi par des fichiers `.sql` incorporés [02:21 – 02:24].
- La procédure contenait `SET NOCOUNT ON`, qui supprime le renvoi du nombre de lignes modifiées. `ExecuteNonQuery` renvoie alors `-1`, le test `> 0` est toujours faux et le delete renvoie `false` même quand il réussit [02:26 – 02:30].
- Correctif : retirer `SET NOCOUNT ON` de la procédure, dans le test *et en production*. Le test passe ensuite. « Vous voyez que c'est important de faire des tests » [02:30 – 02:31].

> **En une phrase :** les tests d'intégration vérifient les contraintes réelles de la base (unicité, non-nullité) et révèlent des bugs invisibles avec les fakes, comme l'effet de `SET NOCOUNT ON`.

---

## 5. Fiches de révision

---

### 📌 FICHE 1 — Fichier SQL en ressource incorporée (Embedded Resource)
`Thème : Externalisation du SQL` · `Difficulté : ●○○` · `[00:24]`

**Définition**
> Une ressource incorporée (*Embedded Resource*) est un fichier du projet dont le contenu est intégré dans l'assembly (DLL ou EXE) à la compilation. Il n'est pas copié comme fichier séparé dans le dossier de sortie.

**À retenir**
- Un fichier `.sql` par requête, rangé dans un dossier `SQL` du projet repository et nommé comme la méthode (`AddProduct.sql`).
- Sans réglage de l'action de génération, le fichier reste dans le dossier des sources et n'est pas disponible à l'exécution, exactement comme `appsettings.json`.
- Les autres options de copie sont « ne pas copier » et « copier si plus récent ». Avec Embedded Resource, aucun `.sql` n'apparaît dans `bin/Debug`.
- Intérêts : lisibilité des requêtes longues ou multiples, et SQL invisible pour le client.
- Un fichier copié-collé conserve l'action Embedded Resource.

**Formule / schéma**
Arborescence décrite en cours :
`DemoRepo.Repositories/SQL/` → `AddProduct.sql`, `DeleteProduct.sql`, `GetAllProduct.sql`, `GetProductById.sql`. Chaque fichier a l'action de génération *Embedded Resource*, et le contenu se retrouve dans `DemoRepo.exe`.

**Exemple donné en cours**
L'`INSERT` paramétré d'`AddProduct` est déplacé dans `AddProduct.sql`. Dans l'explorateur, le dossier `bin/Debug` ne contient aucun fichier `.sql` : celui-ci est dans `DemoRepo.exe`.

**Piège fréquent** ⚡
Oublier de régler l'action de génération : le fichier existe dans la solution mais pas à l'exécution. Autre point : une ressource incorporée n'est pas un fichier de configuration. Pour la modifier, il faut recompiler.

**Lien avec le reste du cours**
Prolonge le repository ADO.NET et la factory de la séance précédente. La même technique est demandée dans le devoir pour les scripts de test (create table, procédures).

---

### 📌 FICHE 2 — Lecture d'une ressource depuis le manifeste de l'assembly
`Thème : Externalisation du SQL` · `Difficulté : ●●○` · `[00:28]`

**Définition**
> L'assembly contient un manifeste qui décrit tout ce qu'elle renferme, y compris les ressources incorporées. Pour lire une ressource, on recherche son nom complet dans ce manifeste, on ouvre son flux et on lit son contenu en chaîne de caractères.

**À retenir**
- Le nom d'une ressource est préfixé par l'espace de noms et le chemin : `DemoRepo.Repositories.SQL.AddProduct.sql`.
- `GetManifestResourceNames()` liste toutes les ressources incorporées, qui peuvent aussi être des images ou d'autres fichiers.
- Recherche retenue : le nom qui se termine par `NomFichier.sql` (`EndsWith`). L'extension est importante.
- La version avec `Contains` (équivalent de `%name%`) est plus risquée en cas de noms proches.
- La méthode du repository appelle `LoadFromAssembly("AddProduct.sql")` puis passe la chaîne obtenue à la commande SQL.

**Formule / schéma**
Enchaînement décrit oralement :
assembly en cours → `GetManifestResourceNames()` → nom qui se termine par `"AddProduct.sql"` → ouverture du flux de la ressource → lecture complète → `string sql` → `new SqlCommand(sql, …)`.

**Exemple donné en cours**
Au débogueur (F10), le manifeste contient d'abord une seule ressource, `DemoRepo.Repositories.SQL.AddProduct.sql`, puis 4 après ajout des autres fichiers. La variable lue contient bien l'`INSERT`.

**Piège fréquent** ⚡
Deux ressources dont les noms se terminent de la même façon : la recherche devient ambiguë, et c'est au développeur d'être plus précis. Autre piège rencontré : un breakpoint non atteint en mode Release, ou parce que `appsettings.json` pointe sur l'implémentation *fake*.

**Lien avec le reste du cours**
Utilise la configuration `appsettings.json` et la factory vues précédemment. Le professeur suggère de réutiliser cette fonction pour charger les scripts d'initialisation des tests.

---

### 📌 FICHE 3 — Procédure stockée vs requête dynamique
`Thème : Procédures stockées` · `Difficulté : ●●○` · `[00:57]`

**Définition**
> Une procédure stockée est un ensemble d'ordres SQL enregistré et compilé dans la base de données, appelé par son nom avec des paramètres. Une requête dynamique est un texte SQL envoyé par l'application à chaque exécution.

**À retenir**
- Côté ADO.NET : `CommandType = StoredProcedure`, et le texte de la commande est le nom de la procédure (`Product_Delete`).
- La procédure est compilée et son plan d'exécution est connu d'avance. Le gain se voit sur de gros volumes.
- On la modifie dans la base, sans redéployer l'application, et elle peut être partagée entre plusieurs projets.
- Elle sert de couche d'abstraction : l'application ne touche jamais directement aux tables.
- Inconvénients : la gestion devient lourde avec des milliers de procédures, et elle s'accorde mal avec l'approche Code First.

**Formule / schéma**

| Critère | Procédure stockée | Requête dynamique |
|---|---|---|
| Lieu du SQL | Base de données | Code / fichier `.sql` |
| Exécution | Compilée, chemin connu | Chemin recalculé à chaque appel |
| Modification | Dans la base, sans redéploiement | Recompiler et redéployer |
| Partage | Plusieurs projets ou personnes | Propre à l'application |
| Usage typique | Applications « classiques » protégées | Frameworks, Code First |

**Exemple donné en cours**
`Product_Delete(@ProductId)` remplace le `DELETE FROM Product WHERE ProductId = @ProductId`. Exemple d'abstraction : l'ajout d'un étudiant passe par une procédure qui fait trois inserts (*Personne*, *Étudiant*, table propre à l'école). Si le schéma passe à deux tables, l'application ne change pas.

**Piège fréquent** ⚡
Croire que la performance est le seul argument : avec deux enregistrements, on ne voit aucune différence. Autre piège : le code appelle une procédure qui n'existe pas dans la base de test, et rien ne fonctionne.

**Lien avec le reste du cours**
Annonce les applications Code First (après les vacances). Le devoir impose une procédure stockée pour l'update, et la fiche 8 montre l'effet de `SET NOCOUNT ON`.

---

### 📌 FICHE 4 — Stratégie de test avec base de données (init database / init data)
`Thème : Tests d'accès aux données` · `Difficulté : ●●○` · `[01:04]`

**Définition**
> Un test d'accès aux données s'exécute sur une base isolée. On y recrée d'abord toute la structure de production (init database), puis on y insère uniquement les données nécessaires au test (init data).

**À retenir**
- Une base de test partagée pose trois problèmes : l'accès depuis GitHub, la pollution par les inserts des autres tests et la structure à maintenir.
- La base doit être réinitialisée à chaque test.
- La structure doit être complète : tables, index, vues, triggers, procédures stockées.
- Les données doivent être minimales : seulement celles qu'exige le test.
- Avec des tables imbriquées, il faut parfois remplir d'autres tables pour que la table ciblée fonctionne.

**Formule / schéma**
Séquence d'un test : démarrer le serveur → init database (structure) → init data (jeu minimal) → exécution et assertions → destruction.

**Exemple donné en cours**
Pour `Product`, l'init database crée la table avec son index unique sur `Name` et la procédure `Product_Delete`. Chaque test ajoute ensuite lui-même le produit qu'il manipule (« ma base de données est vierge donc je construis ce que je veux tester »).

**Piège fréquent** ⚡
Oublier un élément de structure, une procédure ou un index : le test échoue ou, pire, passe sans vérifier la contrainte. Autre piège : peupler la base avec des volumes inutiles.

**Lien avec le reste du cours**
Remplace les tests avec fakes de la séance précédente. Mis en œuvre avec Docker et Testcontainers (fiches 5 et 6).

---

### 📌 FICHE 5 — Docker et conteneurs SQL Server
`Thème : Tests d'accès aux données` · `Difficulté : ●○○` · `[01:07]`

**Définition**
> Docker exécute des programmes dans des conteneurs, environnements isolés et minimaux créés à partir d'images, sans installer ces programmes sur la machine hôte.

**À retenir**
- Une image est un modèle, par exemple SQL Server, PostgreSQL ou RabbitMQ. Un conteneur est une instance démarrée de cette image.
- Les images sont de petites machines Linux minimales, utilisables en développement (Docker Desktop) comme en production (Azure, OVH ou serveur Docker interne).
- Démarrer un SQL Server est l'opération la plus coûteuse. Il vaut donc mieux un serveur pour tous les tests, avec une base par test.
- GitHub et GitLab savent charger des images Docker et exécuter les tests dans leur propre environnement.
- Un projet créé avec l'option Docker contient un Dockerfile : runtime, construction, publication.

**Formule / schéma**
Démonstration Docker Desktop : onglet *Images* (images disponibles) et onglet *Containers* (instances démarrées ou arrêtées). Lancée en mode Docker, l'application console apparaît comme conteneur `demorepo`, puis disparaît à l'arrêt.

**Exemple donné en cours**
L'application console `DemoRepo` est exécutée une fois en local, puis dans un conteneur Docker avec débogage. Le démarrage est plus long.

**Piège fréquent** ⚡
Docker Desktop doit être lancé, sinon les tests Testcontainers ne s'exécutent pas : ils restent grisés.

**Lien avec le reste du cours**
Base technique de Testcontainers (fiche 6) et de l'exécution des tests SQL Server dans la CI GitHub.

---

### 📌 FICHE 6 — Testcontainers et IAsyncLifetime
`Thème : Tests d'intégration` · `Difficulté : ●●●` · `[01:23]`

**Définition**
> Testcontainers (package `Testcontainers.MsSql`) démarre un conteneur SQL Server pour les tests. `IAsyncLifetime` est l'interface qui impose `InitializeAsync` et `DisposeAsync`, pour construire et détruire de manière asynchrone les ressources de la classe de test.

**À retenir**
- La classe de test implémente obligatoirement `IAsyncLifetime`.
- `InitializeAsync` : `await` du démarrage du conteneur, récupération de sa chaîne de connexion, création de la base et des tables, instanciation du repository via la factory (type ADO.NET + chaîne de connexion).
- `DisposeAsync` : libération du conteneur.
- `await` attend la fin du démarrage. Sans lui, le code continuerait sur un serveur pas encore prêt.
- Avec cette recette, un conteneur est démarré par test, ce qui est coûteux. Une amélioration est annoncée.

**Formule / schéma**
Structure décrite en cours :
- un champ `_sqlContainer` (préfixe underscore pour un champ de classe) construit avec le builder MsSql ;
- dans `InitializeAsync`, la suite : démarrage → `GetConnectionString()` → création de la base et de la table → `factory(ADO.NET, connectionString)` ;
- dans `DisposeAsync`, la destruction du conteneur ;
- ensuite, les méthodes de test.

**Exemple donné en cours**
`AdoNetProductRepositoryTest`. Dans Docker Desktop, on voit à chaque test un conteneur SQL Server démarrer puis s'arrêter (nom automatique du type `practical_margulis`). Les 23 tests passent ensuite sur GitHub, y compris ceux sur SQL Server.

**Piège fréquent** ⚡
Installer un mauvais package : `MsSqlBuilder` reste introuvable malgré le `using`. Autre piège : oublier `await`, ou lancer les tests sans Docker démarré.

**Lien avec le reste du cours**
Met en œuvre la stratégie init database / init data (fiche 4) et teste le repository et la factory des séances précédentes.

---

### 📌 FICHE 7 — Base de test dédiée plutôt que master (Initial Catalog)
`Thème : Tests d'intégration` · `Difficulté : ●●○` · `[02:17]`

**Définition**
> La chaîne de connexion fournie par le conteneur pointe sur `master`, une base système. On l'utilise uniquement pour créer une base de test dédiée, puis on remplace son *Initial Catalog* par le nom de cette base avant de créer les tables et de lancer les tests.

**À retenir**
- On ne crée jamais de tables de test sur `master` : « ça, ça ne se fait pas ».
- Sous SQL Server, il faut être connecté à la bonne base pour que le `CREATE TABLE` s'y applique.
- Il y a deux connexions successives : sur `master` pour le `CREATE DATABASE`, puis sur la base créée.
- Le nom de la base est en dur (`SGBD2026` ⚠️). Son paramétrage est annoncé.

**Formule / schéma**
Chaîne du conteneur (`master`) → `CREATE DATABASE SGBD2026` → copie de la chaîne avec `Initial Catalog = SGBD2026` → `CREATE TABLE Product …` → tests.

**Exemple donné en cours**
Le professeur demande à Claude si le `CREATE TABLE` se fait sur `master` ou sur une autre base. Le code est ensuite corrigé pour créer la base `SGBD2026` puis modifier l'Initial Catalog.

**Piège fréquent** ⚡
Tout « marche » même sur `master`. L'erreur est donc silencieuse : il faut vérifier sur quelle base le script s'exécute réellement.

**Lien avec le reste du cours**
Fait partie de l'init database (fiche 4) dans `InitializeAsync` (fiche 6).

---

### 📌 FICHE 8 — Tests de contraintes et piège SET NOCOUNT ON
`Thème : Tests d'intégration` · `Difficulté : ●●●` · `[02:27]`

**Définition**
> `SET NOCOUNT ON` empêche SQL Server de renvoyer le nombre de lignes affectées. Dans une procédure appelée par ADO.NET, `ExecuteNonQuery` renvoie alors `-1`, et toute logique fondée sur le nombre de lignes modifiées devient fausse.

**À retenir**
- Tests de base sur `Product` : ajout puis lecture par ID, doublon de nom (index unique, exception attendue), `Name` nul, `Price` nul.
- Le test d'unicité vérifie l'exception SQL levée, son numéro et son message.
- `Name` et `Price` non nullables en C# (pas de `?`) : le cas nul est déjà bloqué par le typage, le test est implicite.
- Avec `SET NOCOUNT ON`, la condition `lignes > 0` est toujours fausse et le delete renvoie `false` même en cas de succès.
- Correctif : retirer `SET NOCOUNT ON`, dans la base de test et aussi en production.

**Formule / schéma**
$$ \texttt{SET NOCOUNT ON} \Rightarrow \texttt{ExecuteNonQuery()} = -1 \Rightarrow (-1 > 0) = \texttt{false} $$

**Exemple donné en cours**
Le test du delete ajoute un produit puis le supprime via `Product_Delete`. Il échoue tant que la procédure contient `SET NOCOUNT ON`, et passe une fois la ligne retirée. Un autre test vérifie qu'un produit inexistant renvoie `false`.

**Piège fréquent** ⚡
Le bug existe aussi en production, mais les fakes ne peuvent pas le révéler : seul un test sur une vraie base le met en évidence. Autre piège : un test qui « passe » pour la mauvaise raison, comme le cas « produit inexistant → false » qui passait aussi avec le bug.

**Lien avec le reste du cours**
Relie les procédures stockées (fiche 3) aux tests Testcontainers (fiche 6). Le devoir demande les mêmes tests de contraintes pour `Manager`.

---

## 6. Signaux examen 🎯

| Élément | Signal donné | Horodatage |
|---|---|---|
| Comprendre et savoir expliquer le code généré par Claude | *« N'oubliez pas qu'à l'examen, il vous faudra expliquer ce que Claudette a fait. »* | [02:48] |
| Comprendre le code plutôt que copier-coller | *« Essayez au moins de comprendre ce qu'il fait, parce que si vous ne comprenez pas ce qu'il fait ça ne va pas aller »* | [02:49] |
| Le fichier `.sql` se comporte comme `appsettings.json` (copie à la compilation) | *« Souvenez-vous que ceci, c'est comme l'appsettings.json »* | [00:23] |
| Rechercher la ressource par nom complet avec extension | *« le .sql est important »* | [00:34] |
| Une application n'accède jamais directement aux tables | *« normalement une application n'accède jamais à la base de données en direct. Jamais aux tables en direct. »* (répété) | [01:00] |
| Coût du démarrage d'un SQL Server : un serveur pour tous les tests, une base par test | *« Est-ce que vous pouvez répéter ce que j'ai dit ? »* | [01:10] |
| `IAsyncLifetime` dans la classe de test | *« ça c'est obligatoire »* | [01:23] |
| Ne jamais créer les tests sur `master` | *« Ça, ça ne se fait pas. Master est une base de données système »* | [02:17] |
| Structure complète, données minimales, tables imbriquées | *« ok ça c'est important »* | [02:21] |
| Créer les procédures stockées dans la base de test | *« il faut la créer, sinon ça ne marche pas »* (répété) | [02:22] |
| Valeur des tests (bug `SET NOCOUNT ON`) | *« Vous voyez que c'est important de faire des tests »* | [02:31] |
| Faire tourner le repo et ses tests | *« il faut vraiment le faire »* | [02:50] |

---

## 7. Informations pratiques

- **Devoir (entité `Manager`)** [02:35 – 02:48] :
  - Table `Manager` : `Id`, `FirstName`, `LastName`, avec un index unique sur le couple nom + prénom.
  - Méthodes : ajouter, supprimer, modifier, rechercher par ID, lister tous les managers.
  - Implémentation *fake* (liste de managers codée en dur) et implémentation ADO.NET.
  - Tests pour tous les cas : valeurs nulles et non nulles, unicité nom + prénom, en fake comme en ADO.NET.
  - Créer aussi les interfaces et une partie console pour tester.
  - Mettre tous les scripts (create table, procédures stockées…) dans des fichiers `.sql` incorporés à l'assembly.
  - Au minimum une procédure stockée, pour l'update.
  - Une règle de nullité du prénom est à respecter ⚠️ (voir section 10).
- **Échéance** : d'abord annoncée pour le lendemain 16 octobre à 18 h, puis reportée au mercredi 21 octobre, 23 h au plus tard ⚠️ (le calcul « 15 + 7 = 22 » prête à confusion) [02:42].
- **Remise** : sur la plateforme de cours (Moodle), projet complet avec l'entièreté de la solution, en archive ⚠️ (transcrit : "en forme asile") [02:43 – 02:44].
- **Ressources** : vidéos des séances et repository du professeur. On peut s'en servir, mais il déconseille le copier-coller [02:41 – 02:42, 02:48].
- **Consigne** : récupérer le repo et y faire tourner les exemples et les tests [02:50].
- **Prochaine séance** : le professeur sera probablement absent la semaine prochaine. Un cours depuis l'aéroport reste possible [01:00, 02:45].
- **Suite du cours** : applications Code First après les vacances. Bases d'exemple Microsoft (Northwind…) avec des dizaines de tables et des centaines d'enregistrements. Gestion des erreurs, transactions [01:00, 02:45 – 02:46].
- **Outil** : nouvelle « Boîte à outils » sur l'intranet unifié des étudiants [01:12 – 01:13].

---

## 8. Lexique

| Terme | Définition courte |
|---|---|
| ADO.NET | Bibliothèque .NET d'accès aux bases de données (connexions, commandes, paramètres). |
| Assembly | Unité compilée .NET (DLL ou EXE) contenant le code, ses définitions et ses ressources. |
| `await` | Attend la fin d'une méthode asynchrone avant de poursuivre l'exécution. |
| Code First | Approche où l'on décrit les entités dans le code et où la base est créée à la volée. |
| `CommandType.StoredProcedure` | Indique à la commande ADO.NET que son texte est le nom d'une procédure stockée. |
| Conteneur | Instance démarrée d'une image Docker. |
| Docker | Plateforme d'exécution d'applications dans des conteneurs isolés et minimaux. |
| Docker Desktop | Application locale pour gérer et déboguer images et conteneurs. |
| Dockerfile | Fichier qui décrit la construction et la publication de l'image d'un projet. |
| Embedded Resource | Action de génération qui intègre un fichier dans l'assembly. |
| `ExecuteNonQuery` | Exécute un ordre SQL sans résultat tabulaire et renvoie le nombre de lignes affectées (ou −1). |
| Factory | Classe qui instancie le repository adéquat (fake ou ADO.NET) selon la configuration. |
| `IAsyncLifetime` | Interface de test imposant `InitializeAsync` et `DisposeAsync`. |
| Image Docker | Modèle prêt à l'emploi (SQL Server, PostgreSQL…) à partir duquel on démarre des conteneurs. |
| Index unique | Contrainte interdisant deux valeurs identiques dans une colonne ou un groupe de colonnes. |
| Init data | Étape qui insère les seules données nécessaires à un test. |
| Init database | Étape qui recrée toute la structure de la base (tables, index, vues, procédures). |
| Initial Catalog | Paramètre de la chaîne de connexion qui désigne la base utilisée. |
| Manifeste | Description du contenu d'une assembly, dont la liste des ressources incorporées. |
| `master` | Base système de SQL Server, à ne pas utiliser pour les données de test. |
| Procédure stockée | Ensemble d'ordres SQL compilé et stocké dans la base, appelé par son nom. |
| Pull request | Demande d'intégration d'une branche, déclenchant revue et tests automatiques. |
| Repository | Classe qui encapsule l'accès aux données d'une entité. |
| Requête dynamique | Texte SQL envoyé par l'application et recalculé par le serveur à chaque exécution. |
| `SET NOCOUNT ON` | Option SQL Server qui supprime le renvoi du nombre de lignes affectées. |
| Testcontainers | Bibliothèque qui démarre des conteneurs Docker (ici SQL Server) pendant les tests. |

---

## 9. Auto-évaluation

1. **Q :** Pourquoi le professeur sort-il les ordres SQL du code C# ?
   <details><summary>Réponse</summary>Pour la lisibilité : les requêtes réelles sont longues, parfois plusieurs ordres dans un fichier, et deviennent illisibles dans une chaîne. Il veut aussi isoler le SQL du code. Avec une ressource incorporée, le client ne voit pas les ordres SQL.</details>

2. **Q :** Que se passe-t-il à la compilation si l'on ne règle pas l'action de génération d'un fichier `.sql` ?
   <details><summary>Réponse</summary>Il reste dans le dossier des sources et n'est pas disponible dans le dossier de sortie (Debug/Release), comme `appsettings.json` non copié.</details>

3. **Q :** Où se trouve un fichier réglé en Embedded Resource après compilation ?
   <details><summary>Réponse</summary>Dans l'assembly (DLL ou EXE, ici `DemoRepo.exe`). Aucun fichier `.sql` n'apparaît dans `bin/Debug`.</details>

4. **Q :** Pourquoi rechercher la ressource avec « se termine par `AddProduct.sql` » plutôt qu'avec « contient » ?
   <details><summary>Réponse</summary>Le nom complet est préfixé (`DemoRepo.Repositories.SQL.AddProduct.sql`). « Contient » peut correspondre à plusieurs fichiers aux noms proches, alors que le nom complet avec son extension `.sql` est plus précis.</details>

5. **Q :** Comment appelle-t-on une procédure stockée avec ADO.NET ?
   <details><summary>Réponse</summary>On fixe `CommandType` à `StoredProcedure`, on met le nom de la procédure (`Product_Delete`) comme texte de commande et on passe les paramètres (`@ProductId`).</details>

6. **Q :** Citez trois avantages et un inconvénient des procédures stockées par rapport aux requêtes dynamiques.
   <details><summary>Réponse</summary>Avantages : compilées et plus rapides sur de gros volumes, modifiables sans redéployer l'application, partageables entre projets, couche d'abstraction devant les tables. Inconvénients : lourdes à gérer par milliers, et mal acceptées par l'approche Code First.</details>

7. **Q :** Quels problèmes pose une base de test partagée en ligne ?
   <details><summary>Réponse</summary>La CI (GitHub) doit pouvoir y accéder. Les inserts des tests s'accumulent et faussent les tests suivants. La structure complète (tables, index, procédures, vues) doit y exister.</details>

8. **Q :** Que contiennent `InitializeAsync` et `DisposeAsync` dans la classe de test Testcontainers ?
   <details><summary>Réponse</summary>`InitializeAsync` : `await` du démarrage du conteneur SQL Server, récupération de la chaîne de connexion, création de la base, des tables et des procédures, instanciation du repository via la factory. `DisposeAsync` : libération du conteneur.</details>

9. **Q :** Pourquoi ne pas créer les tables de test sur `master`, et comment l'éviter ?
   <details><summary>Réponse</summary>`master` est une base système. On s'y connecte uniquement pour créer une base dédiée, puis on modifie l'Initial Catalog de la chaîne de connexion pour pointer sur cette base avant le `CREATE TABLE`.</details>

10. **Q :** Pourquoi le test du delete échouait-il alors que la ligne était bien supprimée, et quel est le correctif ?
    <details><summary>Réponse</summary>La procédure contenait `SET NOCOUNT ON`. `ExecuteNonQuery` renvoyait donc `-1`, la condition `> 0` était toujours fausse et le repository renvoyait `false`. Correctif : retirer `SET NOCOUNT ON` de la procédure, dans les tests et en production.</details>

---

## 10. Zones d'ombre

- Quel est le nom exact de la base de test créée dans `InitializeAsync` : `SGBD2026` ⚠️ (transcrit : "l'HGBD 2026", "EGBD 2026") ? Comment le professeur compte-t-il le paramétrer ?
- Quelle est la règle exacte de nullité pour `Manager` ? La phrase « attention que le prénom peut être nul aussi dans le domaine si on n'est pas non qui peut pas être nulle » ⚠️ est incohérente. Faut-il comprendre « le prénom peut être nul, le nom ne peut pas l'être » ?
- Quelle est l'échéance définitive du devoir ? D'abord le 16 octobre à 18 h, puis « jeudi prochain », puis « mercredi 21 à 23 h maximum » ⚠️.
- Sous quel format remettre la solution : « en forme asile » ⚠️, compris comme une archive `.zip` ?
- Le devoir `Manager` doit-il être un projet séparé de `Product` ou une classe et une interface supplémentaires dans le même projet ? La réponse du professeur était ambiguë [02:41].
- Que signifie la consigne ajoutée « les .sql, un bad at, pour ce qui est SQL » ⚠️ [02:48] ? Est-ce bien « en Embedded Resource » ?
- Quel numéro d'erreur SQL Server le test d'unicité doit-il vérifier ? Il a été lu à l'écran mais pas énoncé.
- Comment passer concrètement à « un seul SQL Server pour tous les tests, une base par test » ? L'amélioration a été annoncée mais pas montrée.
- Que s'est-il dit entre [01:46] et [02:04] ? Il manque des explications à cause de la panne du PC.
- Le code complet de `LoadFromAssembly`, du builder MsSql et du changement d'Initial Catalog a été généré par Claude et affiché à l'écran, sans être dicté. À vérifier dans le repository du professeur.
- La conversion de l'update en fichier `.sql` ou en procédure stockée n'a pas été faite en cours. Y a-t-il un modèle de référence pour la procédure d'update demandée dans le devoir ?
- `SET NOCOUNT ON` faut-il le retirer de toutes les procédures, ou plutôt renvoyer `@@ROWCOUNT` explicitement, comme Claude l'a évoqué [02:28] ?
- Les fichiers `.sql` peuvent-ils aussi servir aux scripts du projet de test, ce qui suppose de charger l'assembly du projet de tests ? Le professeur l'a suggéré sans le montrer [02:24].
