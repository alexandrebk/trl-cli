# trl

Interface en ligne de commande épurée par-dessus Trello. Trello reste le
backend : `trl` change juste l'affichage pour éviter la surcharge d'infos de
l'interface web. Tu travailles par projet, chaque projet a son board, et `trl`
le retrouve tout seul via un fichier `.trello`.

## Installation

`trl` est un script **bash** (curl + jq), sans dépendance liée à une version de
langage — il s'utilise donc tel quel depuis n'importe quel projet, quelle que
soit sa stack (pas de souci rbenv/bundler/venv).

Dépendances :

- `curl` (présent par défaut sur macOS)
- `jq` — **requis** : `brew install jq`
- `fzf` — **optionnel**, pour le mode interactif : `brew install fzf`

```bash
# Rendre le script accessible partout via un dossier de ton PATH
chmod +x trl
ln -sf "$PWD/trl" ~/bin/trl       # ~/bin est dans le PATH ; sinon /usr/local/bin
```

## Authentification

`trl` a besoin de deux identifiants Trello, à générer sur
<https://trello.com/power-ups/admin>. Le plus simple est un fichier `.env`
placé à côté du script (il est ignoré par git) :

```bash
cp .env.example .env
# puis édite .env :
#   TRELLO_API_KEY=…
#   TRELLO_TOKEN=…
```

`trl` charge automatiquement ce `.env`. Tu peux aussi exporter les variables
dans ton shell — elles prennent alors le pas sur le `.env` :

```bash
export TRELLO_API_KEY="…"
export TRELLO_TOKEN="…"
```

Si l'une manque, `trl` s'arrête avec un message clair.

## Board courant

Le board d'un projet est défini par un fichier `.trello` à sa racine :

```
board: mon-projet
```

(le nom **ou** l'ID/shortLink du board). À chaque commande, `trl` cherche ce
fichier en remontant l'arborescence depuis le répertoire courant (comme `git`
avec `.git`). Un `<board>` passé explicitement en argument prime toujours.

```bash
trl init mon-projet     # crée le .trello ici
```

## Commandes

```
trl                      mode interactif fzf (sinon liste statique = trl ls)
trl pick [board]         force le mode interactif (carte -> menu d'actions)
trl ls [board]           mes cartes, groupées par colonne (statique)
trl all [board]          toutes les cartes du board
trl boards               mes boards (identifiant court réutilisable + nom)
trl mv <carte> <liste>   déplace une carte vers une autre colonne
trl add <liste> "titre"  crée une carte dans une colonne
trl check <carte> <item> coche un item de checklist
trl init <board>         crée un .trello dans le répertoire courant
trl help                 affiche l'aide
```

### Mode interactif (fzf)

Lancé sans argument dans un terminal, `trl` ouvre la liste de **mes cartes**
dans `fzf` (si installé). Tu navigues, tu filtres en tapant, puis :

- **↑/↓** ou **Ctrl-J / Ctrl-K** : se déplacer
- **Entrée** : ouvrir le menu d'actions sur la carte :
  - **Démarrer une branche git** (= « lancer une tâche ») — en une fois :
    1. crée/checkout `<numéro>/[feature/]<description>` (numéro + description
       dérivés de l'URL de la carte, description en underscores ; ex.
       `15567/corriger_le_bug_de_connexion`) ;
    2. déplace la carte vers la colonne **En cours** (si pas déjà dedans) ;
    3. copie la description dans `todo.md` (racine du repo) — écrase sans
       demander sur une branche **neuve**, demande confirmation sinon ;
    4. ajoute la tâche à `daily.md` (racine du repo, groupé par date) ;
    5. puis quitte `trl` pour te laisser coder.
  - **Copier la description dans todo.md** — écrit la description + les
    checklists (en cases `- [ ]`) dans `todo.md`, avec confirmation si le
    fichier existe déjà
  - Ouvrir dans le navigateur
  - Déplacer vers une autre colonne
  - Cocher / décocher un item de checklist
  - Voir les détails (description + checklist, dans le terminal)
  - ← Retour à la liste
- **Échap** : remonter d'un niveau (menu d'actions → liste ; liste → quitter)

Les cartes sont **groupées par colonne dans l'ordre du board** (gauche→droite).

Pour rester lisible sur les gros boards, l'affichage est limité aux **20 cartes
les plus récentes** (par date de création) ; une note indique le total quand
c'est tronqué. Surchargeable via `TRL_CARD_LIMIT`, soit ponctuellement
(`TRL_CARD_LIMIT=50 trl`), soit durablement dans le `.env`
(ligne `TRL_CARD_LIMIT=50`). La variable du shell prime sur le `.env`.

Sans `fzf` (ou hors terminal, ex. dans un pipe), `trl` retombe sur la liste
statique de `trl ls`.

### Arguments flous

`<board>`, `<carte>`, `<liste>` et `<item>` acceptent soit un ID Trello, soit
un match partiel insensible à la casse sur le nom. En cas d'ambiguïté, `trl`
affiche les candidats et te demande de préciser.

### « Mes cartes »

`trl ls` ne montre que les cartes où tu es membre assigné (identité récupérée
via `members/me`). Utilise `trl all` pour voir l'intégralité du board.

## Sortie

Couleurs ANSI sobres, automatiquement désactivées si la sortie n'est pas un
terminal (ou si `NO_COLOR` est défini). Indicateurs par carte : `☐ x/y` pour
une checklist, `📅 date` pour une échéance (rouge si dépassée).
