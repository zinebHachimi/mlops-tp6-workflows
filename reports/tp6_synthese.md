# Orchestrateur ou integration continue ? (TP06)

| Critere | Prefect (orchestrateur) | GitHub Actions (CI) |
|---|---|---|
| Ce qui declenche l'execution | Le temps : cron `0 6 * * 1` (Africa/Casablanca) ou lancement manuel | Un evenement Git : `push` ou `pull_request` |
| Ou s'execute le traitement | Sur un worker du work pool `pool-tp` (notre machine) | Sur un runner GitHub ephemere (VM propre) |
| Qui surveille, et ou | Le serveur Prefect, via l'UI `127.0.0.1:4200` (flow runs, task runs, logs) | GitHub, dans l'onglet Actions (OK/echec par commit) |
| Ce qui se passe en cas d'echec | Run `Failed` (`F1 0.25 < seuil 0.82`) : `valider` bloque, pas de notification ; relance possible | Job en echec : commit ou PR en rouge, fusion bloquee ; correction puis nouveau `push` |
| Type de tache adapte | Taches recurrentes, longues, avec donnees : reentrainement, `dvc repro`, validation du F1 | Taches courtes et sans etat : installation, `verifier_qualite.py`, tests |
| Ce qui est versionne dans Git | `flow_mlops.py`, `planifier.py`, `prefect.yaml` (l'historique des runs reste dans Prefect) | `.github/workflows/ci.yml`, `requirements-ci.txt`, `verifier_qualite.py` |

## Conclusion

1. Prefect repond a « quand relancer ? », GitHub Actions a « ce code est-il sur ? » : ils ne se concurrencent pas.
2. Prefect orchestre le cycle de vie du modele : reentrainement planifie, validation du F1 >= 0.82, notification.
3. GitHub Actions protege le depot : a chaque `push`, il rejoue `verifier_qualite.py` et bloque les regressions.
4. Meme regle de qualite (F1 >= 0.82), appliquee a deux moments : en continu (Prefect) et a chaque modification du code (CI).
5. En MLOps on les combine : CI pour le code, Prefect pour l'execution, DVC + Git pour la reproductibilite.