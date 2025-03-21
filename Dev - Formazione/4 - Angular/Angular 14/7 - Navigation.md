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

`RouterOutlet` viene utilizzato come `<router-outlet>` per mostrare le pagine gestita dal router. In base a dove lo disponiamo decideremo dove mostrate le pagine gestite.

>`RouterOutlet` è una delle direttive che diventano disponibili all'`AppComponent` poiché `AppModule` importa `AppRoutingModule` che esporta il `RouterModule`. Quando nell'esempio precedente abbiamo lanciato `ng generate` con il flag `--module=app`, abbiamo aggiunto questo import.

# RouterLink

La direttiva `RouterLink` è un'altra direttiva pubblica messa a disposizione dal `RouterModule` e aggiunge capacità di navigazione all'interno dell'app creando link:

```ts
<h1>{{title}}</h1>
<nav>
  <a routerLink="/heroes">Heroes</a>
</nav>
<router-outlet></router-outlet>
<app-messages></app-messages>
```

Viene utilizzato l'attributo `routerLink` che è il selettore per la direttiva `RouterLink` che converte il click utente in una navigazione del router.

# Redirect

Quando la pagina viene avviata, di default ha come route iniziale quella del dominio, nel caso in cui non avessimo pagine definite per quella route finirebbe per non mostrare niente. Per dare una route di default, basta fare un redirect:

```ts
{ path: '', redirectTo: '/dashboard', pathMatch: 'full' },
```

# Parametri

per creare una rotta parametrizzata basta fare:

```ts
{ path: 'detail/:id', component: HeroDetailComponent },
```

Il componente che dovrà mostrare il dato avrà bisogno di:

- informazioni sulla rotta corrente
- il service per il recupero del modello
- l'oggetto che permette ad Angular di interagire con il browser

Inietteremo quindi:

```ts
constructor(
  private route: ActivatedRoute,
  private heroService: HeroService,
  private location: Location
) {}
```

Il componente all'init si farà dare il modello da mostrare:

```ts
ngOnInit(): void {
  this.getHero();
}

getHero(): void {
  const id = Number(this.route.snapshot.paramMap.get('id'));
  this.heroService.getHero(id)
    .subscribe(hero => this.hero = hero);
}
```

- `route.snapshot` è un'immagine statica delle informazioni della route di poco dopo la sua creazione.
- `paramMap` è un dictionary dei valori dei parametri della route estratti dall'[URL](URL).

>I parametri delle rotte sono sempre `stringhe`.

`location` è molto utile per interagire con il browser, ad esempio per fare un back:

```ts
goBack(): void {
  this.location.back();
}
```