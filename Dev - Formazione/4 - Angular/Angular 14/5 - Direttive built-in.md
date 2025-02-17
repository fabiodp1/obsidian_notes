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

può essere usato con un'espressione:

```ts
<!-- toggle the "special" class on/off with a property -->
<div [ngClass]="isSpecial ? 'special' : ''">This div is special</div>
```

 o con un metodo:

```ts
// .ts
currentClasses: Record<string, boolean> = {};
/* . . . */
setCurrentClasses() {
  // CSS classes: added/removed per current state of component properties
  this.currentClasses =  {
    saveable: this.canSave,
    modified: !this.isUnchanged,
    special:  this.isSpecial
  };
}

// .html
<div [ngClass]="currentClasses">This div is initially saveable, unchanged, and special.</div>
```

Nell'esempio sopra il metodo verrà chiamato all'interno di `ngOnInit()`.

## NgStyle

Come `NgClass` può essere usato tramite metodo:

```ts
// .ts
currentStyles: Record<string, string> = {};
/* . . . */
setCurrentStyles() {
  // CSS styles: set per current state of component properties
  this.currentStyles = {
    'font-style':  this.canSave      ? 'italic' : 'normal',
    'font-weight': !this.isUnchanged ? 'bold'   : 'normal',
    'font-size':   this.isSpecial    ? '24px'   : '12px'
  };
}

// .html
<div [ngStyle]="currentStyles">
  This div is initially italic, normal weight, and extra large (24px).
</div>
```

Anche qui da utilizzare in `ngOnInit()` o quando vogliamo aggiornare gli stili.