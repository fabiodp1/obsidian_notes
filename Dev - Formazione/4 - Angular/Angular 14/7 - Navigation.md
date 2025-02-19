# Modulo

In [Angular](Angular) il miglior modo per impostare la navigazione, è caricare e configurare il router in un modulo separato. Questo viene poi importato dell'`AppModule`.

Per convenzione il nome della classe del modulo è `AppRoutingModule` e si trova nel file `app-routing.module.ts` nella directory `app`.

Per creare il modulo che useremo per il [[routing]] va lanciato il comando:

```terminal
ng generate module [module-name] --flat --module=app
```

```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';

@NgModule({
  declarations: [],
  imports: [
    CommonModule
  ]
})
export class AppRoutingModule { }
```

- `--flat` crea il file in `src/app` piuttosto che nella sua directory.
- `--module=app` dice a `ng generate` di registrarlo nell'array degli `import` dell'`AppModule`.

Per configurare il router nel modulo creato, va utilizzato il modulo `RouterModule`:

```ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HeroesComponent } from './heroes/heroes.component';

const routes: Routes = [
  { path: 'heroes', component: HeroesComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

Prima di tutto importa `RouterModule` e `Routes` in modo che l'app abbia capacità di routing (`CommonModule` qui non è più necessario).

# Rotte

`Routes` dice al `Router` quale view mostrare per la relativa path.

```ts
const routes: Routes = [
  { path: 'heroes', component: HeroesComponent }
];
```

## RouterModule.forRoot()

Il metadata di `@NgModule` inizializza il router e lo fa iniziare ad ascoltare per cambiamenti di rotta.
Questa linea di codice aggiugne `RouterModule` agli imports di `AppRoutingModule` e ne configura le routes in una volta sola chiamando `RouterModule.forRoot()`:

```ts
imports: [ RouterModule.forRoot(routes) ],
```

>Si chiama `forRoot` perchè il router viene configurato al livello root dell'applicazione. Il metodo `forRoot()` fornisce i service provider e direttive utili per il routing, ed effettua la navigazione iniziale basata sull'URL corrente del browser.

Per finire `AppRoutingModule` esporta `RouterModule` per renderlo disponibile al resto dell'applicazione:

```ts
exports: [ RouterModule ]
```

# RouterOutlet

