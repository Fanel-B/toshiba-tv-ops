# Projet TV : plan de travail

1er novembre 2026, Fanel

## Sommaire

Le projet est arrêté sur un blocage : le Wi-Fi de la TV ne fonctionne plus depuis le redémarrage du 27 octobre 2026. Tout le reste attend cette réparation (tranche 1). Les tranches sont ordonnées par rapport valeur sur effort : ce qui rend le projet défendable passe avant ce qui se voit.

| N° | Objet | Durée estimée | État |
| --- | --- | --- | --- |
| 1 | Réparer le Wi-Fi de la TV | 2 à 4 h | À faire, bloquant |
| 2 | Créer le dépôt, figer l'état, proposer la liste BODRUM | 3 à 4 h | À faire |
| 3 | Rendre l'accès à la TV fiable (IP fixe, veille, réveil) | 3 h | À faire |
| 4 | Partage d'écran natif depuis le S24 FE | 1 à 2 h | À faire |
| 5 | Pont ADB local et verrou une personne à la fois | 1 à 2 jours | À faire |
| 6 | Télécommande web sur le téléphone | 1 à 2 jours | À faire |
| 7 | Accueil composé côté serveur | 1 à 2 jours | À faire |
| 8 | Agent IA qui pilote la TV, en texte | 2 jours | À faire |
| 9 | Voix depuis le téléphone | 1 à 2 jours | À faire |

Si tu n'as que cinq minutes, lis ceci : le câble Ethernet est branché, lance le diagnostic de la tranche 1, et ne construis rien d'autre tant que le Wi-Fi n'est pas réparé et que la cause n'est pas connue.

## En deux minutes

Le projet transforme une TV Toshiba Android TV lente et pleine de pubs en TV rapide, propre et pilotable depuis un téléphone par toute la coloc, sans compte. Aujourd'hui, seule la première moitié existe : la TV est nettoyée, et son Wi-Fi est cassé.

La TV est une Toshiba 50UA2363DG, fabriquée par Vestel : carte BODRUM, Android TV 11, 1,4 Go de RAM, 131 paquets. Le débloat reprend la méthode du dépôt open source CleanYourTV : on désactive des paquets avec `pm disable-user`, on n'en supprime aucun, et un script de restauration annule tout.

**Ce qui marche**

- Connexion ADB à la TV et inventaire des 131 paquets
- 15 paquets désactivés, en trois lots, avec fichiers de restauration
- Nouveau launcher `com.gathin.tvhome` installé et défini comme Home (démarrage à froid 2,3 s)
- Sauvegardes avant modification (getprop, listes de paquets, réglages)
- Liste BODRUM écrite au format du dépôt CleanYourTV

**Ce qui n'existe pas encore**

- Un Wi-Fi qui fonctionne sur la TV
- Un dépôt Git à toi (le travail est dans un clone du dépôt de Gathin23)
- La mesure finale de RAM et de swap après le lot 5
- Le serveur, la télécommande web, l'accueil photos et météo
- L'agent IA, la voix, le partage d'écran testé

Comment lire ce document : les sections 3 à 5 servent à reprendre le travail, la section 6 donne les tâches, les sections 8 et 10 listent ce qui est cassé ou encore à décider.

Limite de ce document : je n'ai pas accès à ton PC ni à un dépôt. Tout ce qui est écrit comme fait vient des comptes rendus des sessions Claude Code du 27 octobre, collés dans la conversation. Rien n'a été relancé pour le rédiger.

## Démarrer en 15 minutes

Il faut un PC Windows, Git, Python 3, un JDK 21, Android Studio avec la platform 34 et les build-tools 34.0.0, et la TV branchée en Ethernet avec le débogage USB et réseau activés. Les commandes ci-dessous sont celles utilisées dans les sessions, sous Git Bash.

```bash
git clone https://github.com/Gathin23/cleanyourtv
cd cleanyourtv
python -m venv tools/venv
tools/venv/Scripts/python -m pip install -r tools/requirements.txt
export TV_HOST=<IP de la TV>
tools/venv/Scripts/python -I tools/tv.py --probe
```

Si `--probe` renvoie le modèle, la version d'Android et le firmware, la liaison marche. Au premier contact, accepter « Autoriser le débogage USB ? » sur la TV (toujours autoriser depuis cet ordinateur).

Pour reconstruire et réinstaller le launcher sans toucher au launcher d'origine :

```bash
export JAVA_HOME=<chemin du JDK 21>
export ANDROID_HOME="$LOCALAPPDATA/Android/Sdk"
STOCK_LAUNCHER=none TV_HOST=<IP de la TV> launcher/build-and-install.sh
```

Pour tout annuler, dans cet ordre (cette commande n'a encore jamais été exécutée) :

```bash
TV_HOST=<IP de la TV> tools/restore.sh work/lot5.txt
TV_HOST=<IP de la TV> tools/restore.sh work/lot3.txt
TV_HOST=<IP de la TV> tools/restore.sh work/lot1.txt
```

## Carte du dépôt

Le dépôt à toi n'existe pas encore. Voici ce qu'on trouve aujourd'hui dans le clone, et ce que la tranche 2 crée.

| Dossier ou fichier | Contenu | Existe aujourd'hui |
| --- | --- | --- |
| `tools/` | `tv.py` (client ADB en Python), `disable.sh`, `restore.sh`, `privacy.py`, `check.py` | Oui, dans le clone de Gathin23 |
| `launcher/` | Sources du launcher, `build-and-install.sh` | Oui, dans le clone |
| `examples/` | Une liste de débloat testée par modèle de TV | Oui, plus `toshiba-50ua2363dg-bodrum/` écrit par nous |
| `work/` | Notes d'inventaire, `tableau-131.md`, `lot1.txt`, `lot3.txt`, `lot5.txt` | Oui, local |
| `backup/` | getprop, 5 listes de paquets, 3 espaces de réglages | Oui, local, ne jamais publier |
| `CONTEXT-TOSHIBA.md`, `RAPPORT.md` | Contraintes du projet, rapport de session | Oui, `RAPPORT.md` non vérifié |
| `server/` | Pont ADB et API de la télécommande | Non, tranche 5 |
| `web/` | Télécommande web (PWA) | Non, tranche 6 |
| `home/` | Composition de l'image d'accueil | Non, tranche 7 |
| `docs/` | Ce plan, décisions, schémas | Non, tranche 2 |

## Les trois règles avant de toucher au code

1. On désactive, on ne désinstalle jamais. Un paquet désinstallé ne revient pas, un paquet désactivé revient avec `restore.sh`.
2. On ne touche jamais à `com.google.android.apps.mediashell` (Chromecast), `com.google.android.tv.remote.service` (Google TV Remote), `gms`, `gsf`, `com.android.vending`, ni à `com.mediatek.network`. Le reste du projet en dépend.
3. Une modification de la TV se fait avec une sauvegarde, un lot à la fois, un redémarrage de test et une mesure avant et après. Le Wi-Fi s'est cassé après un redémarrage, pas pendant l'application d'un lot : un test sans redémarrage ne suffit donc pas.

## Comment on travaille ensemble

Une tranche, une branche, une pull request, et une seule personne à la fois sur la TV. Ces règles sont proposées : le dépôt n'existe pas, rien n'est encore appliqué.

| Sujet | Règle |
| --- | --- |
| Branches | `main` toujours fonctionnelle. Une branche par tranche : `tranche-3-acces-tv`, `tranche-5-pont-adb`. |
| Pull requests | Une PR par tranche, avec la case « critère de fin » cochée et la preuve (capture, sortie de commande). |
| Relecture | L'autre personne relit avant fusion. Seul au départ : relire soi-même le lendemain, jamais le soir même. |
| Journal | `CHANGELOG.md` à la racine, une ligne par modification de la TV : date, paquet ou fichier, lot, mesure avant et après. |
| Se marcher dessus | Une personne par tranche. La TV est une ressource partagée : personne ne désactive un paquet ou ne redémarre la TV sans l'annoncer. |
| Liste de paquets | Aucun paquet n'est ajouté à une liste de débloat sans PR et sans ligne « ce qu'on perd ». |
| Secrets | Clé API Claude dans `.env`, jamais dans Git. `.gitignore` obligatoire pour `.env`, `backup/`, `work/` et les clés ADB. |

**Points de vigilance sur les secrets.** Les clés ADB et les fichiers `recovery-*.json` sont stockés hors du dépôt, dans `C:\Users\bfane\.cleanyourtv\`. `backup/` contient les réglages de la TV et peut contenir des identifiants de compte et de réseau : il ne se publie pas, même dans un dépôt privé, avant relecture ligne par ligne.

## Ce qui est déjà fait

Le débloat et le launcher ont été vérifiés étape par étape le 27 octobre 2026. Le redémarrage qui a suivi le lot 5 a cassé le Wi-Fi, et la mesure finale n'a jamais été relevée. Source de tout ce tableau : les comptes rendus des sessions Claude Code, pas une vérification refaite pour ce document.

| Vérification | Résultat | Date |
| --- | --- | --- |
| Connexion ADB et `--probe` | OK : Toshiba/BODRUM, device finch, MediaTek m7332, Android 11, SDK 30, 32 bits, `low_ram=true` | 27/10/2026 |
| Inventaire des paquets | 131 paquets (130 activés, 1 déjà désactivé : talkback). Le premier chiffre de 130 était faux | 27/10/2026 |
| `dumpsys` après redémarrage | Hors service avant (`hung system process`), rétabli après | 27/10/2026 |
| Test Wi-Fi cname (`setupwraith`, `com.mediatek.network`) | Les 108 réseaux enregistrés appartiennent à `android.uid.system:1000`. `setupwraith` a l'UID 10049, donc n'est pas le propriétaire | 27/10/2026 |
| Lot 1 : 5 paquets (katniss, tvrecommendations, vestelreporter, partnercustomizer, fusion.android.tv.demo) | 5 sur 5 désactivés, 74,7 Mo libérés (estimation initiale : 117 Mo), aucun FATAL ni ANR | 27/10/2026 |
| Lot 3 : 9 paquets prévus | 8 sur 9 désactivés. `android.autoinstalls.config.vestel.malin` refusé par le garde-fou, laissé actif volontairement | 27/10/2026 |
| Launcher `com.gathin.tvhome` | Build avec Gradle 8.9 sur JDK 21, APK de 538 581 octets, installation `Status: ok`, démarrage à froid 2,3 s, navigation à la télécommande vérifiée par captures | 27/10/2026 |
| Lot 5 : launcher d'origine et `backdrop` | Désactivés après confirmation du nouveau Home, qui résout vers `com.gathin.tvhome` | 27/10/2026 |
| Mesure de RAM | Avant lot 1 (14 min après redémarrage) : MemAvailable 570,7 Mo. Après lot 3 : 17 Mo sous cette base, car le launcher d'origine a grossi de 158 à 190 Mo | 27/10/2026 |
| Wi-Fi après le redémarrage du lot 5 | Échec : TV absente du réseau (aucune entrée ARP), reconnexion impossible, YouTube répond « réseau introuvable », la TV affiche « indisponible » | 27/10/2026, vers 05 h |
| Câble Ethernet branché | Fait, IP de la TV en Ethernet non notée | 28/10/2026 |

**Ce qui n'a jamais été testé :**

- La mesure finale de RAM et de swap après le lot 5 (la session attendait 860 s d'uptime quand la TV est devenue injoignable).
- `restore.sh`, sur aucun des trois lots.
- Chromecast depuis le Samsung S24 FE.
- Le service Google TV Remote (protocole Remote v2).
- Le comportement de la TV en veille. La note « la TV coupe le Wi-Fi en veille » vient du dépôt, pour une carte voisine (Cosmos). Elle n'a pas été vérifiée sur ta carte, et l'hypothèse de la veille n'explique pas le Wi-Fi cassé, puisque tu as dit que la TV n'était pas en veille.
- Tout test automatisé : aucun n'existe pour ta partie du projet.

## Les tranches de travail

Neuf tranches, chacune de quelques heures à deux jours. Dépendances : la tranche 1 est prérequis des tranches 3 et 4 (sans réseau, on ne peut ni mesurer ni caster). La 3 est prérequis de la 5 (le pont ADB a besoin d'une adresse qui ne bouge pas). La 5 est prérequis des 6, 7 et 8 (tout passe par le pont). La 8 est prérequis de la 9. La tranche 2 peut se faire en parallèle de la 1, sauf la pull request vers CleanYourTV.

### Tranche 1 : réparer le Wi-Fi de la TV

Objectif : retrouver une TV qui se connecte au Wi-Fi, et savoir quelle modification l'a cassé. Durée : 2 à 4 h. Prérequis : câble Ethernet branché.

Diagnostic, sans rien modifier :

- Noter l'IP de la TV en Ethernet (Paramètres, Réseau et Internet).
- Lancer `tv.py --probe` sur cette IP.
- Lire `dumpsys wifi` : interface wlan0 haute ou basse, état du supplicant, réseaux enregistrés présents ou effacés.
- Lire logcat filtré sur wifi et supplicant pendant une tentative de connexion.
- Comparer les réglages global et secure actuels avec ceux de `backup/`.
- Vérifier que les paquets désactivés sont bien ceux des lots 1, 3 et 5, plus talkback.

Recherche de la cause par bissection (un redémarrage et un test du Wi-Fi après chaque étape) :

- Restaurer le lot 5 (launcher d'origine et `backdrop`), redémarrer, tester.
- Si le Wi-Fi ne revient pas : restaurer le lot 3, redémarrer, tester.
- Si le Wi-Fi ne revient pas : restaurer le lot 1, redémarrer, tester.
- Si rien ne revient avec tout restauré : la cause n'est pas un paquet. Essayer la réinitialisation réseau de la TV, puis le reset usine en dernier recours.

Fermeture :

- Réappliquer les lots sauf le coupable, un lot à la fois, redémarrage et test du Wi-Fi après chacun.
- Écrire la cause en une ligne dans `CHANGELOG.md` et `RAPPORT.md`.

**Critère de fin :** la TV retrouve le Wi-Fi et Internet après trois redémarrages de suite avec la liste de paquets finale, et la cause de la panne est écrite.

### Tranche 2 : créer le dépôt et figer l'état

Objectif : ne plus risquer de perdre le travail, et avoir quelque chose à montrer. Durée : 3 à 4 h. Prérequis : aucun pour le dépôt. La pull request vers CleanYourTV attend la fin de la tranche 1, car une liste qui casse le Wi-Fi ne doit pas être proposée à d'autres.

Dépôt :

- Choisir le nom (question ouverte).
- Créer le dépôt GitHub, privé au départ.
- `.gitignore` pour `.env`, `backup/`, `work/` et les clés.
- README avec la promesse, des captures avant et après, et la section « démarrer en 15 minutes ».
- Ajouter ce plan dans `docs/PLAN.md`.
- Workflow GitHub Actions qui échoue si `.env` ou `backup/` est présent dans un commit.

Rapport et mesure :

- Relever la mesure finale 860 s après redémarrage : MemAvailable, MemFree, swap utilisé, mémoire du launcher.
- Terminer `RAPPORT.md` : tableau avant et après, cause du Wi-Fi, paquets épargnés et pourquoi.

Contribution à CleanYourTV (après la tranche 1) :

- Lire `LICENSE` avant de copier du code du dépôt dans le tien.
- Forker `Gathin23/cleanyourtv`, ouvrir une pull request pour `examples/toshiba-50ua2363dg-bodrum/` (liste et `tv.json`) avec l'en-tête demandé par `CONTRIBUTING.md`.

**Critère de fin :** une personne qui arrive clone le dépôt, suit le README et obtient un `--probe` sans te poser de question, et la CI refuse un commit qui contient `backup/`.

### Tranche 3 : rendre l'accès à la TV fiable

Objectif : que le serveur retrouve la TV à la même adresse, et savoir dans quels états elle reste joignable. Durée : 3 h. Prérequis : tranche 1.

Adresse :

- Réserver un bail DHCP fixe dans la box pour l'Ethernet et pour le Wi-Fi (deux adresses MAC).
- Vérifier que le port 5555 reste ouvert après un redémarrage de la TV. Sinon, appliquer la méthode de `docs/TROUBLESHOOTING.md` du dépôt.

Veille et réveil :

- Mettre la TV en veille, tester ping et le port 5555 après 5 minutes puis après 30 minutes.
- Tester le réveil depuis le téléphone (Chromecast), et par Wake-on-LAN si la TV est en Ethernet.
- Remplir le tableau de résultats dans `docs/ACCES-TV.md`.

**Critère de fin :** le tableau « état de la TV (allumée, veille 5 min, veille 30 min) contre (ping, ADB, réveil possible) » est rempli avec des essais réels, et l'adresse de la TV est identique après un redémarrage de la TV et un redémarrage de la box.

### Tranche 4 : partage d'écran natif

Objectif : montrer l'écran du téléphone sur la TV sans écrire une ligne de code. Durée : 1 à 2 h. Prérequis : tranche 1, et Chromecast intact (`com.google.android.apps.mediashell` activé).

- Vérifier que `mediashell` ne figure pas dans la liste des paquets désactivés.
- Depuis le S24 FE, lancer un cast depuis YouTube.
- Depuis le S24 FE, tester la duplication d'écran (Smart View).
- Noter ce qui marche, la latence ressentie et les blocages éventuels (audio, applications protégées).

**Critère de fin :** un colocataire envoie une vidéo de son propre téléphone sur la TV sans ton aide, et le résultat est écrit dans `docs/ACCES-TV.md`.

### Tranche 5 : pont ADB et verrou une personne à la fois

Objectif : un seul programme du réseau local parle à la TV, et toutes les tranches suivantes passent par lui. Durée : 1 à 2 jours. Prérequis : tranche 3, et la décision sur l'endroit où tourne le serveur (question ouverte).

Serveur :

- Choisir le langage (Node ou Python, question ouverte) et la façon d'appeler ADB : `tools/tv.py` en sous-processus ou un client ADB natif.
- Exposer `POST /key`, `POST /app`, `GET /screenshot`, `GET /status`.
- Liste blanche des touches et des applications. Aucune commande shell libre.
- Écouter sur le réseau local seulement, jamais sur Internet.

Verrou :

- Une seule session active. Prise du verrou, libération manuelle, expiration après 5 minutes d'inactivité.
- Refus du second client avec le nom du détenteur.
- Concevoir le verrou comme un salon de taille 1 (identifiant de salon, liste de joueurs), pour que la version multi-joueurs n'oblige pas à tout refaire.

Qualité :

- Mode fausse TV pour tester sans la vraie.
- Tests : touche envoyée, verrou refusé à un second client, TV injoignable renvoie une erreur lisible.
- La CI lance ces tests à chaque pull request.

**Critère de fin :** un second téléphone qui envoie une touche pendant qu'un premier tient le verrou reçoit un refus qui nomme le détenteur, et la CI refuse une pull request qui casse ce test.

### Tranche 6 : télécommande web sur le téléphone

Objectif : piloter la TV depuis le S24 FE avec une adresse web, sans installation ni compte. Durée : 1 à 2 jours. Prérequis : tranche 5.

Avant de coder :

- Passer 30 minutes sur l'application Google TV et sur Home Assistant, et écrire en cinq lignes ce que ton interface apporte de plus (verrou, accueil, agent).

Interface :

- Application web installable (PWA) : croix directionnelle, OK, retour, accueil, volume.
- Raccourcis vers Netflix, YouTube, Prime Video, le navigateur (`com.internet.tvbrowser`) et le lecteur USB.
- Aperçu de l'écran de la TV grâce à `GET /screenshot`.
- Indicateur de verrou : libre ou occupé par qui.

**Critère de fin :** depuis le S24 FE, sans rien installer, on ouvre l'adresse, on lance YouTube et on navigue. Un colocataire fait de même après la libération du verrou.

### Tranche 7 : accueil composé côté serveur

Objectif : un accueil soigné avec photos, météo et infos, sans reprendre de mémoire à la TV. Durée : 1 à 2 jours. Prérequis : tranche 5. Le mécanisme (fond d'écran poussé par un thème) reste à valider en premier.

Mécanisme :

- Pousser un thème de test avec `tools/theme.sh` et noter le temps pris et ce que l'écran fait pendant la mise à jour.
- Mesurer le coût du redémarrage de Home à chaque rafraîchissement. Si c'est gênant, rafraîchir une fois par jour ou quand la TV est au repos (question ouverte).

Contenu :

- Rotation de photos 4K libres de droits (Unsplash, Pexels), dont une de Centrafrique (les chutes de Boali par exemple). Vérifier la licence de chaque image et noter l'auteur.
- Redimensionner chaque image en 1920×1080 côté serveur. Une image 4K décodée occupe environ 33 Mo, ce que la TV ne peut pas se permettre.
- Choisir une API météo gratuite et afficher la météo du jour.
- Décider des infos affichées (date, une ligne d'actualité, prochain événement), question ouverte.
- Composer l'image en HTML et CSS côté serveur, puis la convertir en image fixe.

**Critère de fin :** l'accueil de la TV change d'image sans intervention et affiche la météo du jour. La mémoire disponible mesurée avec l'accueil actif reste à 20 Mo près de la mesure sans le serveur.

### Tranche 8 : agent IA qui pilote la TV, en texte

Objectif : taper une demande en français sur le site et voir la TV l'exécuter. Durée : 2 jours. Prérequis : tranches 5 et 6 (la capture d'écran).

Outils de l'agent :

- `lancer_app`, `appuyer_touche`, `capture_ecran`, `etat_tv`, tous branchés sur le pont ADB de la tranche 5.
- Boucle avec l'API Claude : après chaque action, la capture est renvoyée au modèle pour qu'il voie où il en est.
- Plafond d'actions par demande (10 au départ) et arrêt immédiat si le verrou est perdu.

Sécurité et coût :

- Liste blanche des actions. Jamais d'achat, jamais de saisie de mot de passe.
- Journal de chaque demande et de chaque action.
- Clé API dans `.env`, plafond de dépense mensuel fixé côté compte.
- Scénarios de test rejoués sur la fausse TV.

**Critère de fin :** trois demandes écrites (« ouvre YouTube », « mets Netflix », « reviens à l'accueil ») produisent la bonne action sur la TV, chacune apparaît dans le journal, et une demande hors liste blanche est refusée.

### Tranche 9 : voix depuis le téléphone

Objectif : parler au téléphone au lieu de taper. Durée : 1 à 2 jours. Prérequis : tranche 8.

- Tester la reconnaissance vocale du navigateur du S24 FE en français, sur 10 phrases de commande.
- Bouton « appuyer pour parler » dans l'application web, transcription affichée avant envoi.
- Décider où la réponse s'affiche : sur le téléphone seulement, ou aussi sur la TV (question ouverte).
- Gérer le verrou : une commande vocale sans verrou est refusée avec un message clair.

**Critère de fin :** dire « ouvre YouTube » au téléphone ouvre YouTube sur la TV en moins de 5 secondes, sur 10 essais sur 10. Le seuil de 5 secondes est une proposition à valider.

## Les 12 étapes d'un projet professionnel

Le projet a avancé sur le développement du débloat et sur un début de cadrage. Les autres étapes sont à faire ou à peine entamées, et la production et la conformité pèsent le plus pour un projet défendable.

| Étape | Où en est le projet | Tranche |
| --- | --- | --- |
| Discovery | Besoin identifié : TV lente et pleine de pubs, pas de télécommande partagée. Aucun colocataire n'a été interrogé | 2 |
| Benchmark | CleanYourTV étudié. L'application Google TV et Home Assistant ne sont pas comparés | 6 |
| Cadrage | Partiel : ce document. Ambition, temps disponible et public ne sont pas tranchés | 2 |
| Design | Rien de dessiné. L'accueil et la télécommande n'ont pas de maquette | 6 et 7 |
| Architecture | Schéma proposé en conversation (téléphone, serveur, pont ADB, API Claude), non validé, aucun diagramme dans un dépôt | 5 |
| Développement | Débloat et launcher faits avec les outils de CleanYourTV. Aucun code à toi | 5 à 9 |
| Qualité | Aucun test, aucune CI | 2 et 5 |
| Production | Pas d'endroit pour faire tourner le serveur. Un serveur distant (le VPS) ne peut pas joindre la TV sans tunnel | 5 |
| Observabilité | Aucune. Prévu : journal des actions du pont et état de la TV | 5 et 8 |
| Conformité | Port ADB ouvert sur le réseau local sans authentification, licence de CleanYourTV à lire, licences des photos | 2, 5 et 7 |
| Mesure | RAM et swap mesurés avant et après le débloat, mesure finale manquante. Aucun suivi d'usage | 2 |
| Modèle économique | Aucun. Projet personnel et portfolio, à confirmer | Hors tranches |

## Dette technique et décisions ouvertes

Le problème le plus grave est le Wi-Fi cassé, dont la cause est inconnue. Le deuxième est que le travail n'est dans aucun dépôt à toi.

### Dette technique connue

| Problème | Gravité | Tranche |
| --- | --- | --- |
| Wi-Fi de la TV cassé depuis le redémarrage suivant le lot 5, cause inconnue | Critique | 1 |
| Travail dans un clone de Gathin23/cleanyourtv : pas de dépôt à toi, risque de perte | Haute | 2 |
| `backup/` contient des réglages de la TV (identifiants de compte et de réseau possibles) et ne doit jamais être publié | Haute | 2 |
| ADB réseau ouvert sur le réseau local sans authentification : tout appareil du Wi-Fi peut parler à la TV | Haute | 5 |
| Adresse IP de la TV instable (192.168.1.98 en Wi-Fi, autre adresse en Ethernet, non notée) | Moyenne | 3 |
| Mesure finale de RAM et de swap jamais relevée | Moyenne | 2 |
| Comportement en veille non vérifié sur ta carte | Moyenne | 3 |
| `restore.sh` jamais exécuté : le retour arrière est supposé, pas prouvé | Moyenne | 1 |
| `android.autoinstalls.config.vestel.malin` non désactivé (garde-fou), gain nul | Faible | Aucune |
| Tuile « Play Films » inutilisable depuis la coupure de `com.google.android.videos` | Faible | Aucune |
| Un fichier déposé dans `work/` pendant l'audit reste à nettoyer | Faible | 2 |

### Décisions encore ouvertes

| Décision | Options | Gravité si on tarde |
| --- | --- | --- |
| Où tourne le serveur | PC de la coloc allumé en permanence, mini-ordinateur type Raspberry Pi, ou VPS avec tunnel vers le réseau local | Haute : bloque la tranche 5 |
| Langage du serveur | Node ou Python | Moyenne |
| Un ou deux dépôts | Un dépôt à toi, plus un fork de CleanYourTV pour la pull request | Moyenne |
| Authentification de la télécommande | Aucune (réseau de la coloc), code de salon, ou liste d'appareils | Moyenne |
| Fréquence de rafraîchissement de l'accueil | Toutes les 15 minutes, une fois par jour, ou quand la TV est au repos | Moyenne |

## Ce qu'on refuse de faire pour l'instant

- Désinstaller un paquet de la TV. Désactiver suffit et garde le retour arrière.
- Rooter la TV.
- Forker le launcher pour lui ajouter le réseau et des images en mémoire : la TV n'a pas la RAM.
- Exposer le pont ADB ou la télécommande sur Internet.
- Laisser l'agent IA acheter, saisir un mot de passe ou lancer une commande shell libre.
- Désactiver `mediashell`, `com.google.android.tv.remote.service` ou `com.mediatek.network`, même pour gagner de la RAM.

## Vision long terme

Non prioritaire : rien de ce qui suit ne se commence avant la fin de la tranche 6.

- Version 2, mode salon : plusieurs personnes se connectent au même salon avec leur téléphone pour jouer ensemble à des jeux, à des quiz ou à d'autres activités sur la TV. Le verrou d'une personne à la fois de la tranche 5 devient alors un salon avec plusieurs joueurs.
- Application Android TV légère qui affiche les réponses de l'agent sur la TV.
- Thèmes de l'accueil par saison, photos choisies par la coloc.
- Liaison avec la domotique de la coloc.
- Mode démonstration sans TV, pour montrer le projet à quelqu'un qui n'a pas la même carte.

## Questions ouvertes

Les réponses du premier groupe changent toutes les autres : commence par elles.

### Ambition et cadre

| Question | Ce que la réponse débloque |
| --- | --- |
| Portfolio d'abord, ou produit pour la coloc d'abord ? | L'ordre des tranches 5 à 7 et le niveau de tests et de documentation exigé |
| Combien d'heures par semaine, et quelle date butoir (candidatures, rentrée) ? | Le découpage, et le nombre de tranches réellement faisables |
| Seul, ou avec un colocataire sur le code ? | Les règles de la section « Comment on travaille ensemble » |
| Dépôt public ou privé ? | Ce qu'on peut publier (captures, listes, journal), et la relecture de `backup/` |
| Le projet doit-il se montrer sans la TV (vidéo, mode démonstration) ? | La fausse TV de la tranche 5 devient un livrable et pas un outil de test |
| Nom du projet ? | Le dépôt, le README, l'adresse de la télécommande |

### Technique

| Question | Ce que la réponse débloque |
| --- | --- |
| Quelle est l'IP de la TV en Ethernet ? | La tranche 1 |
| Où tourne le serveur : PC, mini-ordinateur, VPS avec tunnel ? | La tranche 5 et la fiabilité en veille |
| Node ou Python pour le serveur ? | La structure du dépôt et la CI |
| La TV se réveille-t-elle par Chromecast ou Wake-on-LAN ? | La tranche 3, et donc ce que la télécommande peut faire quand la TV dort |
| Le fond d'écran poussé par un thème tient-il sans redémarrer Home à chaque fois ? | La fréquence de rafraîchissement de la tranche 7 |

### Produit

| Question | Ce que la réponse débloque |
| --- | --- |
| Quelles infos sur l'accueil (météo, date, actualité, agenda de la coloc) ? | Le contenu de la tranche 7 |
| Quelles actions l'agent a-t-il le droit de faire, et lesquelles jamais ? | La liste blanche de la tranche 8 |
| Où s'affiche la réponse de l'agent : téléphone, TV, les deux ? | La tranche 9 et l'idée d'une application Android TV |
| Salon à plusieurs prévu en version 2 : quels jeux ou activités en premier ? | L'architecture du verrou dès la tranche 5 |
