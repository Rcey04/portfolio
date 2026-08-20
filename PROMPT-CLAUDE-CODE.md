# Brief pour Claude Code

À coller comme premier message dans Claude Code, ouvert depuis le dossier
qui contient à la fois `portfolio/` et mes dossiers de projets.

---

## Contexte

Tu reprends le portfolio en ligne de Rojhat Ceylan, médiamaticien CFC basé
à Lausanne. Le site sert à décrocher des postes en agence, en institution
culturelle et en service communication. Le lecteur type est un recruteur
qui accorde trente secondes à la page d'accueil avant de décider s'il
ouvre une étude de cas.

HTML, CSS et JavaScript natifs. Pas de framework, pas de build, pas de
dépendance. Le site est mis en ligne en glissant le dossier sur Netlify
Drop, il doit donc rester ouvrable par simple double-clic sur `index.html`.

## État actuel du dossier portfolio

```
index.html                page d'accueil, index dépliable des cinq projets
projet-primmoclean.html   étude de cas complète, sert de gabarit
images/                   visuels JPEG + LISEZ-MOI listant ce qui manque
apercus/                  copies autonomes en base64, à ignorer et ne pas modifier
```

Les cinq projets : Akoust Atelier (2024), PrimmoClean (2025),
Reyes Salon (2026, en cours), BlenPro (2026), ASFP (2025 à 2026).
Seul PrimmoClean a son étude de cas et ses visuels.

---

# MISSION 1, la plus importante : récupérer les visuels

Le portfolio est vide à quatre-vingts pour cent parce que les visuels
manquent. Ta première tâche est d'aller les chercher toi-même sur ma
machine plutôt que de me demander de les fournir.

## Étape 1, inventaire

Explore le système de fichiers et dresse la liste de tout ce qui concerne
mes cinq projets. Cherche par nom de dossier et par nom de fichier les
termes suivants, en ignorant la casse et les accents :

```
akoust, luthier, tpi
primmoclean, primmo
reyes, salon
blenpro, blen, sanitaire
asfp
```

Regarde en priorité dans `~/Documents`, `~/Desktop`, `~/Downloads`,
`~/Sites`, `~/Projets`, `~/Dev` et leurs sous-dossiers. Signale-moi tout
dossier qui contient un `package.json`, un `index.html`, un `.fig`, un
`.xd`, un `.ai`, un `.indd`, un `.psd` ou un PDF volumineux.

Présente-moi cet inventaire sous forme de tableau avant d'aller plus loin :
projet, chemin, type de contenu, exploitable ou non. Attends ma validation.

## Étape 2, captures des projets web

Pour chaque projet qui est un site, local ou en ligne :

- Installe Playwright si nécessaire (`npm i -D playwright`), c'est la seule
  dépendance de développement autorisée, et elle reste hors du dossier
  `portfolio/`.
- Si le projet a un serveur de développement, lance-le et capture en local.
  Sinon capture la version en ligne. PrimmoClean est sur primmoclean.ch.
- Capture en `deviceScaleFactor: 3`, pas 1. C'est ce qui fait la netteté
  sur écran Retina.
- Trois formats pour chaque page importante :
  desktop 1440 de large, tablette 834, mobile 390.
- Capture la page entière (`fullPage: true`) et aussi les sections
  intéressantes isolées, en ciblant leurs sélecteurs CSS.
- Attends la fin des animations et le chargement des polices avant de
  déclencher : `waitForLoadState('networkidle')` puis
  `document.fonts.ready`.
- Masque les bandeaux de cookies et les widgets tiers avant capture.

Écris ce travail dans `outils/captures.mjs`, avec la liste des pages en
haut du fichier pour que je puisse la modifier.

## Étape 3, extraction des documents

- Pour mon rapport de TPI Akoust Atelier, qui est un PDF de 68 pages :
  extrais toutes les pages en PNG à 300 dpi avec `pdftoppm -r 300 -png`,
  puis montre-moi des vignettes pour que je choisisse les six à dix
  planches les plus fortes.
- Pour les fichiers Illustrator, InDesign ou Photoshop que tu trouves, tu
  ne peux pas les ouvrir. Liste-les-moi simplement, je les exporterai
  moi-même.
- Pour les fichiers Figma, même chose : donne-moi la liste, je les
  exporterai en PNG 3x.

## Étape 4, chaîne d'optimisation

Écris `outils/optimiser-images.sh` qui prend un dossier `sources/`
contenant les originaux et produit dans `images/` quatre variantes par
visuel :

```
nom.webp      largeur standard
nom@2x.webp   largeur doublée
nom.jpg       repli standard
nom@2x.jpg    repli doublé
```

Règles :
- Largeurs standard : 1600 pour les visuels pleine largeur, 900 pour les
  images en colonne, 480 pour les vignettes de l'index.
- WebP en qualité 82, JPEG en qualité 84 avec `-interlace Plane`.
- Redimensionne avec `-filter Lanczos`, jamais par défaut.
- Supprime les métadonnées avec `-strip`.
- Ne recompresse jamais un fichier déjà compressé : pars toujours de
  `sources/`, qui n'est pas versionné et n'est pas mis en ligne.
- Refuse et signale tout original dont la largeur est inférieure à deux
  fois la largeur cible, plutôt que d'agrandir une image.

Vérifie ensuite qu'aucun fichier de `images/` ne dépasse 250 Ko, et
liste-moi ceux qui posent problème.

## Étape 5, intégration dans les pages

Remplace chaque `<img>` du site par un bloc `<picture>` :

```html
<picture>
  <source type="image/webp" srcset="images/nom.webp 1x, images/nom@2x.webp 2x">
  <img src="images/nom.jpg" srcset="images/nom.jpg 1x, images/nom@2x.jpg 2x"
       width="1600" height="900" alt="description précise"
       loading="lazy" decoding="async">
</picture>
```

Sauf pour l'image de hero de chaque page, qui garde `loading="eager"` et
reçoit `fetchpriority="high"`.

Les attributs `width` et `height` sont obligatoires partout, pour éviter
que la page saute pendant le chargement.

Les textes alternatifs doivent décrire ce qu'on voit, pas répéter le nom
du projet.

---

# MISSION 2 : compléter les études de cas

Duplique `projet-primmoclean.html` pour les quatre autres projets, en
gardant strictement la même structure : hero, colonnes de compétences,
objectifs numérotés, sections alternant texte court et grandes images,
palette, résultats, lien vers le projet suivant.

Remplis uniquement ce que tu peux constater dans les fichiers que tu as
trouvés. Pour tout le reste, laisse un marqueur `À COMPLÉTER` visible dans
la page. N'invente jamais un chiffre, un résultat, une contrainte client
ou une intention de conception.

Relie chaque étude de cas depuis `index.html` avec le lien
"Lire l'étude de cas", déjà présent sous PrimmoClean.

---

# MISSION 3 : finitions

- Métadonnées Open Graph et Twitter Card sur chaque page, avec image de
  partage en 1200 x 630, pour que les liens envoyés par email ou LinkedIn
  affichent un aperçu correct.
- `favicon.svg` sobre construit sur les initiales RC.
- `sitemap.xml` et `robots.txt`.
- Vérification du rendu à 375, 768, 1280 et 1920 pixels.
- Lighthouse au-dessus de 95 en performance et en accessibilité sur chaque
  page.

---

## Conventions à respecter absolument

**Design.** Palette `--paper:#FBFBF9`, `--ink:#0C0D0F`, `--grey:#8A8D93`,
`--hair:#E2E1DC`, `--mark:#D8232A`. Polices Archivo et JetBrains Mono via
Google Fonts. Transitions en `cubic-bezier(.16,1,.3,1)`. Aucune nouvelle
couleur, aucune nouvelle police.

**Interactions existantes, à préserver.** Aperçu qui suit le curseur au
survol des lignes de l'index, avec inertie. Dépliage d'un seul projet à la
fois. Navigation au clavier par les flèches haut et bas. Barre de
progression de lecture sur les études de cas. Apparition au défilement via
IntersectionObserver.

**Accessibilité.** `aria-expanded` sur les boutons dépliables, focus
visible partout, `prefers-reduced-motion` respecté.

**Rédaction.** Français, à la première personne. Pas de tirets cadratins,
jamais. Les titres de section racontent quelque chose plutôt que de nommer
une discipline : "Un métier où l'on choisit sur la confiance" et non
"Recherche utilisateur".

**Interdits.** Pas de framework, pas de bundler, pas de localStorage, pas
de dépendance dans le dossier `portfolio/` lui-même, pas de fichier CSS ou
JS séparé. Tout reste dans chaque page HTML. Ne touche pas à `apercus/`.

## Méthode

Commence par l'inventaire de la mission 1 et attends ma validation avant
de capturer quoi que ce soit. Ensuite avance projet par projet, en me
montrant le résultat après chaque étape.
