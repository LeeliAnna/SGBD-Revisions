# SGBD — Révisions et fil rouge

Repo de rattrapage du cours **Programmation orientée SGBD** (C# / .NET 10 + SQL Server).
Objectif : comprendre et savoir **défendre** chaque ligne du projet SGBD à rendre en juin.

## Règles du jeu

- **Je code tout à la main.** Pas de copier-coller depuis le repo du prof ni depuis mes anciens projets.
- **Workflow Git obligatoire** (le prof évalue l'historique) : issue → branche → commits → push → pull request → merge dans `main`.
- Chaque séance = **fiche** (comprendre) + **exercices** (s'entraîner) + **étape du fil rouge** (construire).
- Après chaque séance : je fais relire mon code et je réponds aux questions « style défense ».

## Le fil rouge : `LibraryApp`

Une petite gestion de bibliothèque qui grandit avec les notions du cours :

| Étape | Notion | Ce que j'ajoute |
|---|---|---|
| 1 | Repository + tests | `Book`, `IBookRepository`, `FakeBookRepository`, tests xUnit |
| 2 | SQL Server + ADO.NET | table `Books`, `AdoNetBookRepository`, Factory, `appsettings.json` |
| 3 | SQL hors du code | fichiers `.sql` incorporés, procédure stockée, tests d'intégration (Testcontainers) |
| 4 | Architecture en couches | projets Domain / Application / Infrastructure, async |
| 5 | Web API | controllers, injection de dépendances, DTO |
| 6 | Gestion des erreurs | `ProblemDetails`, traduction des `SqlException` |
| 7 | Dapper + jointures | `Author`, relation N-N `BookAuthor`, multi-mapping |
| … | notions suivantes | Code First (EF Core), transactions… |

## Planning

**Semaine de vacances : 3 × 2h**

- [ ] Séance 1 — Fondations : repository, fake, tests xUnit, workflow Git → [`seances/seance-01`](seances/seance-01)
- [ ] Séance 2 — SQL Server, ADO.NET, Factory, `appsettings.json`
- [ ] Séance 3 — SQL incorporé, procédure stockée, Docker + Testcontainers

**Ensuite : 2h par weekend**

- [ ] Weekend 1 — Architecture en couches + async/await
- [ ] Weekend 2 — Web API ASP.NET Core + injection de dépendances
- [ ] Weekend 3 — Gestion des erreurs (ProblemDetails, SqlException)
- [ ] Weekend 4 — Dapper, jointures, base Pubs
- [ ] Puis : nouvelles notions au fil des cours + projet d'examen

## À préparer

- [ ] Docker Desktop installé (indispensable pour la séance 3)
- [ ] Transcrire les cours manquants (5 nov., 19 nov., et 29 oct. / 21 nov. s'il y en a eu)

## Sources

- **Repo du prof** : https://github.com/VFIBXL/SGBD2026_DemoRepo

  Pour l'avoir en local sur un nouveau PC (à côté de ce repo, sans le mélanger) :
  ```powershell
  cd C:\Users\admin\source\repos
  git clone https://github.com/VFIBXL/SGBD2026_DemoRepo.git
  ```
  Pour récupérer ses nouveautés après un cours :
  ```powershell
  cd C:\Users\admin\source\repos\SGBD2026_DemoRepo
  git pull
  ```
- Résumés de cours : cours du 1, 8 et 15 octobre (transcriptions)
