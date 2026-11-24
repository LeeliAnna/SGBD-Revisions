# Séance 1 — Fondations : Repository, tests xUnit, workflow Git

> Source : cours du 1er octobre. Temps de lecture : ~20 min.

## Déroulé des 2h

| Durée | Quoi |
|---|---|
| 20 min | Lire cette fiche, répondre de tête aux questions de défense |
| 25 min | Exercices A et B (`exercices.md`) |
| 60 min | Exercice C : fil rouge `LibraryApp`, étape 1 |
| 15 min | PR, merge, puis relecture avec Claude |

---

## 1. Le modèle

Un **modèle** est une classe simple qui représente une donnée métier. On le range dans un dossier `Models`, avec un nom en anglais.

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;   // jamais null
    public decimal Price { get; set; }                 // decimal pour l'argent

    public override string ToString() => $"{Id} - {Name} - {Price}";
}
```

- Les propriétés sont en **PascalCase**. Le snippet `prop` + Tab les génère.
- `override` est un **mot-clé** de la signature. Ce n'est pas une annotation `@Override` comme en Java.
- Une classe est `internal` par défaut. Elle doit être `public` pour qu'un autre projet, par exemple le projet de tests, puisse l'utiliser.

## 2. Le pattern Repository

Le **repository** est la couche qui donne accès aux données. Son contrat est une **interface** :

```csharp
public interface IProductRepository
{
    IEnumerable<Product> GetAllProducts();
    Product? GetProductById(int id);   // ? : peut renvoyer null
    int AddProduct(Product product);
    bool UpdateProduct(Product product);
    bool DeleteProduct(int id);
}
```

Le programme ne connaît **que l'interface** :

```csharp
IProductRepository repo = new FakeProductRepository();
```

C'est l'**inversion de dépendance**. Demain, on remplace `FakeProductRepository` par `AdoNetProductRepository` (séance 2), et ni le programme ni les tests ne changent.

> 🎯 Le prof l'a répété : *« je passe toujours par l'interface »*.

### Le fake repository

C'est une implémentation **factice** sur une `List<T>` privée, avec des données codées en dur. Elle sert à développer et à tester sans base de données.

| Méthode | Idée | Valeur de retour |
|---|---|---|
| `GetAll` | renvoie la liste | `IEnumerable<T>` |
| `GetById` | `_items.Find(x => x.Id == id)` | l'objet ou `null` |
| `Delete` | `_items.RemoveAll(x => x.Id == id)` | `true` si plus de 0 élément a été supprimé |
| `Update` | cherche, `false` si absent, sinon remplace | `bool` |
| `Add` | ajoute, renvoie le nouvel Id | `int` |

⚡ **Piège :** ignorer l'avertissement « possible null ». Il faut déclarer `T?` dans l'interface **et** dans la classe, puis tester `!= null` chez l'appelant.

## 3. Les tests unitaires xUnit

- Le projet de tests est un **projet séparé** de type *xUnit Test Project*. Il a une **référence de projet** vers le code testé.
- `[Fact]` marque un test sans paramètre.
- `[Theory]` avec plusieurs `[InlineData(...)]` exécute le même test pour chaque jeu de valeurs.
- Assertions principales : `Assert.Equal(attendu, obtenu)`, `Assert.Null`, `Assert.NotNull`, `Assert.True`, `Assert.False`, `Assert.Throws<TException>(() => ...)`.
- On les lance depuis l'**Explorateur de tests**, pas avec F5. En mode Debug, le test s'arrête sur les breakpoints.

```csharp
[Theory]
[InlineData(1)]
[InlineData(2)]
public void GetProductById_Existing_ReturnsProduct(int id)
{
    IProductRepository repo = new FakeProductRepository();
    var p = repo.GetProductById(id);
    Assert.NotNull(p);
    Assert.Equal(id, p.Id);
}
```

### Isolation : la règle d'or

L'**ordre d'exécution des tests n'est pas garanti**. Si un test supprime un élément dans un repository partagé, un autre test qui lit cet élément peut échouer au hasard.
→ Un test qui **modifie** les données crée son propre `new FakeXxxRepository()`.

### Tester tous les cas

Il faut **un test par branche logique** : le cas qui marche, l'Id inexistant, le `default` d'un `switch`, les exceptions. On teste même le **message** exact d'une exception : supprimer un point dans le message doit faire échouer le test (c'est un test de **régression**).

```csharp
var ex = Assert.Throws<ArgumentOutOfRangeException>(() => repo.ActionOnProduct((ActionEnum)999));
```

## 4. Le workflow Git / GitHub

```
Issue → branche (créée depuis l'issue) → fetch + checkout
     → commits (locaux, dans .git) → push (vers GitHub)
     → pull request → CI verte → merge dans main → supprimer la branche
```

- Un **commit** reste local. Sans **push**, GitHub ne voit rien.
- On ne travaille **jamais** directement sur `main`.
- On merge `main` dans sa branche **au moins une fois par jour**.
- **CI** : un fichier YAML dans `.github/workflows` lance `dotnet build` et `dotnet test` à chaque push.
- **Protection de branche** : on interdit le push direct sur `main` et on exige une PR, une branche à jour et une CI verte.

> 🎯 *« montrez-moi vos repos, montrez-moi vos lignes du temps »* : l'historique Git compte dans l'évaluation.

---

## Questions de défense (réponds à voix haute, sans regarder)

1. Pourquoi déclarer `IProductRepository repo` au lieu de `FakeProductRepository repo` ?
2. Que se passe-t-il si `GetProductById` reçoit un Id qui n'existe pas ? Comment le code appelant le gère-t-il ?
3. Pourquoi un test qui fait `Delete` ne doit-il pas utiliser le repository partagé de la classe de test ?
4. Quelle différence entre `[Fact]` et `[Theory]` ?
5. Ton collègue a committé, mais tu ne vois pas son code sur GitHub. Pourquoi ?
6. À quoi sert la CI si tu lances déjà les tests sur ton PC ?
7. Pourquoi vérifier le **message** d'une exception, et pas seulement son type ?
8. Qu'est-ce qui va changer, et qu'est-ce qui ne va **pas** changer, quand on passera à ADO.NET ?

<details><summary>Éléments de réponse (à lire seulement après avoir essayé)</summary>

1. Pour dépendre du contrat et non de l'implémentation. On change d'implémentation (fake → ADO.NET) en modifiant une seule ligne.
2. `Find` renvoie `null`, donc le type de retour est `Product?`. L'appelant teste `!= null` avant d'utiliser l'objet.
3. L'ordre des tests n'est pas garanti : la suppression casserait les tests qui lisent cet élément.
4. `[Fact]` : un seul cas, sans paramètre. `[Theory]` : le même test rejoué pour chaque `[InlineData]`.
5. Le commit est local. Il faut un `push`.
6. Elle teste sur une machine neutre, à chaque push, pour tout le monde, et elle bloque le merge si quelque chose casse.
7. Pour détecter toute modification involontaire du comportement : c'est un test de régression.
8. Ce qui change : la classe concrète et la ligne d'instanciation. Ce qui ne change pas : l'interface, le programme et les tests.
</details>
