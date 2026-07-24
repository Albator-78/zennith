# ♛ Zénith — Échecs 9×9 aux Entités

Une variante d'échecs jouable dans le navigateur, sur un plateau **9×9** (colonnes `a`–`i`, rangées `1`–`9`), où deux **entités immobiles** — le Dieu **Ω** et le Diable **Ψ** — frappent à distance sans jamais bouger.

Un seul fichier autonome : `index.html`. Aucune dépendance, aucun build, fonctionne hors-ligne.
Moteur d'échecs complet + IA locale (alpha-bêta), pont **llama.cpp** optionnel, et un modèle d'évaluation **appris par self-play**.

---

## Sommaire

- [Règles](#règles)
  - [Le plateau](#le-plateau)
  - [Les pièces](#les-pièces)
  - [Les entités Ω et Ψ](#les-entités-ω-et-ψ)
  - [Les lois](#les-lois)
  - [Le losange sacré](#le-losange-sacré)
  - [Mises en place](#mises-en-place)
- [Installation](#installation)
- [Utilisation](#utilisation)
  - [Jouer](#jouer)
  - [Jouer à deux — local et en ligne](#jouer-à-deux--local-et-en-ligne)
  - [Les adversaires IA](#les-adversaires-ia)
  - [Brancher llama.cpp](#brancher-llamacpp)
  - [Entraîner le modèle](#entraîner-le-modèle)
- [Architecture du code](#architecture-du-code)
- [Développement ultérieur](#développement-ultérieur)
- [Licence](#licence)

---

## Règles

### Le plateau

Damier de 81 cases. Notation algébrique classique étendue à 9 colonnes et 9 rangées.
Chaque camp aligne : **1 Roi, 1 Dame, 2 Tours, 2 Fous, 2 Cavaliers, 9 Pions**, plus **son entité**.
Selon la mise en place, une 9ᵉ pièce (l'**Archange**) complète l'armée.

Règles standard conservées : poussée double des pions, **prise en passant**, **promotion** (Dame, Archange, Tour, Fou ou Cavalier au choix), **roque** généralisé (le Roi va de 2 cases vers la Tour, la Tour se pose juste derrière lui), détection de l'**échec**, du **mat** et du **pat**.

### Les pièces

| Symbole | Pièce | Déplacement |
|:---:|---|---|
| ♚ K | Roi | 1 case dans les 8 directions |
| ♛ Q | Dame | tour + fou |
| ♜ R | Tour | lignes orthogonales |
| ♝ B | Fou | diagonales |
| ♞ N | Cavalier | saut en L |
| ♟ P | Pion | avance ; prend en diagonale ; promotion en dernière rangée |
| ⚜ A | **Archange** | **fou + cavalier** (la pièce propre à la variante) |
| Ω / Ψ | **Entités** | ne se déplacent jamais — voir ci-dessous |

### Les entités Ω et Ψ

**Ω** (le Dieu) appartient aux Blancs, **Ψ** (le Diable) aux Noirs. Elles **ne bougent jamais** et ne peuvent **jamais être prises**.

À son tour, au lieu de jouer une pièce, un camp peut ordonner à son entité une **frappe fugace** :

> L'entité choisit une **pièce adverse** située sur l'une de ses **8 lignes** (4 orthogonales + 4 diagonales), **à condition que toutes les cases intermédiaires soient vides**. Elle s'y rend, capture, et **revient aussitôt** sur sa case. La frappe consomme le tour entier — le plateau est inchangé, seule la pièce visée disparaît.

Comme le retour est instantané, l'entité n'est jamais exposée en chemin et ne débloque aucune ligne pendant le trajet.

### Les lois

1. **La Frappe fugace** — prise à distance sur une ligne entièrement dégagée, sans déplacement.
2. **Les Invocations** — chaque entité ne dispose que d'un nombre limité de frappes pour **toute la partie** (3 par défaut, réglable de 0 à 5). Épuisée, elle devient inerte.
3. **L'Immortalité** — une entité ne peut jamais être capturée, ni sa case occupée.
4. **Le Pacte** — Ω et Ψ ne peuvent pas se frapper l'une l'autre.
5. **L'Appel** — une entité qui vise le Roi adverse sur une ligne dégagée donne **échec** — mais **seulement s'il lui reste au moins une Invocation**. Une entité épuisée ne donne plus échec ; le Roi peut traverser ses lignes.
6. **Le Recueillement** *(optionnel)* — après une frappe, l'entité reste **muette 2 tours** : elle ne frappe plus et ne donne plus échec pendant ce délai. Activé par défaut pour les mises en place où une entité vise d'emblée le Roi adverse.

### Le losange sacré

Les quatre diagonales des deux entités dessinent un losange dont les sommets sont les entités et les cases natales des Rois. La **rangée (ou colonne) centrale devient le « Purgatoire »**, sous feu croisé : la traverser tant que les Invocations subsistent, c'est s'exposer.

Le bouton **Lignes de feu** surligne en temps réel les trajectoires : or pour Ω, cramoisi pour Ψ, dégradé pour les cases battues par les deux.

### Mises en place

Trois dispositions au choix (menu **Mise en place**) :

| Clé | Entités | Particularité |
|---|---|---|
| **l'Axe** *(défaut)* | Ω `e1` · Ψ `e9` | Roi et Dame encadrent l'entité ; armée classique + 1 pion ; grand roque muré par l'entité. |
| **Cœur & Corne** | Ω `e5` · Ψ `i9` | Asymétrique ; l'Archange remplace… voir le code. Recueillement conseillé. |
| **Zénith** | Ω `a5` · Ψ `i5` | Version d'origine, armée en rotation 180°, Archange présent. |

Une entité écrase l'occupant de sa case au montage : ne jamais la placer sur la case d'un Roi.

---

## Installation

**Aucune installation.** Le projet est un unique fichier HTML.

### Option 1 — ouverture directe
Double-cliquez sur `index.html`. Tout fonctionne, **sauf** le pont llama.cpp (le protocole `file://` bloque les requêtes réseau du navigateur) et, selon le navigateur, le **jeu en ligne**.

### Option 2 — serveur local (recommandé)
Nécessaire pour utiliser llama.cpp et pour le jeu en ligne. N'importe quel serveur statique convient :

```bash
# Python (déjà présent sur la plupart des systèmes)
python -m http.server 8123 --directory .
```

Puis ouvrez <http://localhost:8123>.

> Le dépôt contient un `.claude/launch.json` avec une entrée `zenith` qui lance ce serveur — utile si vous travaillez avec Claude Code, ignorable sinon.

**Prérequis :** un navigateur récent (Chrome, Firefox, Edge, Safari). C'est tout.

---

## Utilisation

### Jouer

Cliquez une pièce puis sa destination. Les coups légaux sont surlignés ; les cibles de frappe d'une entité sont cerclées de rouge avec un éclair ⚡. Cliquez Ω ou Ψ pour voir ses frappes disponibles.

Panneau de droite : compteur d'**Invocations** (pastilles), historique en notation `Ω*d8+`, boutons **Nouvelle partie**, **Annuler**, **Lignes de feu**, **Retourner**, et les réglages (mise en place, Invocations, Recueillement).

Notation : `Ω*d8` = frappe divine sur d8 ; `+` échec, `#` mat ; `O-O` roque.

L'interface est disponible en **six langues** (français, anglais, allemand, espagnol, portugais, italien) via le sélecteur en tête de page. Le choix est mémorisé, la notation des pièces suit la langue, et les invites envoyées à llama.cpp sont rédigées dans cette même langue.

### Jouer à deux — local et en ligne

Deux joueurs humains peuvent s'affronter, sur le même appareil ou à distance.

**En local (hot-seat).** Panneau **Adversaire** → *L'IA joue* → **Personne — 2 joueurs**. Les deux camps se jouent tour à tour sur le même écran. L'option **Retourner à chaque tour** fait pivoter automatiquement le plateau pour présenter son camp au joueur au trait — pratique quand on se passe la souris ou le téléphone.

**En ligne.** Panneau **Jeu en ligne** : liaison **pair-à-pair WebRTC, sans aucun serveur**. Les deux joueurs échangent deux courts codes de connexion, par le canal de leur choix (messagerie, mail…). L'hôte joue les **Blancs**, l'invité les **Noirs**, et chacun voit son camp en bas.

| Hôte | Invité |
|---|---|
| 1. **Créer une partie** → un code apparaît, **Copier** et l'envoyer. | 1. **Rejoindre**, coller le code reçu. |
| 3. Coller la réponse reçue, puis **Se connecter**. | 2. **Générer la réponse**, la **Copier** et la renvoyer. |

Dès l'affichage de **« Connecté »** des deux côtés, la partie commence. Ensuite **seuls les coups circulent** : le moteur étant déterministe, les deux plateaux restent identiques sans jamais transmettre la position. **Nouvelle partie** et **Annuler** sont synchronisés entre les deux joueurs ; **Quitter** met fin à la session en laissant la position affichée. Les coups reçus sont revalidés localement contre la liste des coups légaux — un pair ne peut pas imposer un coup illégal.

> **Note réseau.** La connexion s'appuie sur des serveurs **STUN** publics pour traverser la plupart des box domestiques. Derrière un NAT symétrique ou un pare-feu d'entreprise, la liaison directe peut échouer : il faudrait alors un relais **TURN**, volontairement non inclus pour rester sans infrastructure. Servez le jeu via `http://localhost` plutôt qu'en `file://`, certains navigateurs restreignant WebRTC sur ce dernier.

### Les adversaires IA

Panneau **Adversaire** — choisissez qui l'IA incarne (Blancs, Noirs, ou personne) et son cerveau :

| Cerveau | Description |
|---|---|
| **Moteur local** | Alpha-bêta pur. Autonome, hors-ligne, instantané. Trois niveaux (budget de temps). |
| **Oracle** *(conseillé avec un LLM)* | Le moteur présélectionne ses 4 meilleurs coups, **llama.cpp tranche** et justifie en une phrase. Le meilleur des deux mondes. |
| **llama.cpp seul** | Le LLM choisit dans la liste des coups légaux. Sur une variante inédite il joue faible — à réserver à la curiosité. |

### Brancher llama.cpp

1. Récupérez [llama.cpp](https://github.com/ggml-org/llama.cpp) et un modèle `.gguf`.
2. Lancez le serveur (API compatible OpenAI) :
   ```bash
   llama-server -m modele.gguf -c 4096 --host 127.0.0.1 --port 8080
   ```
3. Dans le panneau **Adversaire**, choisissez **Oracle** ou **llama.cpp seul**, vérifiez l'URL (`http://127.0.0.1:8080/v1/chat/completions`) et cliquez **Tester la connexion**.

Le prompt envoyé contient les règles des entités, le plateau en ASCII, les Invocations restantes et **la liste numérotée des coups légaux** : le modèle choisit dedans, il ne génère pas un coup libre. Le parseur tolère JSON strict, JSON noyé dans du texte, coup cité en clair ou choix par numéro, et **refuse tout coup illégal**. En cas d'échec réseau ou de réponse inexploitable, le **moteur local prend le relais** — la partie ne bloque jamais.

### Entraîner le modèle

Panneau **Apprentissage**. Le moteur joue des parties contre lui-même, retient les positions calmes étiquetées par le résultat, puis ajuste ses **poids d'évaluation** (réglage texel) pour que `sigmoïde(évaluation)` prédise l'issue.

- **Entraîner** — lance N parties (30 à 400 ≈ 30 s à 8 min). Barre de progression, ne fige pas l'interface.
- **Réinitialiser** — rétablit les poids d'origine.
- **Exporter / Importer** — le modèle est un fichier `zenith-eval.json` d'environ 200 octets, partageable.

Le modèle appris est conservé automatiquement dans le `localStorage` du navigateur et rechargé au démarrage. Tous les modes d'IA l'utilisent.

> **Note.** Plus de parties = poids plus stables (12 parties suffisent à voir le mécanisme, mais fou et cavalier peuvent s'inverser ; à partir de ~60 l'ordre matériel se fixe). Le modèle n'apprend que *comment évaluer une position* — c'est toujours la recherche alpha-bêta qui calcule les variantes.

---

## Architecture du code

Tout tient dans `index.html` : `<style>`, le balisage, puis un unique `<script>` organisé en sections commentées.

```
Représentation      board[r][f], pièces {t, c}, état {board, turn, charges, dormant, …}
Règles              pseudoMoves / legalMoves / attacked / inCheck / applyMove
                    strikeMoves (frappes), castleMoves (roque généralisé)
Notation            moveText → "Ω*d8+", "O-O", "Pb8xc9=D"…
Interface           render(), gestion des clics, promotion, historique
Moteur              evaluate() = W · features ; negamax alpha-bêta + quiescence ;
                    searchTop() (approfondissement itératif sous budget de temps)
Apprentissage       selfPlayGame() → tuneWeights() (texel) ; save/load du modèle
Pont llama.cpp      asciiBoard(), userPrompt(), callLlama() (fetch OpenAI-compatible)
Arbitrage IA        maybeAI() / runAI() : Moteur / Oracle / LLM + repli
Jeu en ligne        RTCPeerConnection + RTCDataChannel ; signalisation manuelle
                    (offre/réponse base64) ; netSend / netRecv, findLocalMove
Internationalisation table I18N (6 langues), i18t(), applyI18n(), setLang()
```

**Modèle de données du plateau.** `board[r][f]` avec `r` = rangée 0–8 (1–9), `f` = colonne 0–8 (a–i). Une pièce est `{t, c}` où `t ∈ {P,N,B,R,Q,K,A,G,D}` (G = Dieu, D = Diable) et `c ∈ {"w","b"}`.

**Le « modèle IA ».** Un vecteur `W` de ~10 poids : valeurs matérielles (P, N, B, R, Q, A), avance des pions, centralisation, valeur d'une Invocation, et danger des lignes de feu. `evaluate(s)` en est le produit scalaire avec les features de la position. `evaluate()` (chemin critique, sans allocation) est maintenu **mathématiquement identique** à `evalFeat(computeFeatures(s))` (utilisé pour l'entraînement).

**Performance.** La légalité sur 9×9 est coûteuse (chaque coup est validé en simulant l'échec). Le self-play utilise donc une recherche **pseudo-légale** dédiée (`spScore`, capture du Roi = victoire) sur une *shortlist* des meilleurs coups, ce qui rend la génération de milliers de parties praticable dans le navigateur.

---

## Développement ultérieur

Pistes classées par effort croissant.

### Court terme
- **PWA installable** : ajouter un `manifest.json` et un service worker pour un jeu hors-ligne installable (dans l'esprit des autres projets du dossier).
- **Modules de règles optionnels** : l'Offrande (Invocations illimitées contre sacrifice d'un pion), Natures asymétriques (Ψ corrompt au lieu de détruire ; Ω ressuscite), l'Angle mort, l'Éveil — déjà esquissés dans la conception, à câbler en cases à cocher.
- **Sauvegarde de partie** : export/import PGN adapté au 9×9, ou reprise via `localStorage`.

### Moyen terme — renforcer le moteur
- **Table de transposition** (hachage Zobrist) : gros gain de profondeur à budget égal.
- **Move ordering** : killer moves, history heuristic, PV en tête d'itération.
- **Zobrist + répétition** : détection de la nulle par triple répétition et règle des 50 coups.

### Moyen terme — renforcer le modèle
- **NNUE** — remplacer les 10 poids linéaires par un petit réseau incrémental (~100–300 Ko quantifié, toujours pur JS/WASM). Toute l'infrastructure de self-play est réutilisable ; seule l'évaluation change. Nettement plus fort, reste portable et hors-ligne.
- **Self-play itératif** : régénérer les parties avec le modèle fraîchement appris (boucle façon AlphaZero légère) pour améliorer la qualité des cibles.
- **Web Worker** : déporter l'entraînement et la recherche hors du thread principal (fluidité, multi-cœur).

### Long terme
- **AlphaZero + MCTS** : réseau policy+value et recherche arborescente Monte-Carlo. Le plus fort à terme, mais entraînement lourd.
- **Équilibrage** : jouer des milliers de parties moteur-contre-moteur pour régler le nombre d'Invocations et valider chaque mise en place.

### Conventions
- Un seul fichier, JavaScript vanilla, pas de build — garder cette contrainte tant que possible (c'est ce qui rend le projet portable).
- Les objets pièce ne sont **jamais mutés** : `clone()` partage leurs références en toute sécurité. Toute évolution doit préserver cette invariance (le moteur en dépend pour sa vitesse).
- `evaluate()` et `computeFeatures()` doivent rester synchronisés — un test d'égalité existe (`evaluate(s) === evalFeat(computeFeatures(s))`).

---

## Licence

**GNU Affero General Public License v3.0** (AGPL-3.0) — voir le fichier [`LICENSE`](LICENSE).

Vous êtes libre d'utiliser, d'étudier, de modifier et de redistribuer ce logiciel, à condition que toute version distribuée **ou mise à disposition sur un réseau** reste sous la même licence et fournisse son **code source** aux utilisateurs.

> **Pourquoi l'AGPL ?** C'est une variante de la GPL v3 pensée pour les applications web : si vous déployez Zénith (même modifié) sur un serveur accessible à d'autres, la clause **§13 (interaction réseau)** vous oblige à en offrir le code source aux joueurs — par exemple via un lien « Source » dans l'interface. Un simple `index.html` fermé sur un serveur public ne suffit pas : la source doit suivre.

En pratique, si vous mettez une version en ligne :
- gardez le projet ouvert (dépôt public ou lien « Source » dans la page) ;
- conservez les mentions de copyright et de licence.

Le lien « Source (AGPL-3.0) » de l'en-tête pointe déjà vers le dépôt public :
<https://github.com/Albator-78/zennith/releases/tag/echec>. Pensez à le faire suivre
si vous changez d'hébergement, et à publier la version **exactement déployée**
(l'article 13 vise le code effectivement servi aux joueurs).

### À faire lors de la première publication

Remplacez l'espace réservé de l'en-tête par vos nom et année, dans `index.html` (en haut du `<script>` ou en commentaire de tête) :

```
Zénith — Échecs 9×9 aux Entités
Copyright (C) 2026  <votre nom>

Ce programme est un logiciel libre : vous pouvez le redistribuer et/ou le
modifier selon les termes de la GNU Affero General Public License telle que
publiée par la Free Software Foundation, version 3.

Ce programme est distribué dans l'espoir qu'il sera utile, mais SANS AUCUNE
GARANTIE ; voir la GNU Affero General Public License pour plus de détails.
Vous devriez avoir reçu une copie de la licence avec ce programme ; sinon,
voir <https://www.gnu.org/licenses/>.
```

---

*Projet personnel — variante d'échecs originale. Vanilla HTML/CSS/JS, sans dépendance.*
