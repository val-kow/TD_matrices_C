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
    - OK : Tout fonctionne, aucune fuite mémoire
    - ECHEC : Si quelque chose ne marche pas alors il n'a pas été trouvé


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

### Questions théoriques — Exercice 1

**Q3.1** Pourquoi `mat_detruire` prend-elle un `Matrice *` et non un `Matrice` ? Que se passerait-il, pour l’appelant, avec un passage par valeur ?

`mat_detruire` reçoit un pointeur pour pouvoir modifier directement la matrice de l’appelant après avoir libéré sa mémoire. Elle peut notamment remettre ses champs à `MATRICE_VIDE`. Avec un passage par valeur, seule une copie de la structure serait modifiée : la matrice de l’appelant conserverait ses anciens pointeurs, qui deviendraient invalides après la libération. L’appelant pourrait alors provoquer un accès à une mémoire libérée ou une double libération.

**Q3.2** Combien d’appels à `calloc` et à `free` faut-il pour une matrice `n × p` ? Si la k-ième allocation échoue, combien de libérations `mat_allouer` doit-elle effectuer ?

Il faut `n + 1` appels à `calloc` : un pour le tableau de pointeurs des lignes, puis un pour chacune des `n` lignes de `p` entiers. Si tout réussit, la destruction effectue également `n + 1` appels à `free` : un par ligne, puis un pour le tableau de pointeurs. Si la k-ième allocation échoue, les `k - 1` allocations précédentes ont réussi ; il faut donc libérer toutes ces allocations. Dans le cas où la première allocation échoue, aucune libération n’est nécessaire.

**Q3.3** Appeler deux fois `free` sur la même adresse est un comportement indéfini. Pourquoi appeler deux fois `mat_detruire` sur la même matrice est-il sans danger ? Est-ce encore vrai si l’on a fait une copie `Matrice b = a;` puis appelé `mat_detruire(&a)` et `mat_detruire(&b)` ?

`mat_detruire` libère les lignes et le tableau de pointeurs, puis remet la matrice à `MATRICE_VIDE`. Au deuxième appel, il n’y a donc plus de mémoire à libérer : l’appel est sans danger. En revanche, `Matrice b = a` réalise une copie superficielle : `a` et `b` contiennent les mêmes adresses. Après `mat_detruire(&a)`, les pointeurs de `b` deviennent invalides. Appeler ensuite `mat_detruire(&b)` peut donc libérer une seconde fois les mêmes adresses : cela provoque un comportement indéfini.

**Q3.4** Pourquoi `mat_allouer` commence-t-elle par l’affectation `*res = MATRICE_VIDE`, avant même de vérifier les dimensions ?

Cela garantit que le résultat est dans un état vide et connu dès le début de la fonction. Ainsi, si les dimensions sont invalides ou si une allocation échoue, l’appelant ne récupère pas une matrice partiellement construite ni d’anciens pointeurs. Il peut aussi appeler `mat_detruire` sans risque sur le résultat en cas d’échec.


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

### Questions théoriques — Exercice 2

**Q4.1** Pourquoi `mat_lire` remplit-elle d’abord une matrice locale avant de l’affecter à `*res` ?

La matrice locale sert de zone de travail temporaire. Si la lecture d’un coefficient échoue, `mat_lire` peut détruire cette matrice sans laisser `*res` dans un état partiellement rempli. Ce n’est qu’une fois toutes les données lues correctement que la fonction transfère la matrice complète à `*res`. Cela garantit que le résultat est valide en cas de succès.

**Q4.2** La lecture vérifie-t-elle que chaque ligne de la matrice est sur sa propre ligne de texte ? Est-ce gênant ? Justifiez le principe « tolérant en lecture, strict en écriture ».

Non. Les conversions de `fscanf` utilisées pour lire les entiers ignorent les espaces blancs, notamment les espaces, les tabulations et les retours à la ligne. La fonction vérifie donc la présence des bons nombres d’entiers, mais pas leur répartition exacte sur les lignes. Ce n’est pas gênant pour retrouver les coefficients. La lecture est tolérante pour accepter plusieurs présentations équivalentes, tandis que l’écriture reste stricte afin de produire toujours un format canonique et prévisible.

**Q4.3** Après l’appel `mat_lire(f, &m)`, le flux `f` est positionné après le dernier coefficient. Comment exploiter cette propriété pour stocker deux matrices dans un même fichier ?

On peut écrire les deux matrices l’une à la suite de l’autre dans le même fichier, chacune avec son en-tête de dimensions et ses coefficients. Le premier appel à `mat_lire` lit la première matrice et laisse le flux juste après son dernier coefficient. Un deuxième appel sur le même flux lit alors les dimensions et les coefficients de la deuxième matrice, sans avoir à rouvrir ni à repositionner le fichier.

**Q4.4** Pourquoi `mat_ecrire` appelle-t-elle `fflush` avant de renvoyer `MAT_OK` ? Cela dispense-t-il l’appelant de tester `fclose` ?

`fflush` force l’envoi des données encore présentes dans le tampon du flux. Une erreur d’écriture peut ainsi être détectée avant que `mat_ecrire` ne renvoie `MAT_OK`. Cela ne dispense pas l’appelant de tester `fclose` : la fermeture peut elle aussi signaler une erreur, notamment lors de la finalisation des opérations d’écriture.


## Exercice 3 :

(3.a) : /* Vérifie que les deux matrices ont les mêmes dimensions avant l'addition */
```c
    if (a->nb_lignes != b->nb_lignes || a->nb_colonnes != b->nb_colonnes) {
        return MAT_ERR_DIMENSIONS;
    }
```

(3.b) : /* Alloue une matrice résultat avec les dimensions de a */
```c
    Matrice r = MATRICE_VIDE;
    err = mat_allouer(&r, a->nb_lignes, a->nb_colonnes);
    if (err != MAT_OK) {
        return err;
    }
```

(3.c) : /* Calcule chaque somme en long long afin de détecter un dépassement avant la conversion en int */
```c
    long long s = (long long)a->coef[i][j] + b->coef[i][j];

    if (!tient_dans_int(s)) {
        mat_detruire(&r);
        return MAT_ERR_DEPASSEMENT;
    }
    r.coef[i][j] = (int)s;
```

(3.d) : /* Transfère la matrice résultat à res */
```c
    *res = r;
    return MAT_OK;
```

(3.e) : /* Le nombre de colonnes de a doit être égal au nombre de lignes de b */
```c
    if (a->nb_colonnes != b->nb_lignes) {
        return MAT_ERR_DIMENSIONS;
    }
```

(3.f) : /* Alloue le résultat du produit : lignes de a et colonnes de b */
```c
    Matrice r = MATRICE_VIDE;
    err = mat_allouer(&r, a->nb_lignes, b->nb_colonnes);
    if (err != MAT_OK) {
        return err;
    }
```

(3.g) : /* Calcule chaque coefficient du produit en additionnant les produits correspondants */
```c
    for (int k = 0; k < a->nb_colonnes; k++) {
        long long p = (long long)a->coef[i][k] * b->coef[k][j];
        if ((p > 0 && s > LLONG_MAX - p) ||
            (p < 0 && s < LLONG_MIN - p)) {
            mat_detruire(&r);
            return MAT_ERR_DEPASSEMENT;
        }
        s += p;
    }

    if (!tient_dans_int(s)) {
        mat_detruire(&r);
        return MAT_ERR_DEPASSEMENT;
    }
    r.coef[i][j] = (int)s;
```

(3.h) : /* Transfère le résultat du produit à res */
```c
    *res = r;
    return MAT_OK;
```

Bilan ./test_ex3 
```text
=== Exercice 3 : addition ===
[OK]    (3.a) (2 x 3) + (3 x 2) : MAT_ERR_DIMENSIONS
[OK]    (3.d) (2 x 3) + (2 x 3) : MAT_OK
[OK]    (3.c) a + b est correct
[OK]    a et b n'ont pas ete modifiees
[OK]    (3.b) le resultat possede sa propre memoire
[OK]    (3.c) INT_MAX + 1 : MAT_ERR_DEPASSEMENT
=== Exercice 3 : multiplication ===
[OK]    (3.e) (2 x 3) x (2 x 3) : MAT_ERR_DIMENSIONS
[OK]    (3.h) (2 x 3) x (3 x 2) : MAT_OK
[OK]    (3.g) a x c = (58 64 / 139 154)
[OK]    (3.f) (3 x 2) x (2 x 3) donne une matrice 3 x 3
[OK]    a et c n'ont pas ete modifiees
[OK]    (3.g) 2 x 10^10 depasse INT_MAX : MAT_ERR_DEPASSEMENT
=== Exercice 3 : cas particuliers ===
[OK]    resultat = entree refuse, a reste intacte
[OK]    (3.f) echec d'allocation : MAT_ERR_ALLOCATION, r vide

Bilan : 14 / 14 verifications reussies.
```

## Questions théoriques — Exercice 3

**Q5.1** Le paramètre const Matrice *a empêche-t-il d’écrire a->coef[0][0] = 7; ? Que protège exactement ce const ? Que faudrait-il changer pour que le compilateur refuse cette affectation ?

Oui. Comme a est un pointeur vers une matrice constante, les champs de la matrice et les éléments accessibles par ce pointeur ne peuvent pas être modifiés à travers a. Le compilateur refuse donc l’affectation a->coef[0][0] = 7;. Pour permettre cette modification, il faudrait retirer const de la déclaration du paramètre, mais ce ne serait plus conforme à la règle qui impose de ne pas modifier les matrices d’entrée.

**Q5.2** Pourquoi refuse-t-on l’appel mat_additionner(&a, &b, &a) alors que, mathématiquement, a ← a + b a un sens ? Que se passerait-il avec notre implémentation si on l’acceptait ?

Le paramètre res doit être distinct de a et de b. La fonction de vérification des paramètres refuse donc cet appel avec MAT_ERR_ARGUMENT. Si cet alias était accepté, verifier_parametres mettrait *res à MATRICE_VIDE, ce qui viderait aussi a puisque res pointe vers a. Les données d’entrée seraient alors perdues ou invalidées avant le calcul.

**Q5.3** Combien de multiplications d’entiers effectue mat_multiplier pour des matrices n × p et p × q ? Combien pour deux matrices 1000 × 1000 ?

Chaque coefficient du résultat nécessite p multiplications. Le résultat contient n × q coefficients, donc le nombre total de multiplications est n × p × q. Pour deux matrices 1000 × 1000, cela donne 1000 × 1000 × 1000 = 1 000 000 000, soit un milliard de multiplications.

**Q5.4** Pourquoi ne suffit-il pas de calculer int s = a + b; puis de tester s < a pour détecter un dépassement ?

En C, le dépassement d’un entier signé (int) provoque un comportement indéfini. Le dépassement peut donc se produire dès le calcul a + b, avant le test, et le résultat n’est pas fiable. Il faut effectuer le calcul dans un type plus large, ici long long, puis vérifier que le résultat est compris entre INT_MIN et INT_MAX avant de le convertir en int.

## Exercice 4 :
(4.a) : /* Affiche les matrices A et B */
```c
    mat_afficher("A", &a);
    mat_afficher("B", &b);
```

(4.b) :
```c
    MatErreur err = addition ? mat_additionner(&a, &b, &r) : mat_multiplier(&a, &b, &r); // si addition = 1 (Vrai) effectue une addition de matrice sinon effectue une multiplication
        if (err != MAT_OK) { 
            fprintf(stderr, "Erreur : %s (A : %d x %d, B : %d x %d)\n", mat_message_erreur(err), a.nb_lignes, a.nb_colonnes, b.nb_lignes, b.nb_colonnes); // renvoie un message d'erreur si problème
            goto nettoyage; // Envoie à la partie nettoyage en bas de main
```

(4.c) : /* si addition = 1 (Vrai) alors affiche en format addition sinon en format multiplication */
```c
addition ? mat_afficher("R = A + B", &r) : mat_afficher("R = A x B", &r);
```

(4.d) : /* Affiche le lieu de stockage du résultat et quitte le programme */
```c
printf("Resultat enregistre dans \"%s\".\n", argv[4]);
    statut = EXIT_SUCCESS;
```

### Questions théoriques — Exercice 4

**Q6.1** Pourquoi le message d’erreur est-il affiché par `main.c` (ou ses fonctions `static`) et non par `mat_lire` ? Pensez à un programme graphique qui réutiliserait la bibliothèque.

La bibliothèque contenant `mat_lire` doit rester indépendante de l’interface du programme qui l’utilise. Elle renvoie un code d’erreur et laisse le programme appelant décider comment le présenter. Un programme en ligne de commande peut afficher un message dans le terminal, tandis qu’un programme graphique peut afficher une boîte de dialogue. Si `mat_lire` affichait elle-même le message, elle imposerait une manière de gérer les erreurs à tous ses utilisateurs.

**Q6.2** Pourquoi le résultat de `fclose` est-il testé dans `sauvegarder` mais pas dans `charger` ?

Lors d’une sauvegarde, des données peuvent encore être dans le tampon d’écriture. Une erreur, par exemple un manque d’espace disque, peut n’être signalée qu’au moment de la fermeture du fichier. Il faut donc tester le résultat de `fclose` pour ne pas annoncer une sauvegarde réussie alors que l’écriture n’a pas été correctement finalisée. Dans `charger`, le fichier est seulement lu : il n’y a pas de données à vider depuis un tampon d’écriture, raison pour laquelle le code ne traite pas ce résultat de la même façon.

**Q6.3** Dans le scénario 8, le calcul a réussi mais l’enregistrement a échoué. Combien de matrices faut-il libérer ? Vérifiez que votre programme le fait.

Il faut libérer les trois matrices qui sont alors détenues par le programme : les deux matrices d’entrée `a` et `b`, ainsi que la matrice résultat `r`. L’échec de l’enregistrement ne doit pas faire oublier la libération du résultat, même si le calcul a réussi. Le chemin de nettoyage doit donc détruire ces trois matrices avant de quitter le programme.







# 5. VERIFICATIONS MEMOIRE

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
```text
==599== HEAP SUMMARY:
==599==     in use at exit: 0 bytes in 0 blocks
==599==   total heap usage: 37 allocs, 37 frees, 1,412 bytes allocated
==599==
==599== All heap blocks were freed -- no leaks are possible
```

## ./demo_matrices
Scénario 1 : 
```text
valgrind --leak-check=full ./demo_matrices add donnees/A.txt donnees/B.txt R.txt :

==941== HEAP SUMMARY:
==941==     in use at exit: 0 bytes in 0 blocks
==941==   total heap usage: 22 allocs, 22 frees, 14,968 bytes allocated
==941==
==941== All heap blocks were freed -- no leaks are possible

$ echo $?
0
```

Scénario 6 (Scénario d'erreur) : 
```text
valgrind --leak-check=full ./demo_matrices add donnees/invalide.txt donnees/B.txt R.txt :

==1241== Command: ./demo_matrices add donnees/invalide.txt donnees/B.txt R.t
==1241==
Erreur : lecture de "donnees/invalide.txt" : fichier mal formé
==1241==
==1241== HEAP SUMMARY:
==1241==     in use at exit: 0 bytes in 0 blocks
==1241==   total heap usage: 5 allocs, 5 frees, 4,600 bytes allocated
==1241==
==1241== All heap blocks were freed -- no leaks are possible

$ echo $?
1
```

Scénario 4 (Dimensions incompatibles) : 
```text
valgrind --leak-check=full ./demo_matrices mul donnees/A.txt donnees/B.txt R.txt :

==1317== Command: ./demo_matrices mul donnees/A.txt donnees/B.txt R.txt
==1317==
A (4 x 3) :
       25       48       84
       15       33       11
       45       88       10
        0       12       -5
B (4 x 3) :
        1        2        3
        4        5        6
        7        8        9
       10       11       12
Erreur : dimensions incompatibles (A : 4 x 3, B : 4 x 3)
==1317==
==1317== HEAP SUMMARY:
==1317==     in use at exit: 0 bytes in 0 blocks
==1317==   total heap usage: 15 allocs, 15 frees, 10,320 bytes allocated
==1317==
==1317== All heap blocks were freed -- no leaks are possible

$ echo $?
1
```


# 6. DEMONSTRATION FINALE
Scénario 2 : Multiplication de 2 matrices, pas d'erreur de dimensions possible
```text
$ ./demo_matrices mul donnees/A.txt donnees/C.txt R.txt
A (4 x 3) :
       25       48       84
       15       33       11
       45       88       10
        0       12       -5
B (3 x 2) :
        1        0
        0        1
        2       -1
R = A x B (4 x 2) :
      193      -36
       37       22
       65       78
      -10       17
Resultat enregistre dans "R.txt".

echo $?
0
```

Scénario 5 : Erreur attendue car le fichier absent.txt n'existe pas
```text
$ ./demo_matrices mul donnees/A.txt absent.txt R.txt
Erreur : impossible d'ouvrir "absent.txt" : No such file or directory

$ echo $?
1
```

Scénario 8 : Erreur attendue car le dossier fantome n'existe pas
```text
$ ./demo_matrices mul donnees/A.txt donnees/C.txt fantome/R.txt
A (4 x 3) :
       25       48       84
       15       33       11
       45       88       10
        0       12       -5
B (3 x 2) :
        1        0
        0        1
        2       -1
R = A x B (4 x 2) :
      193      -36
       37       22
       65       78
      -10       17
Erreur : impossible de créer "fantome/R.txt" : No such file or directory
$ echo $?
1
```

Scénario 10 : Erreur attendue car manque d'arguments (pas de add ou de mul)
```text
$ ./demo_matrices donnees/A.txt donnees/C.txt R.txt
Usage : ./demo_matrices <add|mul> <fichier_A> <fichier_B> <fichier_R>

$ echo $?
1
```


# 7. BILAN ET DIFFICULTES
Le TD était très bien expliqué et nous étions très guidées (parfois même un peu trop)

Répartition du travail :
Valentin : Réalisation de l'exercice 1, 2 et 4, rédaction du compte rendu sur readme.md et test valgrind sur tous les tests et scénarios
Théo : ...

Aide utilisée :
Valentin : IA pour des commandes GitHub pour récupérer et merge la branche de Théo
Théo : ...