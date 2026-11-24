| Champ | Valeur |
|---|---|
| Matière | Programmation .NET / C# appliquée aux SGBD (cours « SGBD 2026 ») |
| Thème de la séance | Pattern Repository (interface + implémentation factice), tests unitaires xUnit, workflow Git/GitHub (issues, branches, pull requests), intégration continue et protection de branche |
| Durée | 2 h 11 min |
| Densité du cours | élevée |
| Fiabilité de la transcription | moyenne (vocabulaire Git très déformé : « guide » pour *Git*, « mail » pour *main*, « pool request » pour *pull request*) |

## 2. Résumé exécutif

La séance pose les fondations de l'accès aux données en C#, sans encore toucher à une vraie base de données. Le professeur crée une application console en .NET 10. Il y définit un modèle `Product` (Id, Name, Price) et redéfinit sa méthode `ToString`. Il introduit ensuite le **pattern Repository** : une interface `IProductRepository` déclare les opérations CRUD, et une classe `FakeProductRepository` les implémente sur une liste de trois produits codés en dur. Le programme passe toujours par l'interface, pour pouvoir remplacer l'implémentation plus tard sans modifier le code appelant. Le code est ensuite versionné sur GitHub selon un workflow strict : une issue, une branche, des commits, un push, une pull request, puis un merge dans `main`. Le professeur montre aussi comment garder sa branche à jour en y fusionnant régulièrement `main`. Un projet de tests xUnit couvre ensuite tout le repository : `[Fact]`, `[Theory]` avec `[InlineData]`, tests des cas d'erreur et des messages d'exception. Il met en garde contre les données partagées entre tests. Enfin, un workflow GitHub Actions compile et teste le code à chaque push, et une règle de protection empêche toute modification directe de `main`. Cet ensemble est l'enjeu de la séance : la semaine prochaine, une implémentation ADO.NET remplacera le faux repository, et les mêmes tests devront continuer à passer.

## 3. Plan du cours

```
1. Mise en place du projet [00:01]
   1.1 Application console .NET 10 « DemoRepo » [00:01]
   1.2 Modèle Product et override de ToString [00:04]
   1.3 Débogage : breakpoint, F5, F10, F11 [00:07]
2. Pattern Repository [00:08]
   2.1 Interface IProductRepository [00:08]
   2.2 Implémentation FakeProductRepository (liste en mémoire) [00:11]
   2.3 Passage par l'interface et inversion de dépendance [00:15]
   2.4 GetProductById et type nullable [00:18]
   2.5 Delete, Add, Update et scénario complet dans Program [00:21]
3. Workflow Git / GitHub [00:29]
   3.1 Création du dépôt public et push [00:29]
   3.2 Issues, branches, checkout [00:32]
   3.3 Commit local, dossier .git, push, pull request, fetch [00:39]
   3.4 Git Extensions et synchronisation de la branche avec main [00:49]
4. Tests unitaires xUnit [00:56]
   4.1 Projet de test, référence de projet, visibilité public [00:57]
   4.2 [Fact], [Theory], [InlineData], Run et Debug [01:02]
   4.3 Données partagées et isolation des tests [01:09]
   4.4 Tester tous les cas : enum ActionEnum et exceptions [01:20]
   4.5 Mocks (reportés) et merge de la branche [01:45]
5. Intégration continue et protection de main [01:49]
   5.1 GitHub Actions : workflow YAML build + tests [01:49]
   5.2 Démonstration d'une régression détectée par la CI [01:54]
   5.3 Règle de protection de branche [02:00]
6. Bilan et annonce de la séance suivante [02:05]
```

## 4. Points clés par thème

#### Modèle et méthode ToString

- Les classes métier sont rangées dans un dossier `Models`, nommé en anglais. La classe `Product` expose trois propriétés : `int Id`, `string Name` initialisé à `string.Empty`, et `decimal Price` [00:04].
- Les propriétés suivent la convention PascalCase : première lettre en majuscule ⚠️ (transcrit : « l'abonnement nature »). Le snippet `prop` de Visual Studio les génère [00:04].
- Toute classe hérite d'une méthode `ToString`, qu'on peut redéfinir avec `public override string ToString()`. Le `@Override` de Java n'existe pas en C# : `override` est un mot-clé de la signature [00:06].
- La version redéfinie concatène l'Id, le nom et le prix. Un breakpoint posé dedans (F9 puis F5) montre qu'elle est bien appelée au moment de `product.ToString()` [00:07].
- Raccourcis de débogage : F9 pose un breakpoint, F5 lance, F10 avance pas à pas sans entrer dans les méthodes, F11 entre dans la méthode appelée [00:07, 00:20, 01:08].

> **En une phrase :** un modèle est une classe simple à propriétés publiques, et la redéfinition de `ToString` fournit sa représentation textuelle.

#### Pattern Repository et inversion de dépendance

- Un **repository** est la couche qui donne accès aux données. Il se place dans un dossier `Repositories` et commence par une interface [00:08].
- `IProductRepository` déclare `IEnumerable<Product> GetAllProducts()`, `int AddProduct(Product)`, `bool UpdateProduct(Product)`, `bool DeleteProduct(int id)` et `Product? GetProductById(int id)` [00:09–00:10].
- Une classe qui implémente l'interface doit en implémenter toutes les méthodes. Visual Studio génère des corps qui lèvent `NotImplementedException` [00:12–00:13].
- La classe est renommée `FakeProductRepository` : c'est une implémentation de démonstration avec une `private List<Product> _products` de trois produits codés en dur [00:13–00:14].
- Le programme déclare une variable du type de l'interface (`IProductRepository repo = new FakeProductRepository();`). On peut ainsi changer d'implémentation plus tard sans toucher au code appelant : c'est l'**inversion de dépendance** [00:15].
- `GetAllProducts().ToList().ForEach(...)` revient à récupérer un `IEnumerable<Product>` puis à le parcourir avec un `foreach` [00:16].
- La séance suivante remplacera le faux repository par une implémentation ADO.NET, sans modifier les tests [02:06].

> **En une phrase :** le code dépend d'une interface de repository, pas d'une implémentation, pour pouvoir passer de données factices à une vraie base sans rien réécrire.

#### Implémentation des opérations CRUD en mémoire

- `GetAllProducts` retourne directement la liste `_products` [00:14].
- `GetProductById` utilise `_products.Find(p => p.Id == id)`. Le résultat peut être `null`, d'où l'avertissement vert du compilateur. On le corrige en déclarant le retour `Product?` dans l'interface et dans la classe [00:18–00:19].
- L'appelant teste `!= null` : il affiche le produit s'il existe, sinon « Product not found ». C'est vérifié avec l'Id 3 (trouvé) puis l'Id 10 (null) [00:19–00:20].
- `DeleteProduct` utilise `RemoveAll(p => p.Id == id)` et retourne `true` si le nombre d'éléments supprimés est supérieur à 0, `false` sinon [00:21–00:22].
- `AddProduct` ajoute le produit avec `_products.Add(product)` et retourne un entier calculé à partir de `Count + 1` ⚠️ (transcrit : « j'ai le count plus 1… pour retourner le nombre de counts ») [00:22–00:23].
- `UpdateProduct` cherche le produit par son Id et retourne `false` s'il est absent. Sinon, il le supprime puis le réajoute, et retourne `true` [00:23].
- Scénario de démonstration dans `Program` : ajout d'un produit 4, recherche par `newProduct.Id`, modification du nom, `UpdateProduct`, `DeleteProduct`. La liste est réaffichée après chaque opération [00:24–00:28].

> **En une phrase :** le faux repository implémente le CRUD sur une liste, avec des retours booléens ou nullables qui signalent l'échec.

#### Workflow Git et GitHub

- Le projet est ajouté à Git depuis Visual Studio, avec le `.gitignore` Visual Studio par défaut, puis publié sur GitHub comme dépôt public ⚠️ (nom transcrit : « sgbd 2026 demo repo ») [00:29–00:31].
- Une **issue** est une tâche de la to-do list, avec un titre et si possible un contexte et un objectif détaillés. Chaque issue donne lieu à une branche, créée depuis l'issue elle-même [00:33–00:35].
- Dans Visual Studio, « Manage Branches », puis fetch, puis checkout de la nouvelle branche : on ne travaille jamais directement sur `main` [00:33–00:34].
- Le dossier caché `.git` contient le dépôt local. Un **commit** enregistre localement, et un **push** envoie les commits sur le serveur : tant qu'il n'y a pas eu de push, GitHub ne voit rien [00:40–00:42].
- La **pull request** demande l'intégration de la branche dans `main`. On peut continuer à committer dessus avant de la merger, pour n'en ouvrir qu'une seule [00:44–00:45].
- Le fetch rapatrie l'état du serveur dans le dépôt local, et « Fetch & Prune » supprime en plus les références aux branches disparues [00:45, 00:50].
- Le professeur conseille Git Extensions pour visualiser le graphe des branches et effectuer les opérations [00:49–00:50].
- Au moins une fois par jour, il faut merger `main` dans sa branche de travail pour récupérer les modifications des autres. Démonstration sans conflit [00:54–00:55].
- Après le merge de la PR, la branche est supprimée sur le serveur. Localement, elle apparaît alors marquée *gone* et peut être supprimée [01:48–01:52].

> **En une phrase :** chaque tâche suit le cycle issue, branche, commit, push, pull request, merge dans `main`, et la branche de travail est régulièrement resynchronisée avec `main`.

#### Tests unitaires avec xUnit

- On ajoute un projet « xUnit Test Project » à la solution, en .NET 10, puis une référence vers le projet testé (« Add Project Reference ») : sans elle, les classes sont inconnues [00:37, 01:00–01:01].
- Les types testés doivent être `public` (`Product`, `IProductRepository`, `FakeProductRepository`) : par défaut ils sont `internal` [01:01–01:03].
- `[Fact]` marque un test sans paramètre. Les tests apparaissent dans l'**explorateur de tests** et se lancent en Run, ou en Debug pour s'arrêter sur les breakpoints [00:57, 01:08].
- `[Theory]` avec plusieurs `[InlineData(...)]` exécute un même test pour chaque jeu de valeurs, par exemple les Id 1, 2 et 3 [01:06–01:07].
- Assertions utilisées : `Assert.Equal(3, …)` sur le nombre de produits, `Assert.NotNull`, `Assert.Null` (Id 129 inexistant), `Assert.True` et `Assert.False` sur les retours d'Update et de Delete [01:02–01:19].
- Un champ repository partagé convient pour les lectures. En revanche, un test qui supprime ou ajoute modifie les données des autres tests, et l'ordre d'exécution n'est pas garanti : il faut alors créer un `new FakeProductRepository()` local au test [01:09–01:17].
- Onze tests couvrent ainsi le repository. Les relancer à chaque modification détecte les **régressions** [01:19].
- Le professeur recommande d'écrire d'abord le code, puis de tester chaque méthode immédiatement : « sinon après tu ne le fais pas » [01:19–01:20].

> **En une phrase :** chaque méthode est couverte par des tests automatiques xUnit, isolés quand ils modifient les données, et relancés à chaque changement.

#### Couverture de tous les cas, exceptions, CI et protection de branche

- Il faut un test par branche logique d'une méthode : « il faut tester tous tes cas de figure… absolument tous » [01:20–01:21].
- Exemple : une enum `ActionEnum` (Add, Update, Delete, GetAll) et une méthode `ActionOnProduct(ActionEnum)` construite sur un `switch` [01:22–01:29].
  - GetAll appelle `GetAllProducts`.
  - Add, Update et Delete lèvent `NotImplementedException`, chacun avec un message différent.
  - Le cas `default` lève `ArgumentOutOfRangeException`.
- Les trois tests quasi identiques sont fusionnés en une `[Theory]` qui reçoit l'action et le message attendu, et vérifie le type d'exception puis son message exact [01:35–01:43].
- Une valeur invalide, `(ActionEnum)999`, doit provoquer une `ArgumentOutOfRangeException` [01:38].
- Supprimer un simple point dans un message d'exception suffit à faire échouer un test : c'est un test de régression [01:43].
- Vérifier qu'une méthode en appelle une autre exactement une fois demande des **mocks**, reportés à la semaine suivante [01:32–01:45].
- Un workflow GitHub Actions (fichier YAML dans `.github/workflows`, généré par l'IA) compile et lance les tests à chaque push. Une régression volontaire fait échouer la PR, qui repasse au vert après correction [01:49–01:57].
- Une règle de protection de `main` impose trois conditions avant tout merge : passer par une pull request, avoir une branche à jour avec `main`, et réussir le job build/test. Plus aucune modification directe de `main` n'est possible [02:00–02:05].

> **En une phrase :** chaque cas, y compris les erreurs, doit être testé, et la CI couplée à la protection de branche empêche toute régression d'atteindre `main`.

## 5. Fiches de révision

---

### 📌 FICHE 1 — Override de ToString
`Thème : Modèle` · `Difficulté : ●○○` · `[00:05]`

**Définition**
> `ToString` est une méthode héritée par tout objet C#, qui renvoie sa représentation textuelle. On la redéfinit avec le mot-clé `override` dans la signature de la méthode.

**À retenir**
- La syntaxe est `public override string ToString() { return …; }`.
- Elle est appelée explicitement (`product.ToString()`) ou implicitement lors d'un affichage.
- Un breakpoint posé dans la méthode confirme qu'elle s'exécute au moment de l'appel.

**Formule / schéma**
```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }

    public override string ToString() => $"ProductId: {Id}, Name: {Name}, Price: {Price}";
}
```

**Exemple donné en cours**
`Console.WriteLine(product.ToString());` dans `Program`. Avec F9 sur le `return` puis F5, l'exécution s'arrête dans la méthode redéfinie, qui concatène l'Id, le nom et le prix.

**Piège fréquent** ⚡
Un étudiant a proposé « @override » comme en Java. En C#, il n'y a pas d'annotation : `override` fait partie de la signature.

**Lien avec le reste du cours**
Ce `ToString` sert à afficher les produits renvoyés par le repository tout au long de la démonstration.

---

### 📌 FICHE 2 — Interface de repository et inversion de dépendance
`Thème : Pattern Repository` · `Difficulté : ●●○` · `[00:08]` · `[00:15]`

**Définition**
> Un repository est une couche d'accès aux données dont le contrat est défini par une interface. Le code appelant ne manipule que cette interface, ce qui permet de remplacer l'implémentation sans le modifier.

**À retenir**
- Par convention, le nom de l'interface est préfixé par `I` : `IProductRepository`, au singulier.
- Une classe qui implémente l'interface doit implémenter toutes ses méthodes.
- On déclare `IProductRepository repo = new FakeProductRepository();` : le type déclaré est l'interface.
- Une interface utilisée par un autre projet doit être `public`.

**Formule / schéma**
```csharp
public interface IProductRepository
{
    IEnumerable<Product> GetAllProducts();
    int AddProduct(Product product);
    bool UpdateProduct(Product product);
    bool DeleteProduct(int id);
    Product? GetProductById(int id);
}
```
Schéma : `Program` / `Tests` → `IProductRepository` ← `FakeProductRepository` (aujourd'hui) ou repository ADO.NET (séance suivante).

**Exemple donné en cours**
« Je passe toujours par l'interface, et comme ça […] je vais pouvoir modifier par après […] sans devoir changer tous les programmes en amont. »

**Piège fréquent** ⚡
Instancier et typer directement la classe concrète, ce qui lie tout le code à une implémentation. Autre piège : oublier le `using` vers l'espace de noms `Models`, et `Product` apparaît alors en rouge dans l'interface.

**Lien avec le reste du cours**
Les tests passent eux aussi par l'interface. La semaine prochaine, un repository ADO.NET, le pattern Factory et l'injection de dépendance s'appuieront sur ce contrat.

---

### 📌 FICHE 3 — FakeProductRepository : implémentation CRUD en mémoire
`Thème : Pattern Repository` · `Difficulté : ●●○` · `[00:13]`

**Définition**
> Implémentation factice du repository qui stocke les produits dans une liste privée initialisée en dur. Elle permet de développer et de tester sans base de données.

**À retenir**
- Le stockage est un champ `private List<Product> _products = new() { … }` contenant trois produits.
- `GetProductById` retourne `Product?`, puisque `Find` peut renvoyer `null`.
- `DeleteProduct` retourne `RemoveAll(...) > 0`.
- `UpdateProduct` retourne `false` si le produit est absent, et sinon le supprime puis le réajoute.
- `AddProduct` ajoute le produit et retourne un entier basé sur `Count + 1` ⚠️.

**Formule / schéma**
```csharp
public Product? GetProductById(int id) => _products.Find(p => p.Id == id);
public bool DeleteProduct(int id) => _products.RemoveAll(p => p.Id == id) > 0;
```

**Exemple donné en cours**
Dans `Program` : ajout d'un produit 4, recherche par `newProduct.Id`, changement du nom, `UpdateProduct`, puis `DeleteProduct`. La liste passe de 1-2-3 à 1-2-3-4, puis revient à 1-2-3 [00:24–00:28].

**Piège fréquent** ⚡
Ignorer l'avertissement vert sur un retour potentiellement `null`. Il faut rendre le type nullable dans l'interface et dans la classe, puis tester `!= null` chez l'appelant.

**Lien avec le reste du cours**
Ses données codées en dur rendent les résultats prévisibles et donc testables (fiches 6 et 7). L'implémentation ADO.NET le remplacera.

---

### 📌 FICHE 4 — Cycle issue, branche, commit, push, pull request, merge
`Thème : Git / GitHub` · `Difficulté : ●●○` · `[00:33]` · `[00:46]`

**Définition**
> Toute tâche part d'une issue, se développe sur une branche dédiée, est enregistrée localement par des commits, envoyée au serveur par un push, puis intégrée à `main` via une pull request.

**À retenir**
- Une issue est la to-do list du projet : le titre, mais aussi le contexte et l'objectif si possible.
- Le commit est local et stocké dans le dossier `.git`. Le push copie les commits sur le serveur.
- La pull request demande l'intégration dans `main`. On peut y ajouter des commits avant le merge.
- Une fois la PR mergée, on supprime la branche. Localement, elle apparaît alors *gone*.
- Le professeur évaluera le projet SGBD sur l'historique des dépôts.

**Formule / schéma**
Issue → Create branch → (VS) Fetch + Checkout → Commit (local, `.git`) → Push (serveur) → Pull request → Merge dans `main` → suppression de la branche → Fetch & Prune.

**Exemple donné en cours**
L'issue « Création de tests pour FakeProductRepository » donne une branche dédiée, un commit « création test project » et un push. Le professeur montre que le projet n'apparaît sur GitHub qu'après le push [00:42].

**Piège fréquent** ⚡
Croire qu'un commit est visible des autres : sans push, il reste local. Autre piège : travailler directement sur `main`.

**Lien avec le reste du cours**
Ce workflow est le cadre de toutes les séances suivantes. Il est verrouillé par la CI et la protection de branche (fiche 8).

---

### 📌 FICHE 5 — Synchroniser sa branche avec main
`Thème : Git / GitHub` · `Difficulté : ●●○` · `[00:51]`

**Définition**
> Fusionner régulièrement `main` dans sa branche de travail pour y intégrer les modifications des autres développeurs, et réduire ainsi les conflits.

**À retenir**
- Le fetch rapatrie l'état du serveur, et Fetch & Prune nettoie en plus les branches disparues.
- Passer d'une branche à l'autre (checkout) change les fichiers visibles dans Visual Studio.
- Le professeur conseille de merger `main` dans sa branche au minimum une fois par jour.
- S'il y a des conflits, l'outil les signale.

**Formule / schéma**
Graphe Git Extensions : `main` avance (commit « test modif ») → checkout de la branche de travail → « Merge into current branch » depuis `main` → la branche contient les deux historiques.

**Exemple donné en cours**
Sur `main`, ajout de `Console.WriteLine("c'est fini")`, puis commit et push. De retour sur la branche de tests, la ligne est absente. Après le merge de `main` dans la branche, elle réapparaît, sans conflit [00:51–00:55].

**Piège fréquent** ⚡
S'étonner qu'un projet « disparaisse » après un checkout de `main` : il n'existe que sur l'autre branche, ce n'est pas une perte. Sur de gros projets à plusieurs développeurs, les conflits ne sont pas toujours évidents à résoudre.

**Lien avec le reste du cours**
La règle de protection « branche à jour avant merge » rend cette synchronisation obligatoire (fiche 8).

---

### 📌 FICHE 6 — Tests unitaires xUnit : Fact, Theory, Assert
`Thème : Tests` · `Difficulté : ●●○` · `[00:56]` · `[01:06]`

**Définition**
> Un test unitaire xUnit est une méthode marquée `[Fact]`, sans paramètre, ou `[Theory]`, paramétrée par des `[InlineData]`. Elle exécute du code puis vérifie le résultat par des assertions.

**À retenir**
- Il faut un projet xUnit séparé, une référence de projet vers le code testé, et des types `public`.
- `[Theory]` exécute le même test pour chaque `[InlineData]`.
- Assertions vues : `Assert.Equal`, `Assert.NotNull`, `Assert.Null`, `Assert.True`, `Assert.False`, `Assert.Throws<T>`.
- Le Run lance les tests ; le Debug permet de s'arrêter sur un breakpoint et d'entrer dans le code avec F11.

**Formule / schéma**
```csharp
private readonly IProductRepository _productRepository = new FakeProductRepository();

[Theory]
[InlineData(1)]
[InlineData(2)]
[InlineData(3)]
public void GetProductById_ReturnsProduct(int id)
{
    var product = _productRepository.GetProductById(id);
    Assert.NotNull(product);
    Assert.Equal(id, product.Id);
}
```

**Exemple donné en cours**
Les tests vus : `GetAllProducts` retourne 3 produits, l'Id 1 est trouvé, une `[Theory]` couvre les Id 1 à 3, et l'Id 129 renvoie `null`. Les tests d'Add, d'Update (existant et inexistant) et de Delete (existant et Id 99) complètent l'ensemble, pour onze tests au total.

**Piège fréquent** ⚡
Cliquer sur « exécuter » le programme au lieu de lancer les tests depuis l'explorateur de tests (« ça, c'est l'erreur des développeurs »). Oublier de rendre `Product` public : il est `internal` par défaut.

**Lien avec le reste du cours**
Ces tests devront continuer à passer avec l'implémentation ADO.NET. La CI les exécute à chaque push.

---

### 📌 FICHE 7 — Isolation des tests et couverture de tous les cas
`Thème : Tests` · `Difficulté : ●●●` · `[01:09]` · `[01:20]`

**Définition**
> Un test fiable ne dépend ni de l'ordre d'exécution ni des modifications faites par d'autres tests. Une méthode est couverte quand chacune de ses branches logiques, cas d'erreur compris, a son propre test.

**À retenir**
- Un repository partagé suffit pour les lectures. Pour un Delete ou un Add, on utilise un `new FakeProductRepository()` local au test.
- L'ordre d'exécution des tests n'est pas garanti.
- Il faut un test par cas du `switch`, y compris le `default`.
- Des tests quasi identiques se regroupent en une `[Theory]`.
- Tester le message exact d'une exception permet de détecter toute modification involontaire.

**Formule / schéma**
```csharp
[Theory]
[InlineData(ActionEnum.Add, "…message exact…")]
[InlineData(ActionEnum.Update, "…")]
[InlineData(ActionEnum.Delete, "…")]
public void ActionOnProduct_ThrowsNotImplemented(ActionEnum action, string message)
{
    var ex = Assert.Throws<NotImplementedException>(() => _productRepository.ActionOnProduct(action));
    Assert.Equal(message, ex.Message);
}

[Fact]
public void ActionOnProduct_Invalid_Throws() =>
    Assert.Throws<ArgumentOutOfRangeException>(() => _productRepository.ActionOnProduct((ActionEnum)999));
```

**Exemple donné en cours**
`ActionOnProduct` se comporte ainsi :
- GetAll appelle `GetAllProducts` ;
- Add, Update et Delete lèvent `NotImplementedException`, avec trois messages distincts ;
- le cas `default` lève `ArgumentOutOfRangeException`.

Au premier lancement, trois tests échouent parce que les messages attendus étaient faux. Une fois les messages exacts recopiés, tout passe. La suppression d'un point dans un message fait ensuite échouer un test [01:41–01:43].

**Piège fréquent** ⚡
Un test de Delete sur les données partagées peut faire échouer un test de lecture de l'Id 1 selon l'ordre d'exécution. Autre limite : vérifier qu'une méthode est appelée exactement une fois n'est pas possible avec de simples assertions, il faut des mocks.

**Lien avec le reste du cours**
Les mocks sont annoncés pour la semaine suivante. La CI rejoue ces tests de régression à chaque push.

---

### 📌 FICHE 8 — Intégration continue GitHub Actions et protection de main
`Thème : CI / GitHub` · `Difficulté : ●●○` · `[01:49]` · `[02:00]`

**Définition**
> Un workflow GitHub Actions est un fichier YAML placé dans `.github/workflows` qui compile le projet et lance les tests à chaque push. Une règle de protection de branche interdit de merger dans `main` tant que ces vérifications et d'autres conditions ne sont pas remplies.

**À retenir**
- Le workflow se déclenche automatiquement sur chaque push et sur la PR.
- Le professeur fait générer le YAML par l'IA (GitHub Copilot ⚠️, transcrit : « Guitop »). Il faut parfois plusieurs essais.
- La règle de protection impose une pull request avant le merge, une branche à jour avec `main` et la réussite du job build/test.
- Une fois la règle active, personne ne peut plus modifier `main` directement.

**Formule / schéma**
Push → GitHub Actions (`dotnet` build + test) → statut de la PR (vert ou rouge) → merge possible seulement si tout est vert, la branche à jour et la PR approuvée.

**Exemple donné en cours**
Le professeur joue le « mauvais développeur » : il retire le point d'un message d'exception, puis fait un commit « test error » et un push. La CI échoue et désigne le test en cause. Après correction et un nouveau push, elle repasse au vert, et le merge est fait [01:54–01:58].

**Piège fréquent** ⚡
Sans règle de protection, `main` reste modifiable directement, comme lors de la démonstration de [00:52]. Dans la règle, le nom du contrôle requis doit correspondre exactement au nom du job du workflow.

**Lien avec le reste du cours**
Les options de ruleset (relecture obligatoire, restrictions) seront revues la semaine prochaine. Ce cadre s'appliquera au projet évalué.

## 6. Signaux examen 🎯

| Élément | Signal donné | Horodatage |
|---|---|---|
| Passer toujours par l'interface du repository | *« je passe toujours par l'interface »* (répété aux tests) | [00:15] · [01:02] |
| Workflow commit, push, pull request, merge | *« ça c'est vraiment important la manière de travailler »* | [00:46] |
| Merger `main` dans sa branche au moins une fois par jour | *« tout bon développeur doit se dire… au minimum une fois par jour »* | [00:54] |
| Développer par itération de tests | *« C'est comme ça qu'on développe maintenant »* | [01:08] |
| Données globales et isolement des tests | *« ce qui est important dans les tests… »* | [01:09] |
| Tester tous les cas de figure d'une méthode | *« ah bah oui, c'est le but du jeu sinon tu testes pas »* | [01:21] |
| Workflow tests et CI dans tous les projets | *« c'est comme ça que vous devez travailler dans tous les projets »* | [01:55] |
| Évaluation du projet SGBD sur l'historique Git | *« montrez-moi vos repos, montrez-moi vos lignes du temps »* | [01:59] |
| Interdire la modification directe de `main` | *« ça c'est le plus important »* | [02:02] |
| Refaire toute la séance | *« il faut absolument que vous suiviez ce qu'on a fait aujourd'hui »* | [02:11] |

## 7. Informations pratiques

- **Code de la séance** : dépôt GitHub public du professeur ⚠️ (compte transcrit : « VFXL » / « VFBXL » ; dépôt : « sgbd 2026 demo repo ») [00:29].
- **À faire** : créer un compte GitHub (gratuit) et refaire toute la démonstration chez soi [00:49] · [02:11].
- **Outil conseillé** : Git Extensions [00:49].
- **Évaluation du projet SGBD** : le professeur examinera les dépôts et l'historique des commits pour juger le travail au fil du temps [01:59].
- **Vidéo du cours** : elle sera publiée sur le canal Teams utilisé pour la séance [01:58].
- **Séance d'information sur les stages** : mercredi prochain, de 17 h 30 à 18 h 30 ⚠️ (puis « de 17 h 30 à 18 h pour poser des questions »). Elle sera enregistrée, et le site des stages sera mis à jour [02:08].
- **Calendrier** [02:09] :
  - cours le 8 et le 15 octobre ;
  - pas de cours le 22 octobre (le professeur est à Vienne) ;
  - cours prévu le 29 octobre, semaine de la Toussaint : une solution de remplacement sera trouvée pour les étudiants absents.
- **Séance suivante** : implémentation ADO.NET du repository sur une petite table, pattern Factory, injection de dépendance, mocks et options de protection de branche [01:45] · [02:01] · [02:06].

## 8. Lexique

| Terme | Définition courte |
|---|---|
| ADO.NET | Technologie .NET d'accès aux bases de données relationnelles, annoncée pour la séance suivante |
| `ArgumentOutOfRangeException` | Exception levée pour une valeur d'argument hors des valeurs admises |
| `Assert` | Classe xUnit de vérification (`Equal`, `NotNull`, `Null`, `True`, `False`, `Throws<T>`) |
| Branche | Ligne de développement parallèle à `main`, créée pour une issue |
| Breakpoint | Point d'arrêt du débogueur (F9) |
| Checkout | Basculement de la copie de travail sur une branche |
| CI (intégration continue) | Compilation et tests automatiques à chaque push |
| Commit | Enregistrement local d'un ensemble de modifications dans `.git` |
| Conflit | Modifications incompatibles d'un même code, à résoudre lors d'un merge |
| Coverage | Proportion du code couverte par les tests |
| `[Fact]` | Attribut xUnit marquant un test sans paramètre |
| Fake repository | Implémentation factice du repository, avec des données en mémoire codées en dur |
| Fetch / Fetch & Prune | Récupération de l'état du serveur ; Prune supprime aussi les références aux branches disparues |
| `.git` | Dossier caché contenant le dépôt local |
| Git Extensions | Client graphique Git qui affiche le graphe des branches |
| GitHub Actions | Service d'automatisation de GitHub qui exécute des workflows YAML |
| *Gone* | État d'une branche locale dont la branche distante a été supprimée |
| `IEnumerable<T>` | Interface de séquence énumérable, parcourable par `foreach` |
| `[InlineData]` | Jeu de valeurs passé à une `[Theory]` |
| Interface | Contrat déclarant des méthodes sans les implémenter |
| `internal` | Visibilité par défaut d'une classe, limitée à son projet |
| Inversion de dépendance | Principe consistant à dépendre d'une abstraction (interface) plutôt que d'une implémentation |
| Issue | Tâche GitHub (to-do) décrivant le travail à faire |
| `main` | Branche principale, source de référence du projet |
| Merge | Fusion d'une branche dans une autre |
| Mock | Objet simulé permettant de vérifier les appels de méthodes (vu la semaine suivante) |
| `NotImplementedException` | Exception signalant une méthode non encore implémentée |
| `override` | Mot-clé C# de redéfinition d'une méthode héritée |
| Product? (type nullable) | Type de référence pouvant valoir `null`, déclaré explicitement |
| Protection de branche (ruleset) | Règles GitHub qui conditionnent le merge dans une branche |
| Pull request | Demande d'intégration d'une branche dans `main` |
| Push | Envoi des commits locaux vers le serveur |
| Référence de projet | Lien entre projets d'une solution, nécessaire pour utiliser leurs types |
| Régression | Défaut introduit dans du code qui fonctionnait, détecté en relançant les tests |
| Repository (pattern) | Couche qui encapsule l'accès aux données derrière une interface |
| `[Theory]` | Attribut xUnit marquant un test paramétré |
| `ToString` | Méthode renvoyant la représentation textuelle d'un objet |
| xUnit | Framework de tests unitaires pour .NET |
| YAML | Format de fichier de configuration des workflows GitHub Actions |

## 9. Auto-évaluation

1. **Q :** Comment redéfinit-on `ToString` en C# ?
   <details><summary>Réponse</summary>Avec <code>public override string ToString() { … }</code>. Le mot-clé <code>override</code> fait partie de la signature ; il n'y a pas d'annotation comme le <code>@Override</code> de Java.</details>

2. **Q :** Quelle différence y a-t-il entre un commit et un push ?
   <details><summary>Réponse</summary>Le commit enregistre les modifications dans le dépôt local, le dossier <code>.git</code>. Le push les envoie sur le serveur : tant qu'il n'y a pas eu de push, GitHub ne voit rien.</details>

3. **Q :** Quelles méthodes déclare `IProductRepository` ?
   <details><summary>Réponse</summary><code>GetAllProducts()</code>, <code>AddProduct(Product)</code>, <code>UpdateProduct(Product)</code>, <code>DeleteProduct(int)</code> et <code>GetProductById(int)</code>, qui retourne <code>Product?</code>.</details>

4. **Q :** Pourquoi `GetProductById` retourne-t-il `Product?` ?
   <details><summary>Réponse</summary>Parce que <code>Find</code> renvoie <code>null</code> quand aucun produit ne correspond. Le type nullable rend ce cas explicite et supprime l'avertissement ; l'appelant doit tester <code>!= null</code>.</details>

5. **Q :** Pourquoi le programme et les tests utilisent-ils une variable de type `IProductRepository` plutôt que `FakeProductRepository` ?
   <details><summary>Réponse</summary>Pour l'inversion de dépendance : on pourra remplacer l'implémentation (par exemple par ADO.NET) sans modifier le code appelant ni les tests.</details>

6. **Q :** Quelle est la différence entre `[Fact]` et `[Theory]` ?
   <details><summary>Réponse</summary><code>[Fact]</code> est un test unique sans paramètre. <code>[Theory]</code> est un test paramétré, exécuté une fois par <code>[InlineData]</code>, comme les Id 1, 2 et 3.</details>

7. **Q :** Que faut-il faire pour qu'un projet de tests puisse utiliser `FakeProductRepository` ?
   <details><summary>Réponse</summary>Ajouter une référence de projet vers le projet testé, et rendre <code>public</code> les types utilisés (<code>Product</code>, l'interface et la classe), qui sont <code>internal</code> par défaut.</details>

8. **Q :** Pourquoi un test de suppression doit-il créer son propre `new FakeProductRepository()` ?
   <details><summary>Réponse</summary>Avec un repository partagé, la suppression modifie les données des autres tests, et l'ordre d'exécution n'est pas garanti. Un test qui lit le produit 1 pourrait alors échouer selon l'ordre.</details>

9. **Q :** Comment tester complètement une méthode `ActionOnProduct(ActionEnum)` construite sur un `switch` ?
   <details><summary>Réponse</summary>Il faut un test par cas :
   <ul>
   <li>GetAll appelle bien <code>GetAllProducts</code>, ce qui demande un mock pour vérifier l'appel ;</li>
   <li>une <code>[Theory]</code> sur Add, Update et Delete vérifie avec <code>Assert.Throws&lt;NotImplementedException&gt;</code> le type de l'exception et son message exact ;</li>
   <li>un <code>[Fact]</code> avec <code>(ActionEnum)999</code> vérifie qu'une <code>ArgumentOutOfRangeException</code> est levée.</li>
   </ul></details>

10. **Q :** Décris le dispositif qui empêche du code cassé d'arriver dans `main`.
    <details><summary>Réponse</summary>Un workflow GitHub Actions en YAML, dans <code>.github/workflows</code>, compile et lance les tests à chaque push. Une règle de protection de <code>main</code> exige en plus une pull request, une branche à jour avec <code>main</code> et la réussite du job build/test avant tout merge. Toute régression, comme un point retiré d'un message, bloque donc la PR.</details>

## 10. Zones d'ombre

- Que retourne exactement `AddProduct` : le nouvel Id calculé comme `Count + 1`, ou le nombre d'éléments de la liste ? L'Id du produit est-il affecté dans la méthode ? (transcrit : « j'ai le count plus 1… pour retourner le nombre de counts ») [00:22]
- Quels sont le nom exact du compte GitHub et du dépôt de démonstration ? (transcrit : « VFXL », « VFBXL GBD2006 », « sgbd 2026 demo repo ») [00:29–00:31]
- Quel est le nom exact du projet de tests et de la classe de test ? (transcrit : « testprojectdemo », « fake product repository test ») [00:37] · [00:58]
- Pourquoi `ActionOnProduct` n'était-il pas reconnu de [01:25] à [01:30] ? Fallait-il le déclarer dans `IProductRepository`, le rendre public, ou ajouter un `using` ? La cause n'a pas été expliquée, et la transcription est confuse sur ce passage.
- Quels étaient les messages exacts des `NotImplementedException` pour Add, Update et Delete ? [01:42]
- Comment vérifier avec xUnit qu'une méthode en appelle une autre exactement une fois ? Le test est resté en commentaire. Quelle bibliothèque de mocks sera utilisée ? [01:32–01:45]
- xUnit crée-t-il une nouvelle instance de la classe de test pour chaque test, ce qui rendrait le champ `_productRepository` déjà isolé ? Ou le risque de dépendance entre tests décrit en cours s'applique-t-il bien tel quel ? [01:09–01:17]
- Quel est le nom exact du fichier YAML et du job requis dans la règle de protection ? (transcrit : « build and test yaml », « dotnet build test ») [01:58] · [02:03]
- Quelles options précises ont été activées dans la règle de protection ? (transcrit : « Adversing Brain Fortitude Rule », probablement « Add branch protection rule » ⚠️) [02:02]
- La séance d'information sur les stages finit-elle à 18 h ou à 18 h 30 ? [02:08]
- Comment sera rattrapé le cours du 29 octobre pour les étudiants absents pendant la Toussaint ? [02:09–02:10]
