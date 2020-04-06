# Normale à une surface et plusieurs objets

Premièrement, définissons ce qu'est une normale, puis utilisons ce concept pour nuancer/ombrager les objets.
Une normale est un vecteur qui est perpendiculaire à la surface au point d'intersection. Il y a deux façons de concevoir des normales.
La première est de faire une normale à partir d'un vecteur de longueur unitaire. C'est pratique pour nuancer/ombrager donc je dirai que c'est une bonne solution, mais comme dit précédemment, je ne vais pas le faire dans le code. Ca peut entrainer des bugs difficiles à débusquer.

Pour une sphère, la normale extérieure est dans la direction du point d'impact, auquel on aurait soustrait la position du centre :

![Normale à une sphère](img/normale_sphere.jpg)

Si on prend la terre pour exemple, ça implique donc que le vecteur qui part du centre de la terre au point jusqu'à vous pointe vers le haut.

Implémentons ça et voyons le résultat.

Pour l'instant, nous n'avons aucune lumière donc nous allons devoir visualiser les normales en utilisant un jeu de couleurs. 
Une astuce utilisée fréquemment pour la visualisation de normales (car facile et intuitif d'assumer que \\(N\\) est un vecteur de longueur unitaire, donc chacun de ses composants est entre -1 et 1) est d'associer à chaque composant de l'intervalle 0 à 1, la valeur x/y/z en r/g/b. 
Pour une normale, nous avons besoin du point d'intersection, pas seulement savoir si le rayon a touché ou non la sphère. Nous allons assumer que la normale est désignée par le point d'intersection le plus proche (le plus petit \\(t\\) possible).

Notre code modifié qui nous permet de calculer et visualiser \\(N\\) est le suivant :
```cpp
double hit_sphere(const vec3& center, double radius, const ray& r) {
    vec3 oc = r.origin() - center;
    auto a = dot(r.direction(), r.direction());
    auto b = 2.0 * dot(oc, r.direction());
    auto c = dot(oc, oc) - radius*radius;
    auto discriminant = b*b - 4*a*c;
    if (discriminant < 0) {
        return -1.0;
    } else {
        return (-b - sqrt(discriminant) ) / (2.0*a);
    }
}

vec3 ray_color(const ray& r) {
    auto t = hit_sphere(vec3(0,0,-1), 0.5, r);
    if (t > 0.0) {
        vec3 N = unit_vector(r.at(t) - vec3(0,0,-1));
        return 0.5*vec3(N.x()+1, N.y()+1, N.z()+1);
    }
    vec3 unit_direction = unit_vector(r.direction());
    t = 0.5*(unit_direction.y() + 1.0);
    return (1.0-t)*vec3(1.0, 1.0, 1.0) + t*vec3(0.5, 0.7, 1.0);
}
```

Ce qui nous donne l'image suivante :

![Visualisation normales d'une sphère](img/visualisation_normales_sphere.png)

Chaque pixel de la sphère est coloré selon son vecteur normal. 

Revisitons l'équation de l'intersection d'un rayon et d'une sphère, précédemment nous avions :

```cpp
vec3 oc = r.origin() - center;
auto a = dot(r.direction(), r.direction());
auto b = 2.0 * dot(oc, r.direction());
auto c = dot(oc, oc) - radius*radius;
auto discriminant = b*b - 4*a*c;
```

Déjà, souvenez-vous qu'un vecteur dont on fait le produit scalaire (*dot product*) avec lui-même est égal au carré de la longueur de ce vecteur.
Deuxièmement, notez comment l'équation suivante a un facteur 2 pour \\(b\\). Observez ce qu'il se passe à l'équation quadratique si \\(b=2h\\) :

\\[ -b \pm \sqrt{b^2 - 4ac} \over{2a}\\]

\\[ = \frac{-2h \pm \sqrt{(2h)^2 - 4ac}}{2a}\\]

\\[ = \frac{-2h \pm 2\sqrt{h^2 - ac}}{2a}\\]

\\[ = \frac{-h \pm \sqrt{h^2 - ac}}{a}\\]

En utilisant ces observations, nous pouvons simplifier le code par ça :

```cpp
vec3 oc = r.origin() - center;
auto a = r.direction().length_squared();
auto half_b = dot(oc, r.direction());
auto c = oc.length_squared() - radius*radius;
auto discriminant = half_b*half_b - a*c;

if (discriminant < 0) {
    return -1.0;
} else {
    return (-half_b - sqrt(discriminant) ) / a;
}
```

Maintenant, comment gérer plusieurs sphères ? Il serait assez tentant de stocker un tableau de sphères, une solution très propre serait d'avoir une classe abtraite (que l'on ne peut pas instancier) pour chaque objet qu'un rayon pourrait traverser. De cette manière, une sphère, ou une liste de sphères pourraient être traversées. Le nom à donner à cette classe est un vrai dilemme : si on l'appelle objet ce n'est pas un bon nom puisqu'on fait de la programmation orientée objet par exemple. "Surface" est souvent utilisé, sauf que parfois nous voulons gérer des volumes... "hittable" (de l'anglais 'hit', touché, 'hittable', que l'on peut toucher) met en avant la principale fonction membre : 'hit'. Je n'aime aucun de ces choix mais partons sur 'hittable'.

Cette classe *hittable* 