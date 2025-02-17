Le direttive sono classi che aggiungono comportamenti agli elementi dell'applicativo. Vengono usati per gestire form, liste, stili e viste.

I tipi di direttiva in [[Angular]] sono:

- `Components`: con cui si definiscono i componenti
- `Attribute`: cambiano il comportamento e l'aspetto di un elemento, componente o un'altra direttiva.
- `Structural`: per cambiare il layout del [[DOM]] aggiungendo e rimuovendo elementi.

# Attribute

Le direttive attributo più comuni sono:

- `NgClass` aggiunge e rimuove un set di classi [[CSS]]
- `NgStyle` aggiunge e rimuove un set di style [[HTML]]
- `NgModel` aggiunge il [[two-way data binding]] ad un elemento form dell'[[HTML]]

>Molti `NgModules` come `RouterModule` e `FormsModule` definiscono le proprie direttive attributo.

## NgClass

può essere usato sia con un'espressione che con un metodo:

```ts
<!-- toggle the "special" class on/off with a property -->
<div [ngClass]="isSpecial ? 'special' : ''">This div is special</div>
```

```ts

```