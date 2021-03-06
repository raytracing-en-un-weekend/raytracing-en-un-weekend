# Créer une image

A chaque fois que l'on démarre la création d'un moteur de rendu, nous avons besoin de voir une image. La façon la plus simple est d'écrire l'image dans un fichier. Le problème est qu'il existe un innombrable format d'images et que beaucoup d'entre eux sont complexes. Je commence toujours par des fichiers textes ppm. Wikipedia en fait une belle description :

![Format PPM](img/format_ppm.png)

Ecrivons un peu de C++ pour sortir un fichier similaire :
```cpp
#include <iostream>

int main() {
    const int image_width = 200;
    const int image_height = 100;

    std::cout << "P3\n" << image_width << ' ' << image_height << "\n255\n";

    for (int j = image_height-1; j >= 0; --j) {
        for (int i = 0; i < image_width; ++i) {
            auto r = double(i) / image_width;
            auto g = double(j) / image_height;
            auto b = 0.2;
            int ir = static_cast<int>(255.999 * r);
            int ig = static_cast<int>(255.999 * g);
            int ib = static_cast<int>(255.999 * b);
            std::cout << ir << ' ' << ig << ' ' << ib << '\n';
        }
    }
}
```

Il y a plusieurs choses à noter dans ce code :
1. Les pixels sont écrits en lignes, de la gauche vers la droite.
2. Les lignes sont écrites du haut vers le bas.
3. Par convention, chaque composant RVB(rouge/vert/bleu ou RGB en anglais) a une valeur allant de 0.0 à 1.0. Nous assouplirons cette contrainte plus tard quand nous utiliserons une plage HDR (high dynamic range), sachant qu'avant de sortir une image HDR nous allons contraindre les valeurs dans l'intervale de zéro à un, donc ce code ne changera pas.
4. La composante rouge va du noir au maximum de gauche à droite et le vert va du noir en bas, jusqu'au maximum en haut. Le rouge et le vert ensemble forment du jaune, donc on doit s'attendre à ce que coin supérieur droit soit jaune.

Parce que le programme n'écrit pas directement dans un fichier mais dans sa sortie standard, nous avons besoin de rédiger celle-ci dans un fichier.
Généralement, c'est fait depuis la ligne de commande en utilisant le chevron '>' pour rediriger la sortie vers un fichier :

`
build\Release\inOneWeekend.exe > image.ppm
`

sous Windows, et pour Mac & Linux :

`
build/inOneWeekend > image.ppm
`

Nous pouvons ouvrir le fichier ainsi créé (avec 'ToyViewer' sur mon Mac par exemple) et nous obtenons le résultat suivant :

![Format PPM](img/first_ppm.png)

Hooray! C'est un peu le 'hello world' en graphique. Si votre image ne ressemble pas à ça, ouvrez le fichier avec un éditeur de texte et regardez à quoi il ressemble. Il devrait commencer par :
```
P3
200 100
255
0 253 51
1 253 51
2 253 51
3 253 51
5 253 51
6 253 51
7 253 51
8 253 51
```

Si ce n'est pas le cas, vous avez peut-être des nouvelles lignes ou quelque chose comme ça. 

Si vous voulez produire des formats autres que le PPM, je suis un fan de `stb_image.h` disponible sur Github.

## Ajouter un indicateur de progression

Avant de continuer, ajoutons un indicateur de progression à notre sortie. C'est une façon pratique de suivre la progression d'un long rendu, et aussi une manière d'identifier un lancement bloqué à cause d'une boucle infinie ou d'un autre problème.

Notre programme sort l'image via la sortie standard (`std::cout`), donc nous laissons ça et on affiche la progression plutôt via la sortie d'erreur (`std::cerr`) :

```cpp
for (int j = image_height-1; j >= 0; --j) {
    std::cerr << "\rScanlines remaining: " << j << ' ' << std::flush;
    for (int i = 0; i < image_width; ++i) {
        auto r = double(i) / image_width;
        auto g = double(j) / image_height;
        auto b = 0.2;
        int ir = static_cast<int>(255.999 * r);
        int ig = static_cast<int>(255.999 * g);
        int ib = static_cast<int>(255.999 * b);
        std::cout << ir << ' ' << ig << ' ' << ib << '\n';
    }
}
std::cerr << "\nDone.\n";
```