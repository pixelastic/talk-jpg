Le but est de coder les données d'une manière qui montre une redondance
apparente, de manière à pouvoir la supprimer.

Exemple, image 250x375. Donc, 93,750 pixels.
Chaque pixel est mélange r, g, b, entiers 0-255.
Un entier = 1 byte donc une pixel = 3 bytes
Donc, image devrait peser 3 x 93,750 = 281.250 bytes.
Mais ne pèse que 32,414, soit environ 9x moins.

## Blocsk 8 x 8

Image découpée en blocks de 8x8 pixels.
Chaque bloc encodé séparement.

## Un bloc de 8x8

Sur un même bloc, on voit que les couleurs sont assez similaires sur les
pixels.
Rappel, le but est de montrer des redondances, pour les supprimer.

## YCbCr

On va transformer le codage rgb en un codage YCbCr. Plus un mélange de Red,
Green, Blue, mais un mélange de Luminance (Y), et Chrominance Blue, Chrominance
Red.

Juste une matrice de transformation. Même information, exposée dans une autre
unité. Complétement reversible.

Autant en RGB, les trois valeurs changent beaucoup pour chaque pixel, autant en
YCbCr, c'est le Y qui fluctue beaucoup alors que les deux autres restent
globalement stables.

## Subsampling

Là, on peut commencer à compresser. Comme Cr et Cb ne changent pas beaucoup, on
peut grouper les pixels 2 par 2, 4 par 4, dans diverses combinaisons. Ça nous
fait moins d'infos à stocker, on stock juste que le pixel B est égal au pixel
A. 

Syntaxe barbare, pour indiquer quels pixels regrouper ensemble.

Première compression lossy.

## DCT : Discrete Cosine Transform

On peut imaginer un bloc de 8 par 8 comme une fonction, qui prends un x et un
y en entrée, et retourne le YCbCr du pixel qui matche.

Et il existe une manière mathématique de rendre ça. On prends l'image
`Dctjpeg.png`, où chaque carré rouge est un "vecteur". On peut alors recoder
notre block 8x8 comme une somme de ces vecteurs, pondérés.

x1V1 + x2V2 + X3V3 + ... X64V64

On lit les vecteurs non pas ligne par ligne, mais en zigzag, d'en haut à gauche
jusqu'en bas à droit.

On peut donc écrire notre bloc comme une suite de 64 entiers, représentant les
coefficients des vecteurs. Deux blocs identiques auront la même suite, si la
suite est différente, les blocs sont différents.

## Quantization

Les premiers vecteurs de la liste sont les plus forts, qui font les gros traits
de notre bloc, les derniers sont les plus faibles, qui font les détails.

On va donc simplifier notre liste de coefficients pour supprimer ceux qu'on
juge ne pas apporter de valeur (typiquement, les derniers de la liste).

On a fait des études empiriques en demandant à des gens de comparer des photos,
et on a tiré des informations sur les éléments qui ne sont pas perceptibles
à l'oeil humain.

Du coup, on va supprimer certains coefficients s'ils sont en dessous d'une
certaine valeur. Ces tables de quantification sont différentes selon les
programmes, appareils photos, etc et sont encodées dans l'image finale (pour la
décompression).

## Compression de zeros

Pour un bloc, ona  donc une liste 64 coefficients, à la suite. Donc on a passé
à zero plus ou moins d'éléments. Le but étant d'en passer un max à zero, on se
retrouve avec une grande liste de 0.

On va pas noter plein de zeros, on va juste coder le nombre de zero, ce qui le
rends encore plus petit.

## Delta des blocks bout à bout

Maintenant qu'on sait coder un bloc de manière super light, on fait la même
chose pour tous les blocs.

Sauf que le premier coefficient de chaque bloc donne le ton général du block et
un bloc par rapport au bloc d'à coté ne change pas forcément beaucoup. Du coup,
on code le premier coefficient de chaque bloc comme un delta par rapport au
premier coefficient du bloc précédent, histoire de gagner encore un peu de
data.

## Huffman encoding

Finalement, on mets donc tout nos blocs bout à bout. Chaque bloc étant codé en
une suite de coefficients et de nombre de 0.

On applique ensuite une simple compression de Huffman. On regarde la totalité
de notre data, et on essaie de retrouver des patterns qui se répetent plusieur
fois, en commencant par les patterns les plus long.

On associe un symbole à chaque pattern, avec les symboles les plus courts pour
les patterns les plus fréquents. C'est ce qu'on appelle une compression par
dictionnaire.

## Conclusion

On se retrouve donc avec une data qui est passée par plusieurs étapes, qu'on
peut executer dans l'autre sens. Certains passages sont lossy, d'autres
lossless.

## Décompression

On va regarder le dictionnaire et transformer les symboles selon le
dictionnaire

On va lire les 64 premiers éléments (représente un bloc). On compte les X zeros
comme X éléments.

On utilise ces éléments comme des coefficients sur les vecteurs, lus en
diagonale.

Ça nous donne une image de 64 pixels exprimé en YCbCr

On convertit chaque pixel en RGB, et on affiche le bloc à l'écran.

On passe au bloc suivant.

## Progressive

En progressive on fait presque pareil.

On décode le dictionnaire, puis on choppe les différents blocs de 64 éléments.

Sur chacun des bloc on décode uniquement les X premiers vecteurs en YCbCr puis
RGB

Puis on décode les X vecteurs suivants, et ainsi de suite.

On passe donc plus de fois sur chaque élément, mais on affiche progressivement
les détails, plutot que d'afficher ligne par ligne.


