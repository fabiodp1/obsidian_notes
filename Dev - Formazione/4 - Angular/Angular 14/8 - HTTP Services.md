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