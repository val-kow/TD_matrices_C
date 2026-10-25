1. EN-TÊTE
TD_matrices_C
Membres (ID Github): MERCIER Théo (tmercier370) et KOWALSKI Valentin (val-kow)
3A STI FISE - TD 2
Date de Remise : ../../2026 //A COMPELETER !


2. INTRODUCTION
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


3. CONCEPTION


4. REALISATION
Exercice 1 :
(1.a) :
case MAT_ERR_DIMENSIONS:   return "dimensions incompatibles";
case MAT_ERR_DEPASSEMENT:  return "résultat hors de l'intervalle des int";

(1.b) :
if (m != NULL && m->coef != NULL && m->nb_lignes >= 1 && m->nb_lignes <= MAT_DIM_MAX && m->nb_colonnes >= 1 && m->nb_colonnes <= MAT_DIM_MAX) {
    return 1;
} else {
    return 0;
}
// Lignes supprimées

(1.c) :


5. VERIFICATIONS MEMOIRE


6. DEMONSTRATION FINALE


7. BILAN ET DIFFICULTES