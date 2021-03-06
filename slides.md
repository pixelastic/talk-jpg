class: full-page, slide-intro
# Autopsie d'un JPG
---
class: content-centered
# Pourquoi parler de JPEG ?
- la web perf est un sujet d'actualité
- le JPEG est massivement utilisé
- ... bien qu'il date du début des années 90
- ... et qu'il est parfois incompris ou mal réglé
- Un exemple simple de compression perceptuelle (images, audio, vidéo...)

---
class: content-centered
# JPEG : historique
- **J**oint **P**hotographic **E**xperts **G**roup a créé le standard *Information technology – Digital compression and coding of continuous-tone still images*
- Le groupe a démarré 1986, le standard a été publié en 1992 (apprové par l'ISO et l'ITU)
- Méthode de compression d'image
- Format de fichier associé : **J**peg **F**ile **I**nterchange **F**ormat (JFIF)

---
class: content-centered
# Images : >50% du payload
.center[
  ![Image payload](./img/img_to_transfer_size.png)

  [HTTP Archive (2011-2015)](http://httparchive.org/trends.php?s=All&minlabel=Jun+1+2011&maxlabel=Apr+1+2015#bytesImg&reqImg)
]

---
class: content-centered
# Sommaire

- Fonctionnement de la compression JPEG
- Metadatas
- REX web perf & outils utiles

---
class: content-centered
# Que se cache-t-il derrière ?
.center[
  ![UI d'export de Gimp](./img/gimp-export.jpg)
]

---
class: content-centered
# Vision naïve d'une image

.center[
  ![Example 400x268](./img/example-400.jpg)
]

 

- 400 x 268 = 107 200px
- RGB = 3 bytes
- 321 600 ≠ 111 858 ≠ 9 412

---
class: content-centered
# Une combinaison de méthodes

### Compression sans perte (lossless)
- Techniques issues de la théorie de l'information 
- Algèbre linéaire

### Compression avec perte (lossy)
- Techniques psychovisuelles
- Quantification

---
class: content-centered
# Encodage et décodage

.center[
  ![Diagramme de flux](./img/500px-JPEG_process.svg.png)
]

- Temps de traitement et optimisations sur l'encodeur
- Décodeur plutôt rapide et standardisé


---
class: content-centered
# Division par blocs de 8x8

.center[
  ![Blocs 8x8](./img/example-400-grid.jpg)
]

- Compression par bloc de 64 pixels (Minimum Coded Unit)
- Artefacts visibles à l'œil nu
- [Bon compromis](http://www.faqs.org/faqs/mpeg-faq/part3/) de taille entre localité et performances (à l'époque)

---
class: content-centered
# Zoom sur un bloc 8x8
.column-left[
![Zoom](./img/example-block-zoom.jpg)
]
.column-right[
![Un bloc](./img/320px-JPEG_ZigZag.svg.png)
]

 

- 8x8 c'est petit, surtout pour les résolutions actuelles
- Chaque MCU est considérée comme un vecteur de taille 64 ordonné en zig-zag

---
class: content-centered
# Changement de perspective
.center[
  ![Example YCbCr](./img/example-YCbCr.png)
]

- **RGB ➔ YCbCr** (Luminance et chrominance)
- Changement de base dans l'espace vectoriel des MCU
- Lossless (tant qu'on n'arrondit pas)
- L'œil humain est bien plus sensible à la luminance 

---
class: content-centered
# Subsampling en chrominance
.center[
  ![Chroma Subsampling](./img/Chroma Subsampling.png)
]

- Reduction de la chrominance moins bien perçue par l'œil humain
- Méthode lossy du [chroma subsampling](http://en.wikipedia.org/wiki/Chroma_subsampling)
- Symboles abscons (4:4:4, 4:2:2, 4:4:0...) connus aussi en vidéo

---
class: content-centered
# Discrete Cosine Transform

.column-left[
- Sur chaque composant (Y, Cb, Cr) de chaque MCU
- Dur à expliquer, partie la plus coûteuse en CPU
- Permet de décomposer sur une base de fréquences spatiales
]
.column-right[
![DCT](./img/300px_dctjpeg.png)
]

---
class: content-centered
# DCT côté maths
.column-right[
 ![DCT butterfly](./img/300_4x4_DCT_butterfly.png)
]

- Proche de la Discrete Fourier Transform (connu et optimisé)
- Bonne approximation de la [Karhunen-Loève Transform](http://en.wikipedia.org/wiki/Karhunen%E2%80%93Lo%C3%A8ve_theorem) ou ACP
- Pour les processus markoviens stationnaires d'ordre 1
  ![1st order Markov process](./img/400_1st_order_markov_process.png)
- Adapté aux variations faibles d'une MCU à l'autre

---
class: content-centered
# Propriétés de l'ACP 

.column-right[
![Energy compaction examples](./img/480-energy_compaction.png)
]
- Décomposition en variables aléatoires décorrélées
- Compaction d'énergie sur des signaux réels ➔ beaucoup d'information sur peu de coordonnées

[*source*](http://www.ee.columbia.edu/~xlx/ee4830/)

---
class: content-centered
# Quantification
.center[
![Erreur de quantification](./img/Quanterr.png)
]
- C'est juste un arrondi avec un pas (diviseur) donné
- L'approximation est irrécupérable (lossy)
- L'opération est appliquée pour chaque coefficient de la décomposition en DCT

---
class: content-centered
# Quantification adaptative
.center[
![Matrice de quantification](./img/quantization-matrix.png)
]


- La perception de l'œil humain privilégie les basses fréquences
- Une table de quantification par composant (Y, Cb, Cr) stockée dans le fichier
- Des tables déduites empiriquement sont fournies par le standard
- Le réglage qualité multiplie ces tables par une constante dépendant de l'encodeur

---
class: content-centered
# Techniques de compression lossless
.center[
![Entropy](./img/Entropy.png)
]
- Compression des zéros dans les hautes fréquences ([RLE](http://en.wikipedia.org/wiki/Run-length_encoding))
- [DPCM](http://en.wikipedia.org/wiki/Differential_pulse-code_modulation) sur la moyenne (DC) de chaque MCU
- Encodage de [Huffman](http://en.wikipedia.org/wiki/Huffman_coding) : optimal pour des données iid si le dictionnaire est déduit de l'image
- Des dictionnaires génériques sont fournis par le standard

---
class: content-centered
# Récapitulatif

.column-right[
  ![Diagramme de flux](./img/360px-JPEG_process.svg.png)
]
.column-left[
- Subsampling chroma
- Découpage en MCU (8x8)
- Passage en DCT
- Quantification
- Compression lossless
]
### Et la décompression c'est à l'envers !

---
class: content-centered
# JPEG progressif
.center[
![baseline vs. progressive](./img/01-02_baseline_vs_progressive.jpg)
([source](http://sixrevisions.com/graphics-design/jpeg-101-a-crash-course-guide-on-jpeg/))
]
- Juste une question d'ordre d'écriture des MCU dans le fichier
- Faites du progressif, sans hésitation !

---
class: content-centered
# Optimisations et tuning

### Plusieurs leviers en baseline
- Subsampling de CbCr
- Optimisation des tables de quantification
- Optimisation des dictionnaires de Huffman par rapport à l'image
- Progressif

### Leviers pas toujours supportés
- Compression arithmétique

---
class: content-centered
# Les opérations lossless
.center[
![Upside down picture](./img/old_bg-intro_upside_down.jpg)
]
- Rotation ➔ réarrangement des MCU
- Passage en progressif ➔ réarrangement de l'ordre des coefficients dans le fichier
- Flou ➔ mise à zéro des hautes fréquences
- Retaillage (multiple de 8x8) ➔ sélection de MCU
- Optimisation de la taille du fichier ➔ recompression avec table de Huffman optimisée

---
class: content-centered
# Les mauvaises idées
.column-left[
- Compresser à la qualité maximum ➔ lossy quand même (arrondis)
- Une taille pour toutes les images ➔  perte de qualité sur les images complexes
- Zipper un JPEG ➔  ajout d'un header: augmente la taille
- Compresser des images discontinues (texte) ➔ artefacts type [Gibbs](http://en.wikipedia.org/wiki/Gibbs_phenomenon)
]

.column-right[
![Hmmm...Nah](./img/hmm_nah.jpg)
]

---
class: full-page, slide-metadata
# Metadatas

---
class: content-centered
# Exiftool

```sh
$ exiftool picture.jpg
```

```sh
File Size                       : 494 kB
Image Size                      : 1920x1265
File Type                       : JPEG
MIME Type                       : image/jpeg
Exif Byte Order                 : Little-endian (Intel, II)
XMP Toolkit                     : Adobe XMP Core 5.3-c011 66.145661
Original Document ID            : xmp.did:999DEC701EB0E311801EFDD03E7C1154
Document ID                     : xmp.did:6C302F067BD111E4A0FFFBC0293A508B
Instance ID                     : xmp.iid:6C302F057BD111E4A0FFFBC0293A508B
Creator Tool                    : Adobe Photoshop CS6 (Windows)
Derived From Instance ID        : xmp.iid:25D692D731CB11E4BB03A7E70526CA15
Derived From Document ID        : xmp.did:25D692D831CB11E4BB03A7E70526CA15
DCT Encode Version              : 100
APP14 Flags 0                   : [14], Encoded with Blend=1 downsampling
APP14 Flags 1                   : (none)
Color Transform                 : YCbCr
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
```

---
class: content-centered
# Trois formats

- EXIF
- IPTC
- XMP

---
class: content-centered
# EXIF (1995)

- **EX**changeable **I**mage **F**ile
- Lors de la prise de photo
- Informations techniques
- 64kb max, no UTF-8, no timezone

---
class: content-centered
# IPTC (1990)

.center[
  ![IPTC](./img/iptc.gif)
]

 

- **I**nternational **P**ress **T**elecommunications **C**ouncil
- **I**nformation **I**nterchange **M**odel
- Texte, audio, video, images, etc
- Auteur, copyright, description, etc
- Deprecated

---
class: content-centered
# XMP (2001)

.center[
  ![XMP](./img/xmp.png)
]

 

- e**X**tensible **M**etadata **P**latform
- EXIF + IPTC en XML
- Extensible, UTF-8, i18n
- Support faible

---
class: content-centered
# Fragmentation

- Info dupliquée
- Reconciliation
- Support limité
- Tags propriétaires

---
class: content-centered
# Compression

- Sur le web, on supprime tout
- ... sauf l'auteur et la license

 

```shell
jpegtran -optimize -copy none file.jpg
```

---
class: full-page, slide-tools
# Outils

---
class: content-centered
# Outils

.center[
![Quality SSIM](./img/jpeg_quality_ssim.png)
]

- jpegoptim -m85 --strip-all
- cjpeg-dssim jpegoptim

---
class: content-centered

.thumbnails[
- ![Compression](./img/jpg-lossy-90.jpg) `-m90, 342Ko, 0.000528`
- ![Compression](./img/jpg-lossy-80.jpg) `-m80, 219Ko, 0.001259`
- ![Compression](./img/jpg-lossy-70.jpg) `-m70, 172Ko, 0.002076`
- ![Compression](./img/jpg-lossy-60.jpg) `-m60, 141Ko, 0.003032`
- ![Compression](./img/jpg-lossy-50.jpg) `-m50, 121Ko, 0.003785`
- ![Compression](./img/jpg-lossy-40.jpg) `-m40, 104Ko, 0.005191`
- ![Compression](./img/jpg-lossy-30.jpg) `-m30, 86Ko, 0.007569`
- ![Compression](./img/jpg-lossy-20.jpg) `-m20, 64Ko, 0.013038`
- ![Compression](./img/jpg-lossy-10.jpg) `-m10, 38Ko, 0.034276`
]

---
class: content-centered
# From the trenches

```shell
File Size              : 117 kB
Flash                  : Off, Did not fire
Focal Length           : 50.0 mm
Photoshop Thumbnail    : (Binary data 6959 bytes)
History Action         : derived, saved, saved...
History Software Agent : Adobe Photoshop Lightroom...
Thumbnail Image        : (Binary data 6959 bytes)
```

- Boulimie de metadata (11% du poids du fichier)

---
class: content-centered
# From the trenches

.center[
  ![Qualité trop élevée](./img/exemple_qualite98.png)
]
- Qualité trop haute (~98/100 sous Gimp)
- Taille écran 228x274px
- 117ko ➔ 23ko (qualité 80/100)

---
class: content-centered
# Le futur du JPEG ?
- Les successeurs : JPEG 2000 (so last century), wavelets compression, multiresolution tiling...
- Le JPEG est largement suffisant dans la plupart des cas de diffusion
- Le gain des méthodes plus avancées est trop faible (<15%, [source](http://www.elektronik.htw-aalen.de/packjpg/_notes/PCS2007_PJPG_paper_final.pdf))
- Ouvert et gratuit (les brevets sont valables 20 ans)

---
class: content-centered
# Good is often enough

- Côté vidéo le besoin a encouragé les avancées (H.264, H.265)
- Les principaux gains liés à notre perception ont majoritairement été exploités (audio, vidéo, image...)
- DVD Audio, HD audio, 4K sont souvent plutôt des besoins marketing que véritablement avérés

---
## Sources

http://www.impulseadventure.com/photo/jpeg-compression.html
http://fr.wikipedia.org/wiki/JPEG#D.C3.A9coupage_en_blocs
http://fr.wikipedia.org/wiki/Codage_de_Huffman
http://en.wikipedia.org/wiki/File:JPEG_process.svg
http://www.ams.org/samplings/feature-column/fcarc-image-compression
http://upload.wikimedia.org/wikipedia/commons/2/23/Dctjpeg.png
http://en.wikipedia.org/wiki/Quantization_(image_processing)
https://ece.uwaterloo.ca/~z70wang/research/ssim/

Bec Brown : http://finda.photo/image/6954
Taylor Swayze : http://finda.photo/image/8420
Ariana Prestes : http://finda.photo/image/4937
http://en.wikipedia.org/wiki/Autopsy#/media/File:Rembrandt_Harmensz._van_Rijn_007.jpg


