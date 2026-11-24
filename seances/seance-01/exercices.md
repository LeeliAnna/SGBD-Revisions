# Séance 1 — Exercices

> Ordre conseillé : A → B → C. Ouvre les indices **seulement si tu bloques plus de 5 minutes**.
> Pas de corrigé complet : tu me montres ton code et on le relit ensemble.

---

## A. Trouver les bugs (10 min)

Ce repository contient **5 problèmes**. Trouve-les et explique pourquoi chacun pose problème.

```csharp
class Book
{
    public int id { get; set; }
    public string Title { get; set; }
}

public class FakeBookRepository : IBookRepository
{
    public List<Book> _books = new() { new Book { id = 1, Title = "Dune" } };

    public Book GetBookById(int id)
    {
        return _books.Find(b => b.id == id);
    }

    public bool DeleteBook(int id)
    {
        _books.RemoveAll(b => b.id == id);
        return true;
    }
}
```

<details><summary>Indice 1</summary>Regarde la visibilité des types et des champs.</details>
<details><summary>Indice 2</summary>Regarde les conventions de nommage, la valeur par défaut des chaînes et ce que renvoie <code>Find</code>.</details>
<details><summary>Indice 3</summary>Que renvoie <code>DeleteBook(42)</code> si le livre 42 n'existe pas ?</details>

---

## B. Écrire les cas de test sur papier (15 min)

Sans coder, liste **tous** les tests nécessaires pour couvrir ces méthodes. Pour chacun, note le nom du test, les données et le résultat attendu.

- `GetAllBooks()`
- `GetBookById(int id)`
- `AddBook(Book book)`
- `UpdateBook(Book book)`
- `DeleteBook(int id)`
- `SearchByTitle(string text)` : renvoie les livres dont le titre **contient** le texte, **sans tenir compte de la casse**

Ensuite, entoure les tests qui **modifient** les données. Ceux-là auront besoin de leur propre instance de repository.

<details><summary>Indice : nommage des tests</summary>Convention pratique : <code>Méthode_Situation_RésultatAttendu</code>, par exemple <code>GetBookById_UnknownId_ReturnsNull</code>.</details>
<details><summary>Indice : combien de tests ?</summary>Vise au moins 12 tests. <code>SearchByTitle</code> à lui seul en mérite 4 : texte trouvé, casse différente, aucun résultat, texte vide.</details>

---

## C. Fil rouge `LibraryApp` — étape 1 (60 min)

### C0. Mise en place Git (10 min)

1. Sur GitHub : crée le repo `SGBD-Revisions` (privé, **sans** README), puis relie-le à ce dossier :
   ```powershell
   git remote add origin https://github.com/<ton-pseudo>/SGBD-Revisions.git
   git push -u origin main
   ```
2. Crée l'issue **« Étape 1 : Book, repository fake et tests »**, avec une ligne de contexte et une ligne d'objectif.
3. Crée la branche depuis l'issue (*Development → Create a branch*), puis `git fetch` et `git checkout` sur cette branche.

### C1. La solution (5 min)

Dans un dossier `src/` :
- une solution `LibraryApp` ;
- un projet console `LibraryApp` (.NET 10) ;
- un projet *xUnit Test Project* `LibraryApp.Tests` qui référence `LibraryApp`.

👉 Fais un premier commit dès que la solution compile.

### C2. Le modèle `Book` (5 min)

Dans `Models/Book.cs` :

| Propriété | Type | Remarque |
|---|---|---|
| `Id` | `int` | |
| `Title` | `string` | jamais `null` |
| `Isbn` | `string` | jamais `null`. On le rendra unique en base à la séance 2 |
| `PublishedYear` | `int` | |

Redéfinis `ToString()` pour afficher les quatre champs.

### C3. L'interface et le fake (15 min)

- `Repositories/IBookRepository.cs` : les 5 méthodes CRUD (même logique que `IProductRepository`), plus `SearchByTitle`.
- `Repositories/FakeBookRepository.cs` : une liste privée de **3 livres** codés en dur.
- `AddBook` renvoie le nouvel Id. **C'est le repository qui le choisit**, pas l'appelant.

<details><summary>Indice : calculer le nouvel Id</summary>Le plus grand Id existant + 1. Pense au cas d'une liste vide : regarde ce que fait <code>Max()</code> sur une liste vide, et ce que permet <code>DefaultIfEmpty()</code>.</details>
<details><summary>Indice : recherche sans casse</summary><code>string.Contains</code> accepte un deuxième paramètre de type <code>StringComparison</code>.</details>

👉 Commit.

### C4. Les tests (20 min)

Code **tous** les tests de ta liste de l'exercice B. Utilise au moins une `[Theory]`.

👉 Commit et push. Tout doit être vert dans l'Explorateur de tests.

### C5. Le programme (5 min)

Dans `Program.cs`, déclare `IBookRepository repo = new FakeBookRepository();`, puis fais le scénario suivant en affichant la liste après chaque étape : ajout, recherche par Id, modification, recherche par titre, suppression.

### C6. Clôture

1. Ouvre la **pull request** et écris `Closes #1` dans sa description.
2. Merge dans `main`, puis supprime la branche.
3. Reviens sur `main` en local et fais un `git pull`.

---

## Défi facultatif : la CI

Ajoute `.github/workflows/dotnet.yml`, qui fait `dotnet build` puis `dotnet test` sur `src/LibraryApp.slnx` (ou `.sln`) à chaque push et à chaque PR. Ensuite :
1. Casse volontairement un test et pousse : la PR doit passer au rouge.
2. Répare : elle repasse au vert.
3. Active la protection de `main` : PR obligatoire et CI verte requise.

---

## ✅ Checklist de fin de séance

- [ ] Je sais expliquer chaque ligne de `FakeBookRepository`
- [ ] Au moins 12 tests, tous verts
- [ ] Au moins 4 commits avec des messages clairs
- [ ] Une issue, une branche, une PR mergée
- [ ] Questions de défense de `fiche.md` : je réponds sans regarder
- [ ] J'ai demandé une relecture à Claude
