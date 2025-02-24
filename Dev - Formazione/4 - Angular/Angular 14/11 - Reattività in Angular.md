# Strategie

Angular implementa 2 strategie per gestire la reattività e quindi rilevare le modifiche a livello di singoli componenti:

- `Default`
- `OnPush`

```ts
export enum ChangeDetectionStrategy {
 OnPush = 0,
 Default = 1,
}
```

Inoltre [Angular](Angular) utilizza diverse strategie per decidere se un componente figlio deve essere controllato quando controlla le modifiche del componente padre.
Quando il padre viene controllato, tutte le direttive figlie sono coinvolte e influenzate dalla strategia definita per quel componente.

>Una volta definita, una strategia non può essere cambiata a runtime.

La strategia di `Default`, internamente denominata `CheckAlways`, implica il rilevamento automatico e regolare delle modifiche di un componente a meno che la view non sia esplicitamente scollegata.

Nella strategia `OnPush` invece, internamente denominata `CheckOnce`, il rilevamento delle modifiche viene saltato a meno che il componente non sia contrassegnato come `Dirty`.

>Angular implementa meccanismi automatici per contrassegnare un componente com `Dirty`. Se necessario può essere fatto anche manualmente utilizzando il metodo `markForCheck` esposto da `ChangeDetectorRef`.

## In depth

Quando [Angular](Angular) crea un'istanza di un componente, imposta un flag corrispondente nell'istanza `LView`, che rappresenta la vista del componente.
Questo flag può essere `CeckAlways` o `Dirty`:

![](Pasted%20image%2020250224122217.png)

Questo vuol dire che tutte le istanze `LView` create per quel componente avrà uno di questi flag impostati.
Nel caso di strategia `OnPush`, il flag `Dirty` verrà disattivato automaticamente dopo il primo passaggio di rilevamento delle modifiche.

I flag impostati su `LView` vengono controllati all'interno della funzione `refreshView` nel momento in cui Angular determina se un componente deve essere controllato:

```ts
function refreshComponent (hostLView, componentHostIdx) { 
	// Solo i componenti collegati che sono CheckAlways o OnPush e dirty 
	// devono essere aggiornati 
	if (viewAttachedToChangeDetector (componentView)) { 
		const tView = ComponentView[TVIEW]; 
		if (componentView[ FLAGS ] & ( LViewFlags.CheckAlways | LViewFlags.Dirty)) 
			RefreshView (tView, ComponentView, tView. template , ComponentView[ CON
		} else if (componentView[ TRANSPLANTED_VIEWS_TO_REFRESH ] > 0) {
			// Solo i componenti collegati che sono CheckAlways 
			// o OnPush e dirty devono essere aggiornati 
			refreshContainsDirtyView(componentView); 
		}
	}
}
```

1. La funzione accetta 2 parametri:
	- `hostLView`: rappresenta la view del componente padre
	- `componentHostIdx`: l'index del componente ospitato all'interno di `hostLView`.
2. Inizialmente viene verificato se la vista del componente è ancora collegata al detector delle modifiche, garantendo che l'aggiornamento avvenga solo se il componente non è già stato distrutto.
3. Se la view è ancora collegata al detector delle modifiche, la funzione procede con il controllare il tipo di strategia di rilevamento delle modifiche applicata al componente, e se il componente è contrassegnato come `Dirty`.
4. Se il componente è marcato come `Dirty` e utilizza una strategia di rilevamento `CheckAlways` o `OnPush`, viene richiamata la funzione `RefreshView` per procedere all'aggiornamento della view. Questa funzione accetta come parametri il `tView` (Template View) del componente, la view stessa (`ComponentView`), il template associato e il contesto del componente.
5. Se il componente non è contrassegnato come `Dirty` ma ci sono ancora view trasferite da aggiornare, viene chiamata la funzione `refreshContainsDirtyView` per controllare e aggiornare eventuali viste trasferite `Dirty`.

>In sintesi `refreshComponent` assicura che solo i componenti con una strategia di rilevamento delle modifiche specifica (`CheckAlways` o `OnPush`) e contrassegnati come `Dirty` vengano aggiornati correttamente all'interno della view, garantendo così un'ottima gestione delle prestazioni e dell'aggiornamento dei componenti.

## CheckAlways (Default)

La strategia di rilevamento delle modifiche `ChangeDetection` di default implica che un componente figlio verrà sempre controllato **se il componente padre viene modificato**.
L'unica eccezione alla regola è se il `ChangeDetection` del figlio viene scollegato:

```ts
@Component({
	selector: 'a-op',
	template: `I am OnPush component`
})

export class AOpComponent {
	constructor(private cdRef: ChangeDetectorRef) {
		cdRef.detach();
	}
}
```

Quando si parla di *componente principale da controllare*, si fa riferimento al padre. In Angular se il padre non è configurato per il `ChangeDetection`, non lo saranno neanche i figli, anche se utilizzano la strategia di rilevamento predefinita.

>Normalmente in [Angular](Angular) se viene controllato il padre, automaticamente vengono controllati anche i figli, facendo parte della struttura del padre. Questo approccio è cruciale per garantire che tutti i componenti all'interno della gerarchia vengano aggiornati in modo coerente quando cambia lo stato dell'applicazione.

Angular non impone un flusso di lavoro per rilevare quando lo stato del componente cambia, quindi il comportamento predefinito è di controllare sempre i componenti per le modifiche.
Tuttavia esistono strategie come `OnPush` che consentono agli sviluppatori di ottimizzare il rilevamento delle modifiche, ad esempio tramite l'immutabilità degli oggetti passati attraverso `@Input`.

```ts
@Component({
 selector: 'parent-component',
 template: `
 <button (click)="changeName()">Change name</button>
 <child-component [userData]="userData"></child-component>
 `,
})

export class ParentComponent {
 userData = { name: 'A' };
 changeName() {
 this.userData.name = 'B';
 }
}

@Component({
 selector: 'child-component',
 template: `<span>User name: {{userData.name}}</span>`,
})

export class ChildComponent {
 @Input() userData;
}
```

Quando clicchiamo sul pulsante del componente padre, Angular attiva un gestore eventi che aggiorna il nome dell'utente nell'oggetto `user`. Durante il successivo ciclo di rilevamento delle modifiche, Angular controlla il comportamento del figlio, e aggiorna lo schermo per riflettere il nuovo nome dell'utente.

Anche se la reference all'oggetto `user` non è cambiato, poiché è stato mutato internamente, vediamo comunque il nuovo nome visualizzato sullo schermo. Questo perché Angular controlla gli oggetti non solo rispetto alla loro reference ma anche in base al loro stato interno e questo è conseguenza del fatto che Angular controlla tutti i componenti. La restrizione sull'immutabilità dell'oggetto è fondamentale perché Angular possa determinare se gli input sono stati modificati o meno.

## CheckOnce (OnPush)

Sebbene [Angular](Angular) non imponga l'immutabilità degli oggetti, ci mette a disposizione un meccanismo per dichiarare un componente come avente input immutabili, riducendo il numero di volte in cui il componente viene controllato.

Questo meccanismo si presenta sotto forma di strategia di `ChangeDetection` `OnPush`.
Internamente viene chiamata `CheckOnce`, poiché implica che il rilevamento delle modifiche venga saltato finché il componente non viene contrassegnato come `Dirty`, quindi controllato una volta e poi saltato nuovamente.

Il componente può essere contrassegnato come `Dirty` automaticamente o manualmente utilizzando il metodo `markForCheck`.

```ts
@Component({
	selector: 'parent-component',
	template: `
		<button (click)="changeName()">Change name</button>
	 	<child-component [userData]="userData"></child-component>
	`,
})

export class ParentComponent {
	userData = { name: 'A' };
	changeName() {
		this.userData.name = 'B';
	}
}

@Component({
	selector: 'child-component',
	template: `<span>User name: {{userData.name}}</span>`,
	changeDetection: ChangeDetectionStrategy.OnPush
})

export class ChildComponent {
	@Input() userData;
}
```

Quando l'applicazione viene eseguita, Angular non rileva più la modifica di `user.name`.
Si può vedere che il componente figlio viene controllato una prima volta al bootstrap restituendo il nome `A`. Ma non viene controllato durante le successive esecuzioni di `ChangeDetection`, quindi il nome non cambierà da `A`  a `B` cliccando sul pulsante.
Questo perché la reference dell'oggetto `user` passato al child come `@Input`, non è cambiata.

# Eventi e reattività

Tutti gli eventi nativi, quando lanciati da un componente, risalgono marchiando come `Dirty` tutti i componenti sopra, fino a quello `root`.

>Il presupposto è che un evento possa innescare un cambiamento nell'albero dei componenti. [Angular](Angular) non sa se i componenti padre cambieranno o meno, per questo controlla sempre ogni componente antenato al lancio di un evento.

Ovviamente verranno marchiati come `Dirty` solo i componenti antenati di quello che emette l'evento:

```
AppComponent (selezionato) 
	HeaderComponent // NON è un parente
	ContentComponent (selezionato) 
		TodosComponent (selezionato) 
			TodoComponent (selezionato) => emette evento
```

# Contrassegnare manualmente i componenti `Dirty`

Tornando all'esempio precedente e supponendo di voler aggiornare il nome di `user` ma di non voler modificare la reference, possiamo contrassegnare manualmente il componente come `Dirty`.
Per fare ciò bisogna fare l'*inject* di `changeDetectorRef` e utilizzare il suo metodo `markForCheck` per indicare ad Angular che questo componente deve essere controllato:

```ts
@Component({...})

export class ChildComponent {
	@Input() user;
	constructor(private cd: ChangeDetectorRef) {}
	someMethodWhichDetectsAndUpdate() {
		this.cd.markForCheck();
	}
}
```



