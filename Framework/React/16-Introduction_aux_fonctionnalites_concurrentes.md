# Introduction aux fonctionnalités concurrentes

Introduction aux fonctionnalités concurrentes
La concurrence avec React

Nous allons utiliser l'analogie officielle de Dan Abramov (membre de l'équipe de React depuis le début) qui explique les nouvelles possibilités offertes par la nouvelle gestion de la concurrence par React depuis la version 18.

Sans concurrence, si j'appelle Alice et que Bob m'appelle, je dois attendre de finir l'appel avec Alice avant de rappeler Bon et de lui parler.

Avec le concurrence, je peux mettre Alice en attente lorsque Bob m'appelle, lui parler un peu, puis reprendre l'appel d'Alice.

La concurrence ne veut donc pas nécessairement dire que je peux traiter plusieurs tâches en même temps. Cela veut dire qu'à tout moment, je peux choisir de traiter une tâche prioritairement sans attendre la fin du traitement de tâches moins importantes.

En définitive, la concurrence permet que plusieurs tâches puissent se chevaucher et être traitées suivant leurs priorités respectives.

Pour revenir à React, cela permet de nombreuses nouvelles possibilités, par exemple de signaler qu'une mise à jour de l'UI n'est pas prioritaire et donc qu'elle peut attendre ou inversement qu'une tâche est prioritaire et doit être traitée immédiatement. 

Grâce à cette très importante refonte de la version 18, React peut travailler à plusieurs mises à jour de l’état de manière concurrente :

    la concurrence permet à une mise à jour plus urgente que celle en cours de traitement d’interrompre le rendu qui a déjà démarré et de créer un rendu avec les nouvelles données.
    la concurrence permet également de commencer le rendu en mémoire avant même que des données n’arrivent (du réseau ou du disque).

Pour le développeur, ces très importants changements ne l'obligent pas à réécrire son application et ne modifient pas toutes les API utilisées dans les dernières versions. Ils lui permettent d'utiliser progressivement de nouvelles fonctionnalités concurrentes (et donc de nouvelles API) au rythme qu'il souhaite. C'est une excellente nouvelle pour l'écosystème React en levant un grand nombre de limitations.

Dans ce chapitre, nous allons voir les fonctionnalités concurrentes disponibles et ajouterons les nouvelles au fur et à mesure.

 
Le composant Suspense

Le composant natif Suspense permet d'afficher un élément React (par exemple un composant) en attendant que ses composants enfants aient terminé de charger.

<Suspense fallback={<Chargement />}>
  <UnComposant />
</Suspense>

Dans notre exemple basique, le composant Suspense affichera le composant Chargement tant que le composant UnComposant n'est pas prêt à être rendu.

Suspense permet également de charger un composant de manière paresseuse (lazy-loading).

Cela permet de charger un composant uniquement lorsqu'il ait affiché pour la première fois. Cette fonctionnalité est très importante dans les grandes applications car cela permet de ne charger que certaines parties de l'arbre des composants en fonction des besoins de l'utilisateur.

Par exemple, si l'utilisateur n'est pas administrateur, cela permet de ne pas charger tous les composants relatifs à la partie administration d'une application.

Autre exemple, si une partie de l'application concerne l'affichage d'articles et une autre partie les articles mis en favoris / dans un panier, cela permet de ne charger que les composants relatifs à l'affichage et de ne charger les autres composants que si l'utilisateur va dans la partie favoris / panier.

Cela permet d'améliorer grandement le temps de chargement initial de l'application. 

 
Les transitions

Les transitions sont une nouvelle fonctionnalité concurrente de React qui permet de définir des mises à jour comme étant des transitions.

Cela permet de dire à React qu'elles ne sont pas urgentes et React saura dans quel ordre il faut traiter les mises à jour en cours. Cette fonctionnalité permet des optimisations fines des rendus et d'éviter des rendus non nécessaires, par exemple avec des données dépassées.

Le composant natif Suspense
Le composant Suspense

Comme nous l'avons vu, le composant natif Suspense permet d'afficher un élément React (par exemple un composant) en attendant que ses composants enfants aient terminé de charger.

<Suspense fallback={<Chargement />}>
  <UnComposant />
</Suspense>

Il est possible d'utiliser Suspense à tout niveau de l'application. Vous pouvez donc l'utiliser sur une branche entière de composants.

fallback permet de passer l'UI alternative à rendre en attendant que l'UI à afficher soit chargée. Tout élément React valide peut être passé mais dans la très grande majorité des cas ce sera un gif de chargement.

 

Prenons un exemple :

<>
  <Articles />
  <Suspense fallback={<Chargement />}>
    <Avis />
  </Suspense>
</>

Supposons que le composant Avis prennent plus de temps à charger que le composant Articles. Sans utilisation de Suspense, React ne pourrait rien afficher tant que les deux composants aient fini de charger.

Autrement dit, le rendu du composant Articles serait bloqué par le chargement du composant Avis.

Grâce à l'utilisation de Suspense, le composant Articles n'attend pas le chargement du composant Avis. React va pouvoir rendre le composant Chargement à la place du composant Avis et directement afficher le composant Articles.

Lorsque le composant Avis aura fini de chargé, React remplacera le composant Chargement par le composant Avis.

Prenons un second exemple :

<>
  <Header />
  <Suspense fallback={<Chargement />}>
    <Grid>
      <Articles />
      <Avis />
    </Grid>
  </Suspense>
</>

Dans ce cas, Suspense affichera le composant Header et le composant Chargement.

Lorsque le composant Articles ET le composant Avis seront prêts, React remplacera le composant Chargement par ces composants.

 
Chargement différé -paresseux- avec Suspense

Le composant Suspense permet de charger de manière différée des composants (lazy-loading).

Habituellement on importe les composants de cette manière :

import UnComposant from './UnComposant.js';

Pour charger un composant de manière différée, il faut utiliser la fonction lazy :

import { lazy } from 'react';

const UnComposant = lazy(() => import('./UnComposant.js'));

Ce code utilise l'import dynamique qui est une fonctionnalité JavaScript récente.

Maintenant que le code du composant est chargé de manière différée, lorsqu'il est demandé par l'application, il faut obligatoirement spécifier quoi afficher pendant le chargement en utilisant le composant Suspense :

<Suspense fallback={<Chargement />}>
  <h2>Titre</h2>
  <UnComposant />
 </Suspense>

Vous pouvez mettre le composant chargé paresseusement dans un Suspense, mais le Suspense peut également être mis à un niveau plus haut dans l'arbre des composants. Il suffit simplement qu'il soit présent à un endroit plus haut dans l'arbre des composants.

 
Limites actuelles de Suspense

Pour le moment, Suspense ne peut être utilisé que pour le chargement différé de composants et pour le chargement de données en utilisant une librairie ou un framework.

Il ne peut par exemple pas être directement utilisée avec l'API fetch.

Suspense avancé, useTransition et startTransition
Utilisation de chaînes de chargements

Il est possible d'utiliser plusieurs composants Suspense avant d'afficher des composants imbriqués au fur et à mesure de leurs chargements.

Lorsque le rendu d'un composant est suspendu car il doit par exemple charger des données, il active le fallback du composant Suspense parent le plus proche.

Cela signifie qu'il est possible d'imbriquer plusieurs composants Suspense afin de créer des séquences de chargements.

Par exemple :

<Suspense fallback={<ChargementPrincipal />}>
  <ComposantPrincipal>
    <Articles />
    <Suspense fallback={<PetitChargement />}>
      <Avis />
    </Suspense>
  </ComposantPrincipal>
</Suspense>

La séquence de chargement sera :

    Si Articles n'est pas encore chargé, le composant ChargementPrincipal sera affiché à la place du ComposantPrincipal.
    Une fois que le composant Articles est chargé, le ChargementPrincipal est remplacé par le ComposantPrincipal.
    Si Avis n'est pas encore chargé, le composant PetitChargement sera affiché à la place du composant Avis.
    Finalement, une fois le composant Avis chargé, il sera affiché.

 
Les transitions

Comme nous l'avons vu, les transitions permettent de dire à React que certaines mises à jour ne sont pas urgentes et React saura dans quel ordre il faut traiter les mises à jour en cours. 

Nous pouvons utiliser le hook useTransition() pour créer une transition :

const [isPending, startTransition] = useTransition();

    isPending est un booléen. Il permet de suivre si la transition est en cours ou non.
    startTransition est une fonction qui permet de dire quelle mise à jour n'est pas urgente.

En reprenant l'exemple de la vidéo nous avons :

onClick={() => startTransition(() => setPage('b'))}

Nous disons ici à React que la mise à jour de l'état de la variable d'état page n'est pas urgente. Autrement dit, qu'il s'agit d'une transition.

L'UI actuelle reste donc affichée le temps que le composant B soit chargé et prêt à être rendu.

Nous affichons un chargement le temps de la transition grâce à isPending qui permet de suivre l'état de la transition :

{isPending && <small>Petit chargement...</small>}