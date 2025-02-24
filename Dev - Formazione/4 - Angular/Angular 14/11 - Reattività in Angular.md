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
4. Se il componente è marcato come `Dirty` e utilizza una strategia di rilevamento `CheckAlways` o `OnPush`, viene richiamata la funzione `RefreshView` per procedere all'aggiornamento della view