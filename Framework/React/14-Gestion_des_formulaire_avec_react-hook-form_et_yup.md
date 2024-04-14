# Gestion des formulaires avec react-hook-form et yup

## Introduction à la gestion des formulaires

### Les formulaires avec _React_

_React_ n’a pas de fonctionnalités spécifiques aux formulaires.

Or, en développement _front-end_ pouvoir créer des formulaires complexes est très important.

La gestion de formulaire comporte une partie **UI** (ce que voit l'utilisateur, aspect design), **UX** (à quel point l'utilisation du formulaire est intuitive et fluide), la validation des champs, l'affichage des erreurs, l'ajout dynamique de champs, la gestion de la soumission, la gestion d'état, et enfin la gestion des événements.

C'est donc une partie très importante du développement _front-end_ qu'il est essentiel de maîtriser.

### Présentation de _React Hook Form_ et de _Yup_

**React Hook Form** est aujourd'hui la librairie la plus moderne et la plus performante pour gérer les formulaires avec _React_. Elle est basée entièrement sur les _hooks_ et a une stratégie d'optimisation des rendus extrêmement performante. C'est également la librairie la plus utilisée (elle a dépassé _Formik_ qui n'est plus maintenu).

**Yup** est une librairie de validation d'objets _JavaScript_. Elle est extrêmement utilisée dans l'écosystème _JavaScript_ et est compatible avec _React Hook Form_ grâce à un adaptateur officiel.

### Installations

Partez sur un nouveau projet pour le chapitre et installez les deux librairies :

```sh
npm i react-hook-form yup @hookform/resolvers
```

## Création d'un formulaire et introduction à useForm()

### Le hook `useForm()`

**Le _hook_ `useForm()` est le hook principal de la librairie. Il permet de gérer un formulaire.**

Voici les options possibles que nous étudierons au fur et à mesure :

```ts
useForm({
  mode: "onSubmit",
  reValidateMode: "onChange",
  defaultValues: {},
  resolver: undefined,
  context: undefined,
  criteriaMode: "firstError",
  shouldFocusError: true,
  shouldUnregister: false,
  shouldUseNativeValidation: false,
  delayError: undefined,
});
```

Le _hook_ retourne également un ensemble de méthodes et d'objets permettant de contrôler finement le formulaire : _register_, _unregister_, _formState_, _watch_, _handleSubmit_, _reset_, _resetField_, _setError_, _clearErrors_, _setValue_, _setFocus_, _getValues_, _getFieldState_, _trigger_, et _control_.

Nous les verrons au fur et à mesure de nos besoins.

Voici un premier résumé des méthodes principales que nous approfondirons ensuite.

### La méthode `handleSubmit()`

Cette méthode permet de prévenir le comportement de la soumission du formulaire, c'est-à-dire le rafraîchissement de la page.

Elle peut prendre deux fonctions de rappel en argument :

- **Le premier argument est une fonction de rappel qui est invoquée lorsque le formulaire est envoyé** (*lorsque le bouton de **type submit**, qui est le type par défaut des boutons dans les formulaires, est cliqué*). **Cette fonction de rappel reçoit en arguments les données du formulaire et l'objet d'événement.**
- **Le deuxième argument est une fonction de rappel qui est invoquée lorsqu'une erreur est émise lors de l'envoi du formulaire. Cette fonction de rappel reçoit en arguments les erreurs du formulaire et l'objet d'événement.**

Voici un exemple simple, nous verrons à quoi sert `register()` juste après :

```ts
import { useForm } from "react-hook-form";

export default function App() {
const { register, handleSubmit } = useForm();
const onSubmit = (data, e) => console.log(data, e);
const onError = (errors, e) => console.log(errors, e);

return (
<form onSubmit={handleSubmit(onSubmit, onError)}>
<input {...register("firstName")} />
<input {...register("lastName")} />
<button type="submit">Submit</button>
</form>
);
}
```

### La méthode `register()`

**Cette méthode permet d’enregistrer un champ et d'y appliquer des règles de validation.**

**Lorsqu'un champ est enregistré sa valeur sera disponible lors de l'envoi** (*dans l'objet **data** de la première fonction de rappel passé à `handleSubmit()`*) **et lors de la validation.**

**Il faut obligatoirement fournir une clé comme nom du champ à enregistrer. Ce nom doit être unique pour le formulaire.**

Elle fonctionne de cette manière :

```ts
const { onChange, onBlur, name, ref } = register('prenom');

<input
  onChange={onChange}
  onBlur={onBlur}
  name={name}
  ref={ref}
/>
```

Mais nous pouvons utiliser la syntaxe raccourcie :

```ts
<input {...register('prenom')} />
```

Cette méthode peut prendre un grand nombre d'options de validation que nous étudierons.

### La méthode `getValues()`

**Cette méthode permet de lire les valeurs du formulaire sans re-déclencher un rendu ni s'abonner aux changements des champs avec des écouteurs d'événement.**

Prenons un exemple avec deux champs et supposons que nous ayons entré *a* dans le champ *test* et *b* dans le champ *test1* :

```ts
import { useForm } from "react-hook-form";

export default function App() {
const { register, getValues } = useForm();

return (
<form>
<input {...register("test")} />
<input {...register("test1")} />

      <button
        type="button"
        onClick={() => {
          const valeurs = getValues(); // { test: "a", test1: "b" }
          const valeurDunChamp = getValues("test"); // "a"
          const valeurDePlusieursChamps = getValues(["test", "test1"]); // ["a", "b"]
        }}
      >
        Get Values
      </button>
    </form>

);
}
```

### La méthode `watch()`

**Cette méthode permet de surveiller un ou plusieurs champs et de retourner leurs valeurs. C'est utile pour afficher à un autre endroit la valeur d'un champ.**

***Attention !*** Contrairement à `getValues()`, la méthode `watch()` déclenche de nombreux rendus supplémentaires, à chaque changement du ou des champs surveillés.

Si vous ne passez rien en argument, tous les champs seront surveillés.

## Les options de register() et de useForm()

### Les principales options de `register()`

Nous allons voir toutes **les options de validation** :

- **required :** Rend un champ obligatoire. L'option prend un message d'erreur en argument ou un objet de type `{value: boolean, message: string }`.
- **maxLength :** Défini une longueur maximale pour un champ de type *text*. L'option prend en argument un objet de type `{value: number, message: string }`.
- **minLength :** Défini une longueur minimale pour un champ de type *text*. L'option prend en argument un objet de type `{value: number, message: string }`.
- **max :** Défini une valeur maximale pour un champ de type *number*. L'option prend en argument un objet de type `{value: number, message: string }`.
- **min :** Défini une valeur minimale pour un champ de type *number*. L'option prend en argument un objet de type `{value: number, message: string }`.
- **pattern :** Défini une expression régulière pour un champ. L'option prend en argument un objet de type {value: RegExp, message: string }.
- `validate()` : Défini une fonction ou plusieurs fonctions personnalisée de validation.

Par exemple pour `validate()` :

```jsx
<input
  {...register("test", {
   validate(value) {
      if (value === '1') {
      return true;
      } else {
        return "Message d'erreur";
      }
    },
  })}
/>
```

Et en version raccourcie :

```jsx
<input
  {...register("test", {
    validate: value => value === '1' || "Message d'erreur"
  })}
/>
```

Et avec plusieurs fonctions de validation :

```jsx
<input
  {...register("test1", {
    validate: {
      positive: v => parseInt(v) > 0 || 'Doit être plus grand que 0',
      lessThanTen: v => parseInt(v) < 10 || 'Doit être plus petit que 10',
      checkUrl: async () => await fetch() || "Message d'erreur",
      messages: v => !v && ['test', 'test2']
    }
  })}
/>
```

**Voici d'autres options qui ne sont pas relatives à la validation :**

- **disabled :** permet de désactiver un champ.
- valueAsNumber : permet de retourner un nombre au lieu d'une chaîne de caractères comme valeur d'un champ.
- **valueAsDate :** permet de retourner une date au lieu d'une chaîne de caractères comme valeur d'un champ.
- `setValueAs()` : permet de passer une fonction qui transforme la chaîne de caractères et la retourne comme valeur.
- `onChange()` : permet de passer une fonction invoquée à chaque changement sur le champ.
- `onBlur()` : permet de passer une fonction invoquée dès que le champ perd le focus.

Voici un exemple pour `setValueAs()` :

```jsx
<input
  type="number"
  {...register("test", {
    setValueAs: v => parseInt(v)
  })}
/>
```

Nous verrons les dernières options plus loin dans le chapitre.

### Les principales options de `useForm()`

Les principales options sont les suivantes :

- **mode :** Cette option permet de configurer la stratégie de validation avant qu'un utilisateur n'envoie le formulaire. Par défaut la valeur est *onSubmit*. Les valeurs possibles sont :
    - **onChange** (validation à chaque changement de chaque champ, déconseillé car énorme impacte sur les performances),
    - **onBlur** (validation lorsqu'un champ perd le focus),
    - **onSubmit** (validation lors de l'envoi),
    - **onTouched** (validation lors du premier événement *blur* puis à chaque changement),
    - **all** (validation sur chaque événement *blur* ou *change*, énorme impacte sur les performances).
- **reValidateMode :** Cette option permet de configurer la stratégie de validation pour les champs qui comportent au moins une erreur ***après*** qu'un utilisateur est envoyé le formulaire. Par défaut la valeur est *onChange*. Les valeurs possibles sont :
    - **onChange** (validation à chaque changement de chaque champ, déconseillé car énorme impacte sur les performances),
    - **onBlur** (validation lorsqu'un champ perd le focus),
    - **onSubmit** (validation lors de l'envoi).
- **defaultValues :** Cette option permet de préciser des valeurs par défaut pour chaque champ enregistré. Il est recommandé de définir des valeurs par défaut (qui peuvent être chaîne de caractères vide ou *null*).

Nous verrons les autres options plus tard.

## Validation avec Yup

### Gestion de la validation avec *Yup*

*Yup* est un validateur de schémas d’objets *JavaScript*.

Cela a l’air compliqué, mais c’est très simple : cela consiste à définir un schéma comme “modèle” pour vos objets et de vérifier qu’ils respectent ces schémas.

### *Yup* et *React Hook Form*

Pour utiliser *Yup* avec *React Hook Form* il faut utiliser le *resolver* prévu par la librairie, que nous avons déjà installé :

```jsx
import { yupResolver } from '@hookform/resolvers/yup';
// ...
const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm({
    resolver: yupResolver(schema),
  });
```

### Définition des schémas

La librairie *Yup* est tout simplement géniale !

Vous pouvez tout faire : de la validation synchrone, asynchrone, des validations multi-champs, et utiliser des dizaines de validateurs pour chaque type.

Impossible de tout voir, et ce n’est pas intéressant. Nous allons voir les principaux validateurs :

- `Yup.object()` : permet de définir un schéma d’objet (il est possible de définir des schémas pour des tableaux, ou seulement des nombres, des chaînes de caractères etc). Mais pour les formulaires ce sera un objet.
- `Yup.string()` : permet de définir un schéma de chaîne de caractères. Notez que par défaut, Yup appellera toString() sur la valeur avant de déterminer si elle est valide.
- `Yup.min(limit: number, message?: string)` : permet de définir une longueur minimale pour la chaîne de caractères en premier argument. En deuxième argument vous pouvez définir optionnellement le message d’erreur qui sera retourné.
- `Yup.required(message?: string)` : permet de rendre obligatoire une propriété et de renvoyer optionnellement un message passé en paramètre.
- `Yup.email(message?: string)` : permet de valider une chaîne de caractères comme étant un email grâce à une expression régulière. 
- `Schema.typeError(message: string)` : permet de définir un message d'erreur si le type ne correspond pas à celui du schéma.
- `Schema.oneOf(tableau, message?: string)` : permet d'accepter uniquement les valeurs du tableau passé en premier argument. Le deuxième argument permet de préciser un message d'erreur.
- `Yup.ref(path: string)` : permet de faire référence à un autre champ dont le chemin est précisé en premier argument.
- `Schema.test(name: string, message: string, test: function)` : permet de créer un test personnalisé. Le premier argument est le nom du validateur, le second le message d'erreur et le troisième la fonction de validation qui doit retourner un booléen.

Vous pouvez retrouver l’ensemble des validateurs [ici](https://github.com/jquense/yup).

### Exemple de la vidéo

Voici le code de l'exemple de la vidéo :

```jsx
import * as yup from 'yup';
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';

function App() {
  const schema = yup.object({
    name: yup
      .string()
      .required('Le champ est obligatoire')
      .min(2, 'Trop court')
      .max(5, 'Trop long')
      .test('isYes', "Vous n'avez pas de chance", async () => {
        const response = await fetch('https://yesno.wtf/api');
        const result = await response.json();
        console.log(result);
        return result.answer === 'yes';
      }),
    age: yup
      .number()
      .typeError('Veuillez entre un nombre')
      .min(18, 'Trop jeune'),
    password: yup
      .string()
      .required('Le mot de passe est obligatoire')
      .min(5, 'Mot de passe trop court')
      .max(10, 'Mot de passe trop long'),
    confirmPassword: yup
      .string()
      .required('Vous devez confirmer votre mot de passe')
      .oneOf(
        [yup.ref('password'), ''],
        'Les mots de passe ne correspondent pas'
      ),
  });

  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm({
    defaultValues: {
      name: '',
    },
    resolver: yupResolver(schema),
  });

  function submit(values) {
    console.log(values);
  }

  return (
    <div
      className="d-flex justify-content-center align-items-center"
      style={{ backgroundColor: '#fefefe', height: '100vh', width: '100%' }}
    >
      <form onSubmit={handleSubmit(submit)}>
        <div className="d-flex flex-column mb-20">
          <label htmlFor="name" className="mb-5">
            Nom
          </label>
          <input id="name" type="text" {...register('name')} />
          {errors?.name && (
            <p style={{ color: 'red' }}>{errors.name.message}</p>
          )}
        </div>
        <div className="d-flex flex-column mb-20">
          <label htmlFor="name" className="mb-5">
            Age
          </label>
          <input
            id="age"
            type="number"
            {...register('age', {
              valueAsNumber: true,
            })}
          />
          {errors?.age && <p style={{ color: 'red' }}>{errors.age.message}</p>}
        </div>
        <div className="d-flex flex-column mb-20">
          <label htmlFor="password" className="mb-5">
            Mot de passe
          </label>
          <input id="password" type="password" {...register('password')} />
          {errors?.password && (
            <p style={{ color: 'red' }}>{errors.password.message}</p>
          )}
        </div>
        <div className="d-flex flex-column mb-20">
          <label htmlFor="confirmPassword" className="mb-5">
            Confirmation du mot de passe
          </label>
          <input
            id="confirmPassword"
            type="password"
            {...register('confirmPassword')}
          />
          {errors?.confirmPassword && (
            <p style={{ color: 'red' }}>{errors.confirmPassword.message}</p>
          )}
        </div>
        <button className="btn btn-primary">Sauvegarder</button>
      </form>
    </div>
  );
}

export default App;
```

## Utilisation de checkbox, radio et select et structure du formulaire

### Utiliser des listes d'options

Pour utiliser des listes d'options, il suffit d'utiliser *register* sur l'élément *select* :

```jsx
import { useForm } from "react-hook-form";

export default function App() {
  const { register, handleSubmit } = useForm();
  const onSubmit = data => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <select {...register("gender")}>
        <option value="female">Femme</option>
        <option value="male">Homme</option>
      </select>
      <input type="submit" />
    </form>
  );
}
```

### Utiliser des boutons radio

Pour utiliser des boutons *radio*, il suffit d'utiliser *register* sur chaque option avec la même clé :

```ts
import { useForm } from "react-hook-form";

export default function App() {
  const { register, handleSubmit } = useForm({
    defaultValues: {
      radio: ""
    }
  });

  return (
    <form onSubmit={handleSubmit(console.log)}>
      <div>
        <label htmlFor="A">A</label>
        <input {...register("radio")} type="radio" value="A" id="A" />
      </div>
      <div>
        <label htmlFor="B">B</label>
        <input {...register("radio")} type="radio" value="B" id="B" />
      </div>
      <div>
        <label htmlFor="C">C</label>
        <input {...register("radio")} type="radio" value="C" id="C" />
      </div>

      <input type="submit" />
    </form>
  );
}
```

### Utiliser des cases à cocher

Pour utiliser des cases à cocher, il suffit d'utiliser *register* sur une case à cocher ou sur plusieurs si vous avez plusieurs choix possible :

```ts
import { useForm } from "react-hook-form";

export default function App() {
  const { register, handleSubmit } = useForm({
    defaultValues: {
      checkbox: []
    }
  });

  return (
    <form onSubmit={handleSubmit(console.log)}>
      <div>
        <label htmlFor="A">A</label>
        <input {...register("checkbox")} type="checkbox" value="A" id="A" />
      </div>
      <div>
        <label htmlFor="B">B</label>
        <input {...register("checkbox")} type="checkbox" value="B" id="B" />
      </div>
      <div>
        <label htmlFor="C">C</label>
        <input {...register("checkbox")} type="checkbox" value="C" id="C" />
      </div>

      <input type="submit" />
    </form>
  );
}
```

## Gestion de la soumission du formulaire

### Réinitialisation du formulaire

**`useForm()` met à disposition la méthode `reset()` permettant de réinitialiser le formulaire.**

Si vous utilisez la méthode sans argument, le formulaire sera réinitialisé avec les valeurs initiales.

Vous pouvez passer en premier argument les valeurs des champs après la réinitialisation et dans ce cas il est recommandé de toutes les fournir.

Vous pouvez réinitialiser le formulaire dans un effet après que le formulaire ait été bien envoyé :

```ts
useEffect(() => {
  reset({
    data: 'test'
  })
}, [isSubmitSuccessful])
```

### L'état du formulaire

**`useForm()` met à disposition un objet *formState* contenant un grand nombre de variables d'état relatives à l'état du formulaire :**

- **isDirty :** passe à *true* dès lors que l'utilisateur modifie un seul champ.
- **dirtyFields :** objet contenant les champs que l'utilisateur a modifié.
- **touchedFields :** objet contenant les champs que l'utilisateur a touché.
- **isSubmitted :** booléen qui vaut *true* dès lors que le formulaire a été envoyé. Reste à *true* tant que le formulaire n'est pas réinitialisé avec `reset()`.
- **isSubmitSuccessful :** booléen qui vaut *true* si le formulaire a été envoyé sans échec (sans rejet de promesse ou sans erreur ayant été renvoyée dans la fonction de rappel `handleSubmit()`).
- **isSubmitting :** booléen qui vaut *true* si le formulaire est en cours d'envoi.
- **submitCount :** nombre de fois que le formulaire a été envoyé.
- **isValid :** booléen qui vaut *true* si le formulaire n'a pas d'erreur (**il faut que le mode du formulaire soit *onChange*, *onBlur* ou *onTouched***).
- **isValidating :** booléen qui vaut *true* si le formulaire est en cours de validation (validation asynchrone).
- **errors :** objet contenant les erreurs du formulaire.

Grâce à l'état du formulaire, vous pouvez par exemple désactiver le bouton d'envoi :

```jsx
const { isDirty, isValid, isSubmitting } = formState;

<button disabled={isDirty || isValid || !isSubmitting} />;
```

### Définir manuellement des erreurs

**`useForm()` met à disposition la méthode `setError()` permettant de définir manuellement des erreurs sur le formulaire.**

**Le plus souvent cette méthode est utilisée dans la méthode `handleSubmit()` pour définir une erreur lors d'une validation asynchrone** (par exemple une API qui retourne une erreur de validation).

Le premier argument de la méthode est le nom du champ concerné par l'erreur, le second argument est un objet qui contient une clé *type* et une clé *message*.

Par exemple :

```ts
setError('nomChamp', { type: 'unType', message: 'Un message d\'erreur' });
```

**A noter que si le champ n'est pas enregistré avec `register()`, l'erreur ne sera enlevée qu'avec la méthode `clearErrors()`.**

La méthode `clearErrors()` retire toutes les erreurs si aucun argument ne lui est passé. Il est possible de ne retirer que certaines erreurs en passant une chaine de caractères ou un tableau de chaines de caractères (par exemple `clearErrors(["firstName", "lastName"]`).

## trigger(), setFocus() et criteriaMode

### Définir le focus manuellement sur un champ

**`useForm()` met à disposition la méthode `setFocus()` permettant de donner manuellement le focus à un champ.**

Le premier argument est le nom du champ enregistré à focus. Optionnellement, vous pouvez passer en deuxième argument `{ shouldSelect: true }` pour sélectionner le contenu du champ.

```jsx
import { useEffect } from 'react';
import { useForm } from 'react-hook-form';

export default function App() {
  const { register, handleSubmit, setFocus } = useForm();
  const onSubmit = (data) => console.log(data);

  useEffect(() => {
    setFocus("firstName");
  }, [setFocus]);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("firstName")} placeholder="Prénom" />
      <input type="submit" />
    </form>
  );
}
```

### Déclencher manuellement la validation

**`useForm()` met à disposition la méthode `trigger()` permettant de manuellement déclencher la validation du formulaire ou d'un seul champ.**

Vous pouvez ne passer aucun argument pour déclencher la validation de tout le formulaire ou passer le nom d'un champ enregistré ou plusieurs noms de champs enregistrés dans un tableau.

La plupart du temps c'est utile lorsque la validation d'un champ dépend de la valeur d'un autre champ.

Optionnellement, vous pouvez passer en deuxième argument `{ shouldFocus: true }` pour sélectionner le champ dont vous déclenchez la validation.

### Afficher toutes les erreurs d'un champ

**Par défaut, seule la première erreur de chaque champ est disponible sur l'objet *errors*.**

**Si vous souhaitez que toutes les erreurs de chaque champ soient disponibles, il faut utiliser l'option `criteriaMode: 'all'` de `useForm()` :**

```jsx
const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting, submitCount },
  } = useForm({
    defaultValues: {
      name: '',
    },
    criteriaMode: 'all',
  });
```

Vous pourrez ensuite accéder à toutes les erreurs d'un champ en utilisant la propriété *types* :

```jsx
{errors?.name && (
  <ul style={{ color: 'red' }}>
    {Object.keys(errors.name.types).map((k) => (
      <li key={k}>{errors.name.types[k]}</li>
    ))}
  </ul>
)}
```

## Gestion de champs dynamiques

### Les formulaires dynamiques avec le hook useFieldArray()

Pour gérer un formulaire dynamique, il faut utiliser le *hook* `useFieldArray()` :

```jsx
function FieldArray() {
  const { control, register } = useForm();
  const { fields, append, prepend, remove, swap, move, insert } = useFieldArray({
    control,
    name: "test",
  });

  return (
    {fields.map((champ, index) => (
      <input
        key={champ.id}
        {...register(`test.${index}.value`)}
      />
    ))}
  );
}
```

#### La propriété *id* doit être utilisée comme *key*

**`useFieldArray` génère automatiquement un identifiant unique sous la propriété *id* pour chaque champ. Elle doit être utilisée comme propriété *key* :**

```jsx
{fields.map((champ, index) => <input key={champ.id} ... />)}
```

#### Les propriétés de `useFieldArray()`

Les propriétés de `useFieldArray()` sont les suivantes :

- **name :** nom du champ contenant un tableau.
- **control :** objet fourni par `useForm()` qui permet de connecter le champ dynamique au formulaire.

#### Les objets et méthodes suivants de `useFieldArray()` 

**`useFieldArray()` retournent les objets et méthodes suivants :**

- **fields :** objet contenant au début la valeur par défaut pour le champ dynamique et une propriété id générée pour être utilisée comme propriété key pour chaque élément.
- **append(obj: object | object[], focusOptions) :** méthode permettant d'ajouter un ou plusieurs champs à la fin du champ dynamique et de définir le focus.
- **prepend(obj: object | object[], focusOptions) :** méthode permettant d'ajouter un ou plusieurs champs au début du champ dynamique et de définir le focus.
- **insert(index: number, value: object | object[], focusOptions) :** méthode permettant d'ajouter un ou plusieurs champs à un index particulier du champ dynamique et de définir le focus.
- **swap(from: number, to: number) :** méthode permettant d'inverser la position de deux champs.
- **move(from: number, to: number) :** méthode permettant de déplacer un champ d'un index à un autre.
- **update(index: number, obj: object) :** méthode permettant de modifier le champ à une position donnée.
- **replace(obj: object) :** méthode permettant de remplacer toutes les valeurs du champ dynamique.
- **remove(index?: number | number[]) :** méthode permettant de supprimer un ou plusieurs champs à la ou aux positions données.
