# CLAUDE.md — SGBD-Revisions

Repo de rattrapage du cours **Programmation orientée SGBD** (EPHEC, 2026-2027) : C# / .NET 10 + SQL Server.
Ce fichier sert de mémoire partagée entre les sessions Claude, sur tous mes PC. **Le tenir à jour** à chaque évolution importante.

## Contexte

- J'ai pris du retard et je reprends tout depuis le début.
- **Examen** : un projet SGBD à rendre en **juin 2027**, à **défendre** à l'oral. Je dois comprendre et savoir expliquer chaque ligne.
- Le prof évalue aussi **l'historique Git** (issues, branches, PR) : *« montrez-moi vos lignes du temps »*.
- **Temps disponible** : environ 2h par semaine pour ce cours (2h par jour le weekend, partagées avec le cours Web). Pendant les vacances, 6h sur la semaine.
- Le cours Web a son propre repo, https://github.com/LeeliAnna/WEB-Revisions (local : `C:\Users\admin\source\repos\WEB-Revisions`) : ne pas mélanger les deux.
- Ce repo-ci : https://github.com/LeeliAnna/SGBD-Revisions

## Règles pédagogiques (importantes)

- **Je code à la main** pour prendre les réflexes. Claude n'écrit **pas** le code des exercices ni du fil rouge à ma place.
- Claude explique, propose des exercices avec des **indices progressifs** (dans des `<details>`), **relit** mon code (pointer le problème et expliquer pourquoi, sans tout réécrire) et pose des **questions de défense**.
- Pas de corrigé complet dans les fiches : la correction se fait en relisant mon code.
- Réponses en français.

## Organisation des repos

- **Ce repo-ci = le cours** : `README.md` (planning), `seances/seance-NN/fiche.md` et `exercices.md`, `sources/` (résumés de cours).
- **Mon code** (exercices + fil rouge `LibraryApp`) est dans un **repo séparé** (lien dans le README, section « Mon code »).
- Repo du prof : https://github.com/VFIBXL/SGBD2026_DemoRepo (cloné localement dans `C:\Users\admin\source\repos\SGBD2026_DemoRepo`). `prof-repo/` est une copie locale ignorée par Git.
- Ancien travail : `C:\Users\admin\source\repos\Ephec_SGBD` contient déjà le devoir `Manager` (fake, ADO.NET, Testcontainers). C'est pour ça que le fil rouge porte sur un autre domaine : la bibliothèque (`Book`, puis `Author` en N-N).

## Sources

- `sources/AAAA-MM-JJ-sujet.md` : résumés de cours générés depuis mes transcriptions (outil « Transcriptor »). Format : en-tête, résumé, plan, points clés, fiches, signaux examen 🎯, infos pratiques, lexique, quiz, zones d'ombre.
- Cours transcrits : 1, 8 et 15 octobre.
- **Cours manquants** (à transcrire) : ceux des commits du prof du 5, 19 et 21 novembre (architecture en couches, Web API, gestion des erreurs, Dapper/Pubs), et peut-être le 29 octobre.

## Quand je donne un nouveau transcript

1. Lire le résumé `.md` (et `transcription.txt` en cas de doute), puis le **copier** dans `sources/` sous la forme `AAAA-MM-JJ-sujet.md`.
2. Croiser avec le repo du prof (`git pull` puis `git log`) pour retrouver le code de la séance.
3. Mettre à jour la carte des notions et le planning du `README.md`. Si nécessaire, réécrire les fiches déjà prévues qui s'appuyaient seulement sur le code du prof.
4. Créer ou ajuster `seances/seance-NN/fiche.md` et `exercices.md`, sur le modèle de la séance 1 (déroulé de 2h, fiche, questions de défense, exercices A/B/C avec indices, étape du fil rouge, checklist).
5. Signaler les 🎯 signaux examen du prof dans la fiche.
6. Committer (message en français), puis demander avant de pousser.

## Environnement (PC principal)

- .NET SDK 9 et 10, instance SQL Server `EPHEC_SGBD`, Git. Pas de `gh` CLI.
- Docker Desktop **à installer** avant la séance 3 (Testcontainers).

## Avancement

- [x] Séance 1 préparée (fiche + exercices)
- [ ] Séance 1 faite
- [ ] Séance 2 à préparer : SQL Server, ADO.NET, Factory, `appsettings.json`
- [ ] Séance 3 à préparer : SQL incorporé, procédure stockée, Testcontainers
