# Des rayons, une caméra et un arrière-plan

Le point commun de tous les ray tracers, c'est une classe permettant de modélisation un rayon de lumière (ray) et un calcul déterminant la couleur vue le long de ce rayon.

Nous pouvons modéliser un rayon comme une fonction :

\\[ p(t)=a+t \vec{b} \\]

Ici \\( p \\) est une position 3D le long d'une ligne 3D. \\( a \\) est l'origine du rayon et \\( \vec{b} \\) est la direction du rayon. Le rayon a également un paramètre \\( t \\), un nombre réel (`double` dans le code).
Fournir un \\( t \\) différent et \\( p(t) \\) déplacera le point le long du rayon. Ajoutez un \\( t \\) négatif et on peut se déplacer n'importe où le long de cette ligne 3D. Pour un \\( t \\) positif, on peut se déplacer seulement sur la partie en face de \\( a \\), et c'est ce qu'on appelle souvent une demi-ligne ou rayon.

![Interpolation linéaire](img/interpolation_lineaire.jpg)

La fonction \\( p(t) \\) peut se modéliser comme une classe `ray` pour laquelle on implémente une fonction `ray::at(t)` :

```cpp
#ifndef RAY_H
#define RAY_H

#include "vec3.h"

class ray {
    public:
        ray() {}
        ray(const vec3& origin, const vec3& direction)
            : orig(origin), dir(direction)
        {}

        vec3 origin() const    { return orig; }
        vec3 direction() const { return dir; }

        vec3 at(double t) const {
            return orig + t*dir;
        }

    public:
        vec3 orig;
        vec3 dir;
};

#endif
```

Maintenant, nous sommes fins prêts à implémenter un ray tracer. A son coeur, le ray tracer envoie des rayons pour chaque pixel de l'image et calcule la couleur perçue dans la direction de ces rayons. Les étapes sont donc au nombre de 3 :
1. calculer le rayon depuis l'oeil (la caméra) au pixel
2. déterminer quels objets le rayon va traverser (calcul d'intersection)
3. calculer une couleur sur ce point d'intersection.

Quand je développe un ray tracer, je commence par créer une caméra simple pour pouvoir visualiser et débugger mon code. Je fais également une petite fonction `color(ray)` qui retourne la couleur de l'arrière-plan (un simple dégradé).

Je tombe souvent sur des problèmes quand j'utilise des images carrées pour déboguer car je transpose \\(x\\) et \\(y\\) trop souvent, donc je fais des images en \\(200\times100\\). 

Je mets "l'oeil" (le centre de la caméra) aux coordonnées \\((0,0,0)\\). Je place l'axe \\(y\\) qui pointe vers le haut et l'axe \\(x\\) dirigé vers la droite. Pour respecter la convention d'un système de coordonées 'main droite', je place un axe \\(z\\) négatif vers l'écran. Je balaye l'écran depuis le coin inférieur gauche et j'utilise 2 vecteurs de décalage le long des côtés de l'écran pour déplacer le bout du rayon sur la surface de l'écran. Notez que le vecteur de direction n'est pas un vecteur de longueur normalisé (longueur unitaire) parce que je pense que ça rend le code plus simple et légèrement plus rapide.


![Géométrie de la caméra](img/geometrie_camera.jpg)

Dessous en code, le rayon \\(r\\) est approximativement dirigé vers les centres des pixels (je ne me focalise pas sur l'exatitude pour l'instant car nous ajouterons l'antialiasing plus tard).

```cpp
#include "ray.h"

#include <iostream>

vec3 ray_color(const ray& r) {
    vec3 unit_direction = unit_vector(r.direction());
    auto t = 0.5*(unit_direction.y() + 1.0);
    return (1.0-t)*vec3(1.0, 1.0, 1.0) + t*vec3(0.5, 0.7, 1.0);
}

int main() {
    const int image_width = 200;
    const int image_height = 100;

    std::cout << "P3\n" << image_width << " " << image_height << "\n255\n";
    vec3 lower_left_corner(-2.0, -1.0, -1.0);
    vec3 horizontal(4.0, 0.0, 0.0);
    vec3 vertical(0.0, 2.0, 0.0);
    vec3 origin(0.0, 0.0, 0.0);
    for (int j = image_height-1; j >= 0; --j) {
        std::cerr << "\rScanlines remaining: " << j << ' ' << std::flush;
        for (int i = 0; i < image_width; ++i) {
            auto u = double(i) / image_width;
            auto v = double(j) / image_height;
            ray r(origin, lower_left_corner + u*horizontal + v*vertical);
            vec3 color = ray_color(r);
            color.write_color(std::cout);
        }
    }

    std::cerr << "\nDone.\n";
}
```

La fonction `ray_color(ray)` mélange linéairement du blanc et du bleu selon la hauteur de la coordonnée \\(y\\) *après* avoir redimensionner la direction du rayon en vecteur unitaire (normalisé) (donc \\(-1.0 < y < 1.0\\)).
Parce qu'on est en train de regarder la hauteur \\(y\\) après avoir normalisé le vecteur, vous remarquerez un dégradé horizontal vers la couleur en plus du dégradé vertical.

Ensuite, je redimensionne grâce une astuce classique de manière à ce que \\(0.0 \leq t \leq 1.0\\). Quand \\(t=1.0\\) je veux du bleu. Quand \\(t=0.0\\), je veux du blanc. Et entre, je veux un dégradé. Cela forme un dégradé linéaire, ou interpolation linéaire, ou 'lerp' (de *linear interpolation*). Une interpolation linéaire est toujours de la forme :

\\[ \text{blendedValue} = (1-t) \times \text{startValue} + t \times \text{endValue} \\]

avec \\(t\\) allant de 0 à 1. Dans notre cas, le code suivant produit :

![Dégradé du bleu au blanc qui dépend du rayon de coordonnées Y](img/degrade_bleu_blanc.png)