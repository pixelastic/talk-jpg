
Vous êtes tous au courant qu'il est indispensable de compresser ses images JPG
quand on réalise un site web. Soit vous utilisez un processus de build
automatisé qui va automatiquement vous les compresser, soit vous le faites
manuellement dans Photoshop avec la boite de dialogue qui va bien.

Celle-ci vous propose plus ou moins d'options pour tweaker votre compression.
Mais est-ce que que vous savez bien à quoi elles correspondent ?

[!Image d'un panel photoshop avec plusieurs options]

---
## Stockage naïf

Prenons une image JPG en exemple. Celle-ci fait 250x375 pixels. Ça fait très
exactement 93.750 pixels.

[!Image de test, avec ses dimensions]

Chaque pixel est un point sur l'écran, d'une certaine couleur. La manière la
plus simple de coder une couleur est de la décomposer en RGB (Red, Green,
Blue). Chaque composante RGB est un entier 0-255.

Pour coder un pixel, on a donc besoin de 3 entiers, donc 3 bytes.

Du coup, avec nos 93.750 pixels, notre image devrait peser normalement 281.250
bytes. Oui sauf que là je vois bien sur mon disque qu'elle n'en pèse que
32.414, soit environ 9 fois moins.

Comment ça se fait ?

---
## Différentes méthodes

Le format JPG utilise plusieurs techniques pour enregistrer vos pixels dans un
format plus léger que l'approche naïve qu'on vient de voir. Il va jouer à la
fois sur des systèmes d'encodage de l'information optimisés, sur les faiblesses
de l'œil humain et sur des algorithmes de compression classique.

Nous allons voir étape par étape comment fonctionne la compression, et ce sur
quoi chacune des options du panel de Photoshop influe.

La logique principale des différentes techniques de compression est d'essayer
de faire ressortir des redondances d'informations, afin de les supprimer.

---
## Découpons notre image en blocs 8x8

On commence par découper notre image en blocs de 8x8 pixels. Si jamais un bloc
dépasse des bords de la photo, on comble les pixels en trop par des pixels
blancs.

[!Image avec un quadrillage de 8x8]

On va ensuite effectuer une compression sur chacun des blocs, séparément, avant
de tous les regrouper ensemble. C'est cette première décomposition qui donne
l'effet de quadrillage sur les fichiers JPG qui sont trop compressés.

---
## Étudions un bloc

Quand on regarde un bloc de 8x8 individuellement, on se rends compte que les
couleurs de chaque pixel sont assez proches. Cette décomposition en bloc de 8x8
est spécialement adaptée au sujet principal de représentation des JPG : des
photos.

[!Exemples de plusieurs blocs pris dans l'image. Dominance de couleur]

Quand vous prenez une photo de n'importe quoi, vous avez beaucoup plus d'aplats
de couleurs que vous n'avez de démarcation nettes entre deux couleurs très
différentes.

Prenez un paysage, un portrait, une photo d'une rue, ou une photo macro d'un
papillon, vous verrez facilement que la majorité de l'image est constituée de
zones de couleurs. En découpant en bloc de 8x8, on a de fortes chances que les
couleurs de tous les pixels d'un bloc soient assez proches.

[!Coloration des différentes zones d'aplat de couleur d'une photo]

Les blocs d'une même teinte étant plus nombreux que les blocs de plusieurs
teintes, nos optimisations vont porter particulièrement sur ceux-ci.
Rapellez-vous, on souhaite mettre en avant les redondances, afin de les
supprimer.

---
## Conversion RGB -> YCbCr

Nous avons vu qu'une manière simple d'encoder la couleur d'un pixel est avec le
tryptique RGB. Il existe une autre notation, nommée YCbCr, qui plutot que de
définir une couleur comme un mélange Rouge/Vert/Bleu, la définit comme un
mélange Luminance/Chrominance Bleue/Chrominance Rouge.

Une simple équation permet de passer de l'un à l'autre. On ne perds pas de
donnée en passant en YCbCr, on représente juste la même couleur, mais selon
d'autres unitées. Cette transformation est donc complétement réversible.

En RGB, si vous prenez deux couleurs assez proches, avec l'une plus foncée que
l'autre, alors vos valeurs de R, G et B vont toutes les trois bouger. Par
contre en YCbCr, les valeurs de Cb/Cr vont très très peu bouger alors que juste
la valeur de Y va osciller.

[!Exemple de plusieurs pixels proches avec leurs équivalents RGB et YCbCr]

Encore une fois, qu'est-ce qu'on souhaite mettre en avant ? Les redondances. Et
là on voit que d'un pixel au suivant les valeurs de Cb/Cr restent les mêmes. Il
y a donc moyen d'optimiser à cet endroit.

---
## Subsampling

Si on ne regarde que les valeurs Cb (ou Cr) pour notre bloc, on voit que des
pixels contigus partagent souvent la même valeur, ou des valeurs proches. On
peut donc les regrouper ensemble pour stocker moins de valeurs.

[!Image avec notre bloc en Cb et les couleurs identiques, la valeur Cb en
surimpression et encadrement de groupes de pixels ensemble + Version modifiée
avec moins de pixels à garder]

C'est la première compression avec perte de donnée qu'on execute. On peut
choisir de regrouper les pixels par groupe de 2 (horizontalement, ou
verticalement) ou par groupe de 4 ou plus. Plus on regroupe de pixels ensemble,
plus on perds d'information.

La syntaxe pour choisir le type de subsampling est infame, et c'est celle que
vous voyez dans le menu de Photoshop

[!Screenshot de Photoshop avec le subsampling à choisir]

Ok, on a fait tout ce qu'on pouvait faire sur les pixels de manière
individuels. Maintenant on va s'attaquer à notre bloc dans son ensemble.

---
## DCT 

Maintenant, on va s'attaquer à la partie la plus complexe de la compression.
C'est à la fois la plus complexe à expliquer, et la plus couteuse à processer
par la machine.

On va recomposer notre bloc comme une somme de vecteurs coefficienté, où chaque
vecteur est une des cases de cette image :

[!Image Dctjpeg.png]

On va donc recomposer notre Y, notre Cb et notre Cr comme une somme de ces
vecteurs, chacun associé d'un coefficient. On numérote les vecteurs depuis
celui en haut à gauche, jusque celui en bas à droite en suivant un zigzag.

Le Y de notre bloc pourra donc s'exprimer `x1 * V1 + x2 * V2 + ... + x64 * V64`
. Les vecteurs `V1-V64` étant déjà connus, il nous suffit d'enregistrer la
suite de coefficients `x1-x64` pour le Y, le Cb et le Cr. Ainsi, une suite
donnera toujours le même bloc, et si on change la suite, on change le rendu du
bloc.

Ici encore, on stocke la même information (un bloc) mais dans une unité
différente. On est passé de 64 YCbCr à une suite de 64 coefficients. La
transformation est presque complétement reversible. En effet, elle utilise
sous le capot ce qu'on appelle une DCT (Discrete Cosine Transform), et on
stocke donc les coefficients comme des `cosinus`, ce qui peut induire quelques
erreurs d'arrondi. Ceux-ci restent néanmoins négligeables.

---
## Quantification

On a donc réussi à représenter un bloc en 3 suites de 64 entiers (un pour Y, un
pour Cb et un pour Cr). Et ces entiers représentent les coefficients des
vecteurs qui servent à tracer l'image. Les vecteurs les plus forts sont placés
en premiers, et plus on avance plus les vecteurs sont fins.

On peut imaginer cette manière de représenter le bloc comme celle d'un peintre
qui commencerait par donner les gros coups de pinceaux génériques, faire les
aplats des grosses zones de couleurs, puis ajouterait les détails petit
à petit.

[!Exemple d'image qui apparait en progressif, avec d'abord les vecteurs
importants, puis les détails, etc]

Ici, on va pouvoir faire jouer une autre méthode de compression, nommée
Quantification (Quantization en anglais). On va simplifier la liste des
coefficients, par exemple en mettant complétement à zero les coefficients des
derniers vecteurs de la liste. Plus on avance dans la liste, moins les vecteurs
sont forts.

C'est sans doute la partie la moins scientifique de la compression. C'est
complétement empirique. On a en fait fait passer des tests à des centaines de
personnes, à qui on a montré des milliers des photos, plus ou moins compressées
et on a noté les seuils où les gens percevaient des différences. On a ensuite
agrégé ces résultats et on a sorti ce qu'on appelle des tables de Quantization.

On utilise ces tables dans la compression pour supprimer certains coefficients
quand on pense que leur absence ne sera pas perceptible par l'œil humain.

La spec propose un exemple de table de quantification mais chaque
implémentation, chaque outil, chaque appareil photo utilise à priori une table
différente. Cette table n'est absolument pas standard et est stockée
directement dans l'image JPG finale.

---
## Compression de zeros

On se retrouve donc avec notre suite de coefficient simplifiée, qui se retrouve
maintenant avec des coefficients qui ont été remis à zero. Plus la compression
est forte, plus on va se retrouver avec beaucoup de zero.

Ici, plutot que de noter 36 fois zero à la suite, on va simplement encoder le
nombre de zero, ce qui prends moins de place.

---
## Fin du bloc

Et voila, on en a fini avec notre bloc.

On est passé de 64 pixels en RGB à 3 suites de coefficients (Y,Cb,Cr), en
passant par plusieurs processus de compression, lossy et lossles, en route.

---
## Autres blocs

Maintenant, on va faire la même chose pour les autres blocs de l'image. On va
juste rajouter un autre petite astuce pour grapiller encore quelques octets.

Deux blocs cote à cote sur la même ligne risquent d'avoir sensiblement la même
teinte. Après tout, 8 pixels, ça fait pas beaucoup. Du coup, plutot que de
coder le coefficient du premier vecteur (le plus fort, celui qui donne la
teinte principale), on va le coder en delta du coefficient du premier vecteur
du bloc précédent.

---
## Huffman encoding

Une fois qu'on a effectué ce travail sur tous les blocs de notre image, il nous
reste une dernière optimisation pour gagner encore un peu de place.

On a maintenant codé notre image comme une suite d'entiers représentant les
différents coefficients de tout nos blocs. On va appliquer à cela une simple
compression par dictionnaire, ou compression de Huffman.

Encore une fois, on cherche les redondances, pour les supprimer.

On va retrouver les patterns qui se répètent souvent, en les triant par
fréquence d'apparition. Et on va attribuer un symbole à chaque pattern, en
utilisant les symboles les plus courts pour les patterns les plus fréquents. Si
un "mot" est présent très souvent, on l'encode finalement simplement en une
lettre et on stocke le dictionnaire qui permet de faire la traduction dans
l'image.

---
## Conclusion

Et c'est fini ! On a donc opéré plusieurs transformations sur notre data
initiale. Des transformation sans perte de donnée, mais qui nous donnaient un
format qui mettait en valeur les redondances que l'on souhaitait faire
disparaitre.

On a aussi opéré des modifications spécifiquement adaptées à l'œil humain qui
va consommer ces images, en supprimant les informations qui ne seront de toutes
façons pas perçues.

---
## Décompression

Maintenant, quand notre navigateur souhaite décompresser une image JPG pour
l'afficher, il joue simplement tout le processus à l'envers.

D'abord il regarde le contenu et remplace les symboles par leurs équivalents
dans le dictionnaire. Il prends ensuite les suites de coefficient 64 par 64
pour former des blocs complets. Il applique ces coefficients aux vecteurs pour
repeindre l'image, couche par couche. Il convertit ensuite chacun des pixels en
YCbCr vers du RGB qu'il affiche à l'écran.

Pour le JPG progressif, l'encodage est exactement le même, il y a juste un flag
ajouté qui indique au lecteur qu'il doit lire le JPG en mode progressif. C'est
à dire qu'au lieu de lire les 64 vecteurs d'un bloc avant de passer au suivant,
il va lire uniquement X vecteurs (ceux des plus gros traits), qu'il va afficher
et passer au bloc suivant. Puis il repasse sur chaque bloc pour afficher les
X vecteurs suivants, et ainsi de suite. Cela donne une affichage progressif,
mais coute plus de tours de CPU (négligeable aujourd'hui).

---
## Sources

http://www.impulseadventure.com/photo/jpeg-compression.html
http://fr.wikipedia.org/wiki/JPEG#D.C3.A9coupage_en_blocs
http://fr.wikipedia.org/wiki/Codage_de_Huffman
http://www.ams.org/samplings/feature-column/fcarc-image-compression
http://upload.wikimedia.org/wikipedia/commons/2/23/Dctjpeg.png

