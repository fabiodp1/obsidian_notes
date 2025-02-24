# Utilizzo scorretto di `ngOnInit()`

### Observable in `ngOnInit()`

Non è un buon pattern fare operazioni con gli observable all'`ngOnInit()`, ad es. per inizializzare i dati del componente. È importante ricordarsi che ci si sottoscrive ad un [Observable](Observable) all'interno di una direttiva, ad es. `ngOnInit()` dovremmo ricordarci di annullare l'iscrizione al destroy per evitare possibili perdite di memoria. Ma questo renderebbe il codice troppo ridondante:

```ts
data: MyData; 

private readonly onDestroy = new Subject(); 

ngOnInit() { 
	this.dataService.getData()
	.pipe( takeUntil(this.onDestroy) )
	.subscribe( data => this.data = data, ); 
} 

ngOnDestroy() { 
	this.onDestroy.next(); 
	this.onDestroy.complete(); 
}
```

In Angular non siamo limitati ad utilizzare nei modelli solo semplici valori statici. Possiamo utilizzare `Observable` direttamente nel modello, in altre parole:

>Raramente è necessario iscriversi agli `Observable` nella classe del componente. Dovremmo iscriverci quando abbiamo effettivamente bisogno del valore.

In realtà non ne abbiamo bisogno nella classe del componente, ma nel modello, quindi bisognerebbe iscriversi li, utilizzando la [AsyncPipe](AsyncPipe) che si sottoscrive all'Observable e restituisce il valore, **Si occupa anche di annullare l'iscrizione al destroy del componente**.
Questo rende il componente molto più leggibile rispetto a `ngOnInit()`.

```ts
@Component({...}) export class Component { 
	readonly data$ = this.dataService.getData(); 
	constructor(private readonly dataService: DataService) { } 
}
```

Nell'esempio si può vedere che:

1. la proprietà `data$` ha un valore sin da subito, senza bisogno di inutili inizializzazioni provvisorie, gli viene assegnato un `Observable` e la proprietà non è più facoltativa
2. Poiché la proprietà viene valorizzata durante la creazione del componente, questa può essere `readonly`, evitando che il suo valore venga modificato.
3. Nella classe non è prevista alcuna sottoscrizione, quindi non è necessario dover gestire l'`ngOnDestroy`.
4. Nel componente non abbiamo prop enigmatiche di cui non sappiamo quando verranno valorizzate:

```ts
<p> {{ (data$ | async).someValue }} </p>
```

### Elaborazione inutile dei dati e sottoscrizioni nidificate

Si potrebbe pensare che la tecnica al punto precedente è valida ma non adatta a casi più complessi, ad es. quando abbiamo bisogno di elaborare prima i dati ritornati dall'`Observable`, o che comunque sia scomodo dover mettere il `| async` ovunque avessimo bisogno del valore ad es:

```ts
readonly vm$ = combineLatest([
	this.route.params.pipe(
		switchMap(params => this.heroService.getHero(params.id))
	),
	this.route.params.pipe(
		switchMap(params => this.heroService.getPet(params.id))
	),
	this.heroService.getCities(),
]).pipe(
	map(([hero, pet, cities]) => {
		return {
			hero,
			pet,
			cities
		}
	})
);
```

In realtà è facilmente utilizzabile utilizzando gli operatori di `RxJS`:

```ts
<ng-container *ngIf="vm$ | async as vm">
 <p> {{ vm.hero.name }} </p>
 <p> {{ vm.hero.surname }} </p>
 <p> {{ vm.hero.city }} </p>
 
 <p> {{ vm.pet.name }} </p>
 
 <ul>
 <li *ngFor="let city of vm.cities"> {{ city }} </li>
 </ul>
</ng-container>
```

Tutti è racchiuso nell'observable `vm$`, una volta risolto tramite `| async` la sintassi diventa molto semplice.

### Inizializzazione tramite valori di `@Input()` scorretta

Spesso i componenti utilizzano le proprietà di input per ottenere lo stato dal componente padre, in modo da mostrarne il risultato, quindi dovrebbero essere reattivi anche in caso di cambiamenti.

>Spesso questi campi di input vengono usati semplicemente come una sorta di informazione di configurazione che utilizziamo durante l'inizializzazione del componente e basta.
>Non gestiamo l'evenienza che questi dati possano cambiare nel tempo. In questo modo sembrerà funzionare all'inizializzazione, ma al cambiare degli input non funzionerà correttamente.

```ts
@Component({
 template: `<p> {{ fullName }} </p>`,
})

export class NameComponent implements OnInit {
 @Input() name: string;
 @Input() surname: string;
 
 fullName: string;
 
 ngOnInit() {
 this.fullName = `${this.name} ${this.surname}`;
 }
}
```

La proprietà `fullName` verrà inizializzata una volta con il primo set di `name + surname` ma successivamente al cambio di questi `fullname` rimarrà invariato.
Dovremmo elaborare i dati di input ogni volta che cambiano, per farlo possiamo scegliere fra 3 opzioni:

1. utilizzando `ngOnChanges()`
2. utilizzando un `getter`
3. utilizzando un `setter`

#### ngOnChanges

Viene eseguito ad ogni modifica degli input, quindi può essere utilizzato per aggiornare il nostro stato `derivato`. Funziona allo stesso modo di `ngOnInit()` ma verrà eseguito ad ogni cambiamento, non solo all'inizio.

```ts
@Component({
 template: `<p> {{ fullName }} </p>`,
})

export class NameComponent implements OnChanges {
 @Input() name: string;
 @Input() surname: string;
 
 fullName: string;
 
 ngOnChanges() {
	 this.fullName = `${this.name} ${this.surname}`;
 }
}
```

#### Getter

Normalmente l'hook `ngOnChanges()` viene usato per logiche più complesse di quella sopra, per casi più semplici come questo è più pratico usare i `getter`.

>Per utilizzare i getter è importante settare fra i metadata del componente `ChangeDetection.OnPush` per non avere problemi di prestazioni nel caso in cui il getter eseguisse logiche più complicate.

```ts
@Component({
 template: `<p> {{ fullName }} </p>`,
 changeDetection: ChangeDetectionStrategy.OnPush, // <==
})

export class NameComponent {
 @Input() name: string;
 @Input() surname: string;
 get fullName(): string {
 return `${this.name} ${this.surname}`
 }
}
```

#### Setter

Questo metodo sfrutta la potenza dei setter di `TypeScript`. Nessuno nega che con i decoratori `@Input()` venga utilizzato un setter. Per questo è possibile utilizzarli con un po' di logica di configurazione leggera (niente di troppo complesso):

```ts
interface MyDTO {
 data: {
 name: string;
 time: string;
 }[]
}

@Component({
 template: `<p> {{ time }} </p>`
})

export class TimeComponent {
 @Input() 
 set vm(value: MyDTO) {
  const first = value.data[0];
 
  this.time = new Date(first.time);
 }
 time: Date;
}
```

>Con questo trucco non abbiamo più bisogno di un `lifecycle hook`, il pregio ma in altri casi anche il limite, è che ci limitiamo ai cambiamenti di una singola proprietà, mentre `ngOnChanges` viene chiamato ogni volta che viene modificata una proprietà di input, anche una che non interessa alla nostra logica di configurazione.
>Quindi è da tenere presente che questo metodo funziona bene quando il risultato non dipende da più di una proprietà di input.

>Inoltre con `ngOnChanges` si ha il vantaggio di sapere che il controllo di rilevamento delle modifiche è stato completato e di poter operare sull'intero nuovo stato, mentre i setter possono essere eseguiti nel bel mezzo del controllo vero e proprio.