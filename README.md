# Projet TV

Une TV Toshiba Android TV rapide, propre, et pilotable depuis un telephone par toute la coloc, sans compte.

## Etat

Le debloat de la TV est fait avec les outils de CleanYourTV. Le serveur, la telecommande web, l'accueil et l'agent IA restent a construire. Le plan complet est dans `docs/PLAN.md`.

## Reference

Ce projet s'appuie sur [CleanYourTV](https://github.com/Gathin23/cleanyourtv) de Gathin23, ajoute ici comme sous-module dans `vendor/cleanyourtv`. Les outils (`tools/tv.py`, `disable.sh`, `restore.sh`) et le launcher viennent de ce depot. Lire sa licence avant toute reutilisation de son code.

## Demarrer

```bash
git clone --recurse-submodules <url de ce depot>
cd <dossier>
python -m venv vendor/cleanyourtv/tools/venv
vendor/cleanyourtv/tools/venv/Scripts/python -m pip install -r vendor/cleanyourtv/tools/requirements.txt
export TV_HOST=<IP de la TV>
vendor/cleanyourtv/tools/venv/Scripts/python -I vendor/cleanyourtv/tools/tv.py --probe
```

## Regles

1. On desactive les paquets de la TV, on ne les desinstalle jamais.
2. On ne touche pas a `mediashell`, `com.google.android.tv.remote.service`, `gms`, `gsf`, `com.android.vending`, ni `com.mediatek.network`.
3. Une modification de la TV se fait avec une sauvegarde, un lot a la fois, un redemarrage de test et une mesure avant et apres.
4. Aucun secret dans Git. Les dossiers `backup/` et `work/` ne se publient jamais.

## Dossiers

| Dossier | Contenu |
| --- | --- |
| `vendor/cleanyourtv/` | CleanYourTV, en reference |
| `listes/` | Listes de debloat de cette TV |
| `docs/` | Plan, decisions, resultats des essais |
| `server/` | Pont ADB et API (a venir) |
| `web/` | Telecommande web (a venir) |
| `home/` | Composition de l'accueil (a venir) |
