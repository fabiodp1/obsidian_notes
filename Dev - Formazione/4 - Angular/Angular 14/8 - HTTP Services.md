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

