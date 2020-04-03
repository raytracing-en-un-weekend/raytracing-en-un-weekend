# La classe `vec3`

Quasiment tous les programmes graphiques utilisent une ou plusieurs classes pour stocker des vecteurs pour la géométrie et des couleurs. Dans beaucoup de ces systèmes, il s'agit de vecteur 4D (3D + une coordonnée homogène pour la géométrie ou + une composante 'alpha' de transparence pour les couleurs).

Pour notre besoin, 3 coordonnées suffisent. Nous allons donc utiliser la classe `vec3` pour les couleurs, les positions, les directions, les décalages, etc. Certaines personnes n'aiment pas ça parce que cette méthode ne vous empêche pas de faire des choses stupides, comme ajouter une couleur et une position ensemble. C'est un bon point, mais nous allons choisir la voie du plus simple lorsque ce n'est pas expressement faux.

Voici une implémentation possible de cette classe `vec3` :

```cpp
#include <iostream>

class vec3 {
    public:
        vec3() : e{0,0,0} {}
        vec3(double e0, double e1, double e2) : e{e0, e1, e2} {}

        double x() const { return e[0]; }
        double y() const { return e[1]; }
        double z() const { return e[2]; }

        vec3 operator-() const { return vec3(-e[0], -e[1], -e[2]); }
        double operator[](int i) const { return e[i]; }
        double& operator[](int i) { return e[i]; }

        vec3& operator+=(const vec3 &v) {
            e[0] += v.e[0];
            e[1] += v.e[1];
            e[2] += v.e[2];
            return *this;
        }

        vec3& operator*=(const double t) {
            e[0] *= t;
            e[1] *= t;
            e[2] *= t;
            return *this;
        }

        vec3& operator/=(const double t) {
            return *this *= 1/t;
        }

        double length() const {
            return sqrt(length_squared());
        }

        double length_squared() const {
            return e[0]*e[0] + e[1]*e[1] + e[2]*e[2];
        }

        void write_color(std::ostream &out) {
            // Write the translated [0,255] value of each color component.
            out << static_cast<int>(255.999 * e[0]) << ' '
                << static_cast<int>(255.999 * e[1]) << ' '
                << static_cast<int>(255.999 * e[2]) << '\n';
        }

    public:
        double e[3];
};
```

Nous utilisons des `double` ici, mais certains ray tracers fonctionnent avec des `float`. C'est selon votre propre goût. La seconde partie du fichier header contient des fonctions utilitaires relatives aux vecteurs :
```cpp
// vec3 Utility Functions

inline std::ostream& operator<<(std::ostream &out, const vec3 &v) {
    return out << v.e[0] << ' ' << v.e[1] << ' ' << v.e[2];
}

inline vec3 operator+(const vec3 &u, const vec3 &v) {
    return vec3(u.e[0] + v.e[0], u.e[1] + v.e[1], u.e[2] + v.e[2]);
}

inline vec3 operator-(const vec3 &u, const vec3 &v) {
    return vec3(u.e[0] - v.e[0], u.e[1] - v.e[1], u.e[2] - v.e[2]);
}

inline vec3 operator*(const vec3 &u, const vec3 &v) {
    return vec3(u.e[0] * v.e[0], u.e[1] * v.e[1], u.e[2] * v.e[2]);
}

inline vec3 operator*(double t, const vec3 &v) {
    return vec3(t*v.e[0], t*v.e[1], t*v.e[2]);
}

inline vec3 operator*(const vec3 &v, double t) {
    return t * v;
}

inline vec3 operator/(vec3 v, double t) {
    return (1/t) * v;
}

inline double dot(const vec3 &u, const vec3 &v) {
    return u.e[0] * v.e[0]
         + u.e[1] * v.e[1]
         + u.e[2] * v.e[2];
}

inline vec3 cross(const vec3 &u, const vec3 &v) {
    return vec3(u.e[1] * v.e[2] - u.e[2] * v.e[1],
                u.e[2] * v.e[0] - u.e[0] * v.e[2],
                u.e[0] * v.e[1] - u.e[1] * v.e[0]);
}

inline vec3 unit_vector(vec3 v) {
    return v / v.length();
}
```

Maintenant, nous pouvons changer notre fonction `main` pour l'utiliser :

```cpp
#include "vec3.h"

#include <iostream>

int main() {
    const int image_width = 200;
    const int image_height = 100;

    std::cout << "P3\n" << image_width << ' ' << image_height << "\n255\n";

    for (int j = image_height-1; j >= 0; --j) {
        std::cerr << "\rScanlines remaining: " << j << ' ' << std::flush;
        for (int i = 0; i < image_width; ++i) {
            vec3 color(double(i)/image_width, double(j)/image_height, 0.2);
            color.write_color(std::cout);
        }
    }

    std::cerr << "\nDone.\n";
}
```