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


