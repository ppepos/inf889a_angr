---
author:
- Philippe Pépos Petitclerc
title: angr
subtitle: Introduction au cadriciel d'analyse de programmes binaires
institute: UQAM - INF889A
mainfont: fira
theme: metropolis
toc: true
urlcolor: mLightBrown
linkcolor: mDarkTeal
---

# Graphes de flot de contrôle
## CFGFast

 * Recouvrement récursif (*linear descent*)
 * Résolution des branchements indirects

## CFGEmulated

 * Émulation de chaque fonction à partir du début
   * Niveau de contexte variable (Appelé, appelant)
 * *Backwards slicing*, *Symbolic back traversal*

# Autres analyses

## Voir le source...

[github.com/angr/angr](https://github.com/angr/angr/tree/master/angr/analyses)

## angr-utils

Librairie qui permet de tracer (*plot*) les graphes résultants des analyses.

# Exploration

## Exploration de `fauxware`

Démonstration

# BitVectorSymbols (BVS), Claripy et SMT Solver

## BitVectorSymbols (BVS)

 * Représenter des vecteurs de bits
 * ... et les opérations dessus

``` python
>>> one = state.solver.BVV(1, 64)
>>> one
<BV64 0x1>
>>> one_hundred = state.solver.BVV(100, 64)
>>> one_hundred
<BV64 0x64>
>>> one + one_hundred
<BV64 0x65>
>>> one_hundred - one*200
<BV64 0xffffffffffffff9c>
```

## SMT Solver

 * Par défaut le solveur utilisé est *Z3* (*Microsoft Research*)
 * Permet de résoudre des systèmes
   * Donner un ensemble de valeurs satisfaisant un système de contraintes
   * Exemple: démonstration

## Claripy

 * Claripy est l'engin (abstraction) de résolution de systèmes d'`angr`.
 * L'objectif est de présenter un API plus proche des concepts d'`angr`.
 * Présente plusieurs *backends*:
   * Solver (SMT)
   * SolverVSA ([Value-Set Analysis](https://research.cs.wisc.edu/wpis/papers/cc04.pdf))
   * SolverReplacement (*Custom*)
   * etc.

### Contraintes manuelles

On peut ajouter des contraintes additionnelles manuellement. Démonstration.

# Hooks et simprocs

## Procédures fournises

On peut remplacer l'exécution de certaines fonctions par des implémentations en python. `angr` en fourni quelques unes.

``` python
p.hook(0x422690, angr.SIM_PROCEDURES['libc']['memcpy']())
p.hook(0x408F10, angr.SIM_PROCEDURES['libc']['puts']())
```

Un API existe également pour implémenter des appels système. ([Voir le code](https://github.com/angr/angr/tree/master/angr/procedures/linux_kernel))

# Veritesting

## Exécution symbolique statique (ESS ou SSE)

Technique de vérification formelle dans laquelle on représente l'exécution d'un programme comme une formule logique.

 * Même formule logique pour tout le programme (tous les chemins d'exécution du programme)

### Exemple de code

``` c
y = 25;
if (x < 18) {
		y = 5;
} else if (x > 60) {
		y = 10;
}
```

## Exécution symbolique statique (Suite)

### Visualisation de l'ESS

\begin{center}
\scalebox{0.5}{
\begin{tikzpicture}[>=latex,line join=bevel,]
%%
\node (B4) at (174.5bp,164.0bp) [draw,rectangle,align=center] {B4: y = 10 \\ $[\Delta = \{y \rightarrow 10\}]$};
  \node (B5) at (225.5bp,91.0bp) [draw,rectangle,align=center] {B5: $[\Delta = \{y \rightarrow ite(x > 18, \perp, ite(x > 60,10,25))\}]$};
  \node (B6) at (119.5bp,18.0bp) [draw,rectangle,align=center] {B6: $[\Delta = \{y \rightarrow ite(x > 18, 5, ite(x > 60,10,25))\}]$};
  \node (B1) at (124.5bp,362.0bp) [draw,rectangle,align=center] {B1: y = 25 \\ $[\Delta = \{y \rightarrow 25\}]$ \\ if (x < 18)};
  \node (B2) at (81.5bp,218.0bp) [draw,rectangle,align=center] {B2: y = 5 \\ $[\Delta = \{y \rightarrow 5\}]$};
  \node (B3) at (174.5bp,272.0bp) [draw,rectangle,align=center] {B3: if (x > 60)};
  \draw [->] (B4) ..controls (192.98bp,137.27bp) and (200.52bp,126.77bp)  .. (B5);
  \draw [->] (B5) ..controls (186.18bp,63.666bp) and (168.73bp,51.974bp)  .. (B6);
  \draw [->] (B3) ..controls (174.5bp,237.38bp) and (174.5bp,211.88bp)  .. (B4);
  \definecolor{strokecol}{rgb}{0.0,0.0,0.0};
  \pgfsetstrokecolor{strokecol}
  \draw (189.0bp,218.0bp) node {true};
  \draw [->] (B3) ..controls (216.49bp,237.66bp) and (242.58bp,211.66bp)  .. (253.5bp,182.0bp) .. controls (261.35bp,160.69bp) and (252.45bp,136.03bp)  .. (B5);
  \draw (272.0bp,164.0bp) node {false};
  \draw [->] (B1) ..controls (116.51bp,338.27bp) and (114.36bp,331.88bp)  .. (112.5bp,326.0bp) .. controls (103.93bp,298.9bp) and (94.988bp,267.65bp)  .. (B2);
  \draw (127.0bp,317.0bp) node {true};
  \draw [->] (B1) ..controls (141.62bp,330.88bp) and (151.56bp,313.37bp)  .. (B3);
  \draw (170.0bp,317.0bp) node {false};
  \draw [->] (B2) ..controls (81.645bp,172.53bp) and (83.727bp,117.69bp)  .. (95.5bp,73.0bp) .. controls (97.93bp,63.774bp) and (101.76bp,54.124bp)  .. (B6);
%
\end{tikzpicture}
}
\end{center}


## ESS: Avantages et inconvénients

Avantages:

 * Synthèse des différents chemins d'exécution au point de confluence
   * Gains de performance (formule compacte)

Inconvénients:

 * Solveur doit résoudre des formules complexes

## Exécution symbolique dynamique (ESD ou DSE)

 * S'effectue lors de l'interprétation du programme
 * Une branche d'exécution à la fois
 * Génère des prédicats de chemins

## ESD: Construction du prédicat de chemin

 * Exécute jusqu'au branchement
 * *Fork* au branchement
 * Ajoute au prédicat de chemin la condition pour suivre la branche *vrai*
   * L'exécuteur *forké* ajoutera le complément à son pdc

## ESD: Exemple

``` c
int i = 0;
int x = read_int();
if (x > 0) {
    i = i - 1;
else {
    i = i + 2;
}
if (x == 42) {
    x = x + i
    i = 42;
}
```

## ESD: Exemple (suite)


\begin{center}
\scalebox{0.75}{
\begin{tikzpicture}[>=latex,line join=bevel,]
%%
\coordinate (invis) at (195.0bp,290.0bp);
  \node (B4) at (335.0bp,18.0bp) [draw,rectangle,align=center] {  };
  \node (A1) at (143.0bp,110.0bp) [draw,rectangle,align=center] {$i \leftarrow -1$};
  \node (A2) at (239.0bp,110.0bp) [draw,rectangle,align=center] {$i \leftarrow + 2$};
  \node (B1) at (49.0bp,18.0bp) [draw,rectangle,align=center] {$x\prime \leftarrow x + 2$ \\ $i \leftarrow 42$};
  \node (B2) at (143.0bp,18.0bp) [draw,rectangle,align=center] {  };
  \node (B3) at (239.0bp,18.0bp) [draw,rectangle,align=center] {$x\prime \leftarrow x + 2$ \\ $i \leftarrow 42$};
  \node (root) at (195.0bp,200.0bp) [draw,rectangle,align=center] {$i \leftarrow 0$};
  \draw [->] (root) ..controls (181.29bp,176.24bp) and (177.47bp,169.85bp)  .. (174.0bp,164.0bp) .. controls (168.81bp,155.24bp) and (163.18bp,145.65bp)  .. (A1);
  \definecolor{strokecol}{rgb}{0.0,0.0,0.0};
  \pgfsetstrokecolor{strokecol}
  \draw (192.0bp,155.0bp) node {$x > 0$};
  \draw [->] (A2) ..controls (269.59bp,86.405bp) and (277.29bp,80.154bp)  .. (284.0bp,74.0bp) .. controls (294.3bp,64.556bp) and (304.93bp,53.365bp)  .. (B4);
  \draw (324.5bp,64.0bp) node {$x \neq 42$};
  \draw [->] (A1) ..controls (143.0bp,78.823bp) and (143.0bp,61.108bp)  .. (B2);
  \draw (163.5bp,64.0bp) node {$x \neq 42$};
  \draw [->] (A1) ..controls (112.33bp,86.49bp) and (104.65bp,80.22bp)  .. (98.0bp,74.0bp) .. controls (88.077bp,64.718bp) and (77.95bp,53.666bp)  .. (B1);
  \draw (118.5bp,64.0bp) node {$x = 42$};
  \draw [->] (A2) ..controls (239.0bp,78.823bp) and (239.0bp,61.108bp)  .. (B3);
  \draw (259.5bp,64.0bp) node {$x = 42$};
  \draw [->] (root) ..controls (210.0bp,169.0bp) and (218.65bp,151.71bp)  .. (A2);
  \draw (242.5bp,155.0bp) node {$x <= 0$};
  \draw [->] (invis) ..controls (195.0bp,259.24bp) and (195.0bp,242.37bp)  .. (root);
  \draw (204.5bp,245.0bp) node {$\top$};
%
\end{tikzpicture}
}
\end{center}

## Avantages

 * Simple à implémenter
 * Résolution rapide des prédicats de chemins (formules plus simples)
 * Grande reproductibilité des résultats
 * Contournement facile du problème de la complétion (Analyse locale)
   * Exécuter concrètement
   * Simuler
   * Substitué
   * Ignoré

## Limitations

Explosion combinatoire de chemins (et donc d'exécuteurs)

. . .

``` c
int counter = 0 , values = 0;
for (i = 0; i < 100; i ++) {
		if (input[i] == 'B') {
				counter++;
				values += 2;
		}
}

if (counter == 75)
		bug();
```
. . .

$2^{100}$ chemins d'exécution possible. 2 sont utiles.

## Veritesting (très résumé)

Profiter de la synthèse des branchements de l'exécution symbolique statique afin de déterminer quels exécuteurs sont réellement nécessaires pour couvrir tous les chemins d'exécution.

. . .

 * DSE jusqu'à un branchement
 * Passe de transformation et d'augmentation du GFC
 * Déterminer la frontière difficile à modéliser
 * SSE
 * Créer les exécuteurs avec les nouveaux pdc
 * DSE

# Vulnerability Discovery
# Automatic Exploit Generation
