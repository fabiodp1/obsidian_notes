`HttpClient` è un meccanismo di `Angular` per comunicare con server remoti su [[HTTP]].

Per poterlo usare nell'applicazione basta importarlo negli `AppModule`:

```ts
import { HttpClientModule } from '@angular/common/http';

@NgModule({
  imports: [
    HttpClientModule,
  ],
})
```

Per poterlo usare nel componente anzi tutto va dichiarato come dipendenza:

```ts
import { HttpClient, HttpHeaders } from '@angular/common/http';

constructor(
  private http: HttpClient
) { }

/** GET heroes from the server */
getHeroes(): Observable<Hero[]> {
  return this.http.get<Hero[]>(this.heroesUrl)
}
```

# Error handling

## catchError

Per fare il `catch` degli errori, l'observable viene incanalato ("pipe") attraverso l'opertatore `catchError()`:

```ts
getHeroes(): Observable<Hero[]> {
  return this.http.get<Hero[]>(this.heroesUrl)
    .pipe(
      catchError(this.handleError<Hero[]>('getHeroes', []))
    );
}
```

Il `catchError` intercetta l'observable che fallisce, sarà poi il  metodo `handleError` a gestirlo e ritornare un risultato innocuo in modo che l'app possa continuare a funzionare.

## handleError

Invece di gestire direttamente l'errore, ritorna una funzione `error handler` per `catchError`, questa funzione viene configurato con il nome dell'operazione fallita, e con un valore di ritorno *safe*.
Può essere riutilizzata da diversi service.

```ts
/**
 * Handle Http operation that failed.
 * Let the app continue.
 *
 * @param operation - name of the operation that failed
 * @param result - optional value to return as the observable result
 */
private handleError<T>(operation = 'operation', result?: T) {
  return (error: any): Observable<T> => {

    console.error(error); // log to console instead

    this.log(`${operation} failed: ${error.message}`);

    // Let the app keep running by returning an empty result.
    return of(result as T);
  };
}
```

Poiché ogni metodo service può ritornare un `Observable` differente, `handleError` viene configurato con un generic:

```ts
/** GET heroes from the server */
getHeroes(): Observable<Hero[]> {
  return this.http.get<Hero[]>(this.heroesUrl)
    .pipe(
      tap(_ => this.log('fetched heroes')),
      catchError(this.handleError<Hero[]>('getHeroes', []))
    );
}
```

L'operatore `tap()` di [[RxJS]] osserva i valori `observable`, facendo qualcosa con quei valori e li passa avanti. La callback di `tap()` non accede ai valori stessi.

# AsyncPipe

```ts
// ...
<li *ngFor="let hero of heroes$ | async" >

// ...
export class HeroSearchComponent implements OnInit {
  heroes$!: Observable<Hero[]>;
  // ...
```

Come si può vedere, la lista heroes viene seguita da un `$`, questa è una convenzione che indica che `heroes$` è un `Observable`, non un'array.

Poichè `*ngFor` non può fare da solo nulla con un `Observable`, va usata la pipe `|` seguita da `async`. Questa sintassi identifica `AsyncPipe` di [Angular](Angular) e sottoscrive a un `Observable` automaticamente, così non è necessario farlo nella classe componente. 

# RxJS subject

```ts
private searchTerms = new Subject<string>();

// Push a search term into the observable stream.
search(term: string): void {
  this.searchTerms.next(term);
}
```

Un `Subject` è sia una sorgente di valori osservabili che un `Observable` stesso. È possibile sottoscriversi a un `Subject` come si fa con un normale `Observable`.

È possibile pushare valori nell'`Observable` chiamando il suo metodo `next(value)`.

```ts
<input #searchBox id="search-box" (input)="search(searchBox.value)" />
```

Ogni volta che l'utente digita nella casella di testo, il binding chiama search() con il valore della casella di testo come termine di ricerca. `searchTerms` diventa un `Observable` che emette un flusso continuo di termini di ricerca.

# Concatenare operatori RxJS

Passando un nuovo valore di ricerca a `searchHeroes()` dopo ogni digitazione dell'utente, crea troppe chiamate `HTTP`.

Piuttosto, il metodo `ngOnInit()` crea una pipe con l'observable `searchTerms` attraverso una sequenza di operatori `RxJS`, riducendo il numero di chiamate a `searchHeroes()`. Alla fine ritorna un observable di risultati di ricerca di eroi, dove ciascuno è un `Hero[]`.

```ts
this.heroes$ = this.searchTerms.pipe(
  // wait 300ms after each keystroke before considering the term
  debounceTime(300),

  // ignore new term if same as previous term
  distinctUntilChanged(),

  // switch to new search observable each time the term changes
  switchMap((term: string) => this.heroService.searchHeroes(term)),
);
```

- `debounceTime(300)` aspetta fino a quando il flusso di nuove entry va in pausa per tot secondi, dopo di che fa passare la nuova stringa.
- `distinctUntilChanged()` fa in modo che non passi una stringa uguale alla precedente.
- `switchMap()` chiama il servizio di ricerca per ogni termine di ricerca cha passa attraverso `debounce()` e `distinctUntilChanged()`, ritornando solo l'ultimo servizio observable.

>Con l'operatore `switchMap`, ogni evento può triggerare un `HttpClient.get()`.
>Anche con una pausa di 300ms fra richieste, si potrebbero avere molte richieste [HTTP](HTTP) e potrebbero non ritornare nell'ordine corretto.
>`switchMap` mantiene l'ordine originale delle richieste, ritornando solo la più recente, i risultati di quelle precedenti vengono scartate e cancellate.