# Minecraft-Aly

Un clone simplifié de Minecraft qui tourne entièrement dans le navigateur, sans installation ni serveur.

## Démarrage rapide

Ouvrir `index.html` directement dans un navigateur moderne (Chrome, Firefox, Edge). Aucune dépendance à installer — Three.js est chargé via CDN.

## Contrôles

| Touche | Action |
|---|---|
| WASD / Flèches | Se déplacer |
| Souris | Regarder (pointer lock) |
| Espace | Sauter (marche) / Monter (vol) |
| Shift | Descendre (vol) |
| F | Basculer marche ↔ vol |
| Clic gauche | Casser un bloc (portée 6 unités) |
| Clic droit | Placer un bloc |
| 1–7 | Sélectionner le type de bloc |
| Molette | Faire défiler les blocs |

## Types de blocs

| # | Bloc | Couleur |
|---|---|---|
| 1 | Gazon | Vert (#5a9e2f dessus, brun sur les côtés) |
| 2 | Terre | Brun (#8b6340) |
| 3 | Pierre | Gris (#888888) |
| 4 | Sable | Beige (#d4c87a) |
| 5 | Bois | Brun foncé (#7a5c2e) |
| 6 | Feuilles | Vert foncé (#3a7a25) |
| 7 | Eau | Bleu (#1a5fa0) |

## Architecture technique

### Fichier unique

Tout le jeu tient dans `index.html` : HTML, CSS et JavaScript en un seul fichier. Moteur 3D : **Three.js 0.165.0** via importmap CDN.

### Génération du monde

- Monde procédural infini généré à la volée par chunks de **16×16** blocs
- Hauteur du monde : **64** blocs (Y-major, `blockIndex = ly*256 + lz*16 + lx`)
- Niveau d'eau : Y=20
- Terrain : bruit fractal à **5 octaves**, échelle 0.008, hauteur entre 8 et 48 blocs
- **Arbres** : probabilité 1,5 % par colonne (gazon uniquement), tronc 4–6 blocs, feuillage 5×5

### Rendu

- Distance de rendu : **4 chunks** dans chaque direction (9×9 = 81 chunks max)
- Les chunks hors portée sont déchargés et leur géométrie libérée
- Meshes construits par face exposée seulement (culling bloc à bloc)
- Ombrage de face fixe : dessus 100 %, côtés +X/-X 80 %, côtés +Z/-Z 65 %, dessous 50 %
- Brouillard atmosphérique de 60 % à 100 % de la distance de rendu
- Éclairage : lumière directionnelle solaire (DirectionalLight) + lumière ambiante

### Physique

- Gravité : −28 m/s²
- Vitesse de marche : 6 m/s, de vol : 12 m/s
- Vitesse de saut : 9 m/s
- Collisions **AABB** : joueur 0.6×1.9×0.6 unités
- Les mouvements X, Y, Z sont résolus séparément pour éviter le blocage aux arêtes
- L'eau et les feuilles sont traversables (non solides)

### Interaction blocs

- Raycast par pas de 0.05 unités jusqu'à 6 unités de distance
- Casser : remplace le bloc visé par AIR
- Placer : insère dans la case précédant l'impact, avec vérification anti-overlap joueur
- Modification d'un bloc sur une frontière de chunk déclenche le rebuild des chunks adjacents

## Constantes clés

```js
CHUNK_SIZE    = 16      // taille d'un chunk en X et Z
WORLD_HEIGHT  = 64      // hauteur max du monde
RENDER_DIST   = 4       // chunks rendus dans chaque direction
GRAVITY       = -28     // accélération gravitationnelle (m/s²)
JUMP_SPEED    = 9       // vitesse initiale de saut (m/s)
WALK_SPEED    = 6       // vitesse de déplacement au sol (m/s)
FLY_SPEED     = 12      // vitesse de déplacement en vol (m/s)
PLAYER_HEIGHT = 1.7     // hauteur de la caméra au-dessus du pied du joueur
```

## HUD

- **Viseur** (croix blanche, position fixe au centre)
- **Barre de blocs** (hotbar 7 slots en bas de l'écran, slot actif surligné en blanc)
- **Infos** (coin supérieur gauche) : position XYZ, bloc sélectionné, mode marche/vol, FPS
