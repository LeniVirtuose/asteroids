# Plan : ajouter un laser de bord toutes les 5 vagues

**But :** toutes les 5 vagues, un danger spécial apparaît :

1. Un **voyant rouge** clignote sur un bord de l'écran pendant **2 secondes** (c'est
   l'avertissement : il montre par où le laser va passer).
2. À la fin des 2 secondes, un **laser rouge** traverse toute la map sur cette ligne.
3. Si le personnage est sur la trajectoire du laser, il perd **100 points de vie**.

L'idée : laisser au joueur 2 secondes pour se déplacer hors de la ligne du laser.

> Comme dans le reste du jeu : on garde tous les nombres (réglages) regroupés en
> haut du fichier `index.html`, et on teste dans le navigateur après chaque étape.

---

## Étape 1 — Les réglages du laser

Ajouter en haut de `index.html`, avec les autres réglages, de nouvelles constantes :

- `LASER_TOUTES_LES = 5` → un laser à chaque vague multiple de 5 (5, 10, 15…).
- `LASER_ALERTE_MS = 2000` → durée du voyant d'avertissement (2 secondes).
- `LASER_DEGATS = 100` → points de vie enlevés si on est touché.
- `LASER_EPAISSEUR = 30` → épaisseur du trait rouge (sert aussi pour savoir si le
  personnage est touché).
- `LASER_DUREE_MS = 250` → combien de temps le laser reste affiché quand il tire
  (un éclair court).
- `COULEUR_LASER = "red"` (et une couleur plus pâle pour le voyant qui clignote).

**Pourquoi :** comme ça on peut facilement régler la fréquence, le temps de réaction
et les dégâts sans toucher au reste du code.

---

## Étape 2 — Mémoriser l'état du laser

Dans l'état du jeu (`etat`, là où il y a déjà `vague`, `ennemis`, etc.), ajouter un
petit objet qui décrit le laser en cours :

```
laser: {
  phase: "inactif",   // "inactif" | "alerte" | "tir"
  y: 0,               // la hauteur (ligne horizontale) où le laser va passer
  finPhase: 0,        // l'heure (Date.now) à laquelle la phase actuelle se termine
  dejaTouche: false,  // pour n'enlever les 100 PV qu'une seule fois par laser
}
```

Penser à le remettre à `"inactif"` dans `etatInitial()` (pour que **Rejouer** reparte
proprement, comme pour le reste).

---

## Étape 3 — Déclencher le laser au début des bonnes vagues

Dans `demarrerVague()` (la fonction appelée au début de chaque vague) :

- Si `etat.vague % LASER_TOUTES_LES === 0`, alors démarrer la phase **alerte** :
  - choisir une hauteur au hasard : `etat.laser.y = Math.random() * hauteur du canvas`
    (c'est la ligne horizontale que le laser va suivre) ;
  - `etat.laser.phase = "alerte"` ;
  - `etat.laser.finPhase = Date.now() + LASER_ALERTE_MS` ;
  - `etat.laser.dejaTouche = false`.

**Test :** à la vague 5, un voyant doit apparaître (on le dessine à l'étape 5).

---

## Étape 4 — Faire avancer le laser dans le temps

Créer une fonction `gererLaser()` appelée à chaque image (dans la boucle de jeu,
à côté de `gererVagues()`), qui regarde la phase actuelle :

- **phase "alerte"** : quand `Date.now() >= etat.laser.finPhase`, passer en phase
  **"tir"** et fixer `etat.laser.finPhase = Date.now() + LASER_DUREE_MS`.
- **phase "tir"** :
  - si le personnage est sur la ligne du laser **et** qu'on ne l'a pas encore touché
    (`!etat.laser.dejaTouche`), lui enlever `LASER_DEGATS` PV puis mettre
    `dejaTouche = true`.
    - « être sur la ligne » = `Math.abs(etat.y - etat.laser.y) < LASER_EPAISSEUR / 2 + RAYON_VAISSEAU`.
    - réutiliser la fonction de dégâts existante (`subirChoc(LASER_DEGATS)`) pour
      profiter de l'invincibilité/clignotement déjà en place.
  - quand `Date.now() >= etat.laser.finPhase`, repasser en phase **"inactif"**.

**Test :** au bout des 2 secondes, si on est resté sur la ligne, on perd 100 PV ;
si on s'est déplacé, on ne perd rien.

---

## Étape 5 — Dessiner le voyant puis le laser

Dans la fonction de dessin, ajouter le visuel selon la phase :

- **phase "alerte"** : dessiner un trait rouge **fin et clignotant** sur toute la
  largeur à la hauteur `etat.laser.y`, plus un petit **voyant rouge** bien visible
  sur le bord (gauche et/ou droite) à cette hauteur.
  - le clignotement : afficher seulement une image sur deux, par exemple en se
    basant sur le temps (`Math.floor(Date.now() / 150) % 2`).
- **phase "tir"** : dessiner un **gros trait rouge** épais (`LASER_EPAISSEUR`) sur
  toute la largeur, à la hauteur `etat.laser.y` (éventuellement avec un effet de
  lueur en dessinant un trait plus large et plus transparent derrière).

**Test :** on voit le voyant clignoter 2 secondes, puis un éclair rouge traverser
l'écran.

---

## Étape 6 — Cas particuliers à vérifier

- **Pause** : pendant la pause, le laser ne doit pas avancer (gérer comme le reste
  du jeu : ne pas appeler `gererLaser()` quand le jeu est en pause).
- **Boss (toutes les 10 vagues)** : la vague 10, 20… est à la fois une grosse vague
  ET un laser → vérifier que les deux fonctionnent ensemble.
- **Rejouer** : après une partie perdue, le laser doit bien repartir à `"inactif"`.

---

## Récapitulatif de l'ordre

1. Ajouter les réglages du laser (constantes en haut du fichier).
2. Ajouter l'objet `laser` dans l'état (et le réinitialiser dans `etatInitial`).
3. Déclencher la phase « alerte » au début des vagues multiples de 5.
4. `gererLaser()` : alerte → tir → inactif, et enlever 100 PV si on est touché.
5. Dessiner le voyant clignotant puis le laser.
6. Vérifier pause, vagues « boss » et Rejouer.

> Conseil : faire les étapes **dans cet ordre** et tester à chaque fois dans le
> navigateur. On peut s'arrêter après n'importe quelle étape, le jeu reste jouable.
