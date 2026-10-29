# 1. EN-TÊTE
TD_matrices_C

Membres (ID Github): MERCIER Théo (tmercier370) et KOWALSKI Valentin (val-kow)

3A STI FISE - TD 2

Date de Remise : ../../2026 //A COMPLETER !


# 2. INTRODUCTION
On cherche à réaliser des allocations, sauvegardes, relectures, opérations et affichages de matrices en langage C 
à l'aide de plusieurs modules.

Les objectifs du TD sont :
    - savoir compiler les différents modules et les assembler avec l'édition de lien
    - éviter les fuites de mémoire
    - Configurer les sorties d'erreurs
    - Ecrire un rapport de chaque exercice réalisé
    - Se familiariser avec l'outil GitHub

Ce qui fonctionne (ou non) à la remise :
    - ... (OK)
    - ... (ECHEC)


# 3. CONCEPTION


# 4. REALISATION
## Exercice 1 :
(1.a) : /* Cas où la matrice a une erreur de dimension ou de coef trop grand/petit */
```c
    case MAT_ERR_DIMENSIONS:   return "dimensions incompatibles";
    case MAT_ERR_DEPASSEMENT:  return "résultat hors de l'intervalle des int";
```

(1.b) : /* si la matrice m existe et tous ses coefs existent et si le nbre de lignes et colonnes est compris entre 1 et MAT_DIM_MAX alors retourne 1 (succès) sinon retourne 0 (ECHEC) */
```c
    if (m != NULL && m->coef != NULL && m->nb_lignes >= 1 && m->nb_lignes <= MAT_DIM_MAX && m->nb_colonnes >= 1 && m->nb_colonnes <= MAT_DIM_MAX) {
        return 1; 
    } 
    else {
        return 0;
    }
// Lignes supprimées
```

(1.c) : /* si un des arguments est trop grand/petit, retourne l'erreur d'argument */
```c
    if (nb_lignes < 1 || nb_lignes > MAT_DIM_MAX || nb_colonnes < 1 || nb_colonnes > MAT_DIM_MAX) {
        return MAT_ERR_ARGUMENT;
    }
```

(1.d) : /* Passage d'arguments de mat_calloc : (size_t) convertit en type attendu par calloc et sizeof *coef[i] correspond à la taille d'un int */
```c
        coef[i] = mat_calloc((size_t)nb_colonnes, sizeof *coef[i]);    // nb_colonnes entiers
```
Schéma des variables et ordre de libération des pointeurs :
```text
coef : Ox1000
coef[0] : Ox2000
coef[1] : Ox2008
coef[2] : NULL
-> k = 1 : free(coef[1]) // libère de la mémoire sur TAS/HEAP
-> k = 0 : free(coef[0])
coef : Ox1000
coef[0] : NULL 
coef[1] : NULL
coef[2] : NULL
-> free (coef)
coef : NULL
coef[0] : NULL
coef[1] : NULL
coef[2] : NULL
```

(1.e) : /* Une fois les erreurs d'allocation traités, on peut assigner les coefs, lignes et colonnes dans la variable res (résultat) */
```c
    res->coef = coef;
    res->nb_lignes = nb_lignes;
    res->nb_colonnes = nb_colonnes;
    return MAT_OK;
```

(1.f) :
```c 
    if (m->coef != NULL) {                          // s'il existe des coefs à la matrice m :
        for (int i = 0; i < m->nb_lignes; i++) {    // pour chaque itération de la 1ère à la dernière ligne
            free(m->coef[i]);                       // on libère une ligne
        }
        free(m->coef);                              // puis on libère la matrice
    }
```
Bilan ./test_ex1 :
```text
=== Exercice 1 : matrice.c ===
[OK]    (1.a) message pour MAT_ERR_DIMENSIONS
[OK]    (1.a) message pour MAT_ERR_DEPASSEMENT
[OK]    (1.b) une matrice 1 x 2 est valide
[OK]    (1.b) NULL n'est pas valide
[OK]    (1.b) MATRICE_VIDE n'est pas valide
[OK]    (1.b) une matrice de 0 ligne n'est pas valide
=== Exercice 1 : matrice_alloc.c ===
[OK]    (1.c) mat_allouer refuse 0 ligne
[OK]    (1.c) mat_allouer refuse -2 colonnes
[OK]    (1.c) mat_allouer refuse une dimension > MAT_DIM_MAX
[OK]    (1.e) mat_allouer(&m, 3, 4) renvoie MAT_OK
[OK]    (1.e) m a 3 lignes et 4 colonnes
[OK]    (1.d) tous les coefficients valent 0
[OK]    (1.f) apres mat_detruire, m est vide
[OK]    (1.f) un second mat_detruire est sans danger
[OK]    (1.f) mat_detruire(NULL) ne plante pas
=== Exercice 1 : echecs d'allocation simules ===
[OK]    (1.d) chaque echec renvoie MAT_ERR_ALLOCATION et laisse *res vide

Bilan : 16 / 16 verifications reussies.
```

## Exercice 2 :
(2.a) : /* Affiche matrice */
```c
    for (int i = 0; i < m->nb_lignes; i++) {
        for (int j = 0; j < m->nb_colonnes; j++)
            printf(" %8d", m->coef[i][j]);
        putchar('\n');
    }
```

(2.b) :
```c 
    for (int i = 0; i < m->nb_lignes; i++) {                // pour chaque ligne
        for (int j = 0; j < m->nb_colonnes; j++) {          // pour chaque colonne
            if (j == 0) {                                   // pour la 1ère colonne :
                if (fprintf(f, "%d", m->coef[i][j]) < 0)    // pas d'espace au début et si erreur d'écriture :
                    return MAT_ERR_ECRITURE;                // retourne une erreur d'écriture
            } else {                                        // sinon :
                if (fprintf(f, " %d", m->coef[i][j]) < 0)   // un espace avant chaque valeur pour les séparer
                    return MAT_ERR_ECRITURE;
            }
        }
        if (fputc('\n', f) == EOF)                  
            return MAT_ERR_ECRITURE;
```

(2.c) : /* Si le nombre de lignes ou de colonnes est inférieur à 1 ou supérieur à MAT_DIM_MAX alors renvoie une erreur de format */
```c
    if (nb_lignes < 1 || nb_lignes > MAT_DIM_MAX || nb_colonnes < 1 || nb_colonnes > MAT_DIM_MAX) 
        return MAT_ERR_FORMAT;
```

(2.d) :
```c
    Matrice tmp;                                        // on initialise une matrice tmp
    err = mat_allouer(&tmp, nb_lignes, nb_colonnes);    // on alloue la matrice tmp
    if (err != MAT_OK) return err;                      // si cette matrice n'est pas remplie correctement alors retourne l'erreur correspondante
```

(2.e) :
```c
    for (int i = 0; i < nb_lignes; i++) {               // pour chaque ligne
        for (int j = 0; j < nb_colonnes; j++) {         // pour chaque colonne de la matrice 
            err = lire_entier(f, &tmp.coef[i][j]);      // vérifie si l'entier existe ou s'il est sous le bon format (si tout est bon, renvoie MAT_OK)
            if (err != MAT_OK) {                        // si une erreur est détectée
                mat_detruire(&tmp);                     // alors détruit la matrice tmp
                return err;                             // et retourne l'erreur correspondante
            }
        }
    }
```

(2.f) :
```c
    *res = tmp;         // Transfère la matrice tmp à la matrice finale res
    return MAT_OK;      // Retourne succès
```

Bilan ./test_ex2 :
```text
=== Exercice 2 : affichage (verification visuelle) ===
M (4 x 3) :
        1        2        3
        4        5        6
        7        8        9
       10       11       12
(2.a) Vous devez voir ci-dessus 4 lignes : 1 2 3 / 4 5 6 / 7 8 9 / 10 11 12
=== Exercice 2 : ecriture ===
[OK]    ouverture de test_ex2_sortie.txt en ecriture
[OK]    (2.b) mat_ecrire renvoie MAT_OK
[OK]    (2.b) le fichier contient exactement le format attendu
[OK]    (2.b) mat_ecrire refuse une matrice vide
=== Exercice 2 : relecture ===
[OK]    (2.f) mat_lire relit le fichier : MAT_OK
[OK]    (2.e) la matrice relue est identique
[OK]    (2.e) espaces et sauts de ligne en trop sont toleres
=== Exercice 2 : fichiers incorrects ===
[OK]    (2.c) fichier vide
[OK]    (2.c) dimension nulle
[OK]    (2.c) dimension negative
[OK]    (2.e) coefficient manquant
[OK]    (2.e) donnee non numerique
[OK]    (2.e) entier trop grand pour un int
=== Exercice 2 : echec d'allocation simule ===
[OK]    (2.d) mat_lire renvoie MAT_ERR_ALLOCATION proprement

Bilan : 14 / 14 verifications reussies.
```

## Exercice 3 :
(3.a) THEEEOEEEOEOEOOEOEOEOEOEOOE FAIT LE


## Exercice 4 :



# 5. VERIFICATIONS MEMOIRE (valgrind)

## ./test_ex1 :
```text
==593== HEAP SUMMARY:
==593==     in use at exit: 0 bytes in 0 blocks
==593==   total heap usage: 11 allocs, 11 frees, 1,228 bytes allocated
==593==
==593== All heap blocks were freed -- no leaks are possible
```

## ./test_ex2 :
```text
==590== HEAP SUMMARY:
==590==     in use at exit: 0 bytes in 0 blocks
==590==   total heap usage: 51 allocs, 51 frees, 60,728 bytes allocated
==590==
==590== All heap blocks were freed -- no leaks are possible
```

## ./test_ex3 :


## ./demo_matrices

# 6. DEMONSTRATION FINALE


# 7. BILAN ET DIFFICULTES