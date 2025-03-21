# Service

Un [[service]] non è altro che una classe che si occupa di fornire della logica ai componenti, è riutilizzabile fra componenti e viene iniettata tramite [[dependency injection]], venendo dichiarata nel costruttore del componente:

```ts
// Service
@Injectable({
  providedIn: 'root'
})
export class HeroService {

  getHeroes(): IHero[] {
    return HEROES;
  }

  constructor() { }
}

// Component
export class HeroesComponent implements OnInit {

  heroes: IHero[] = [];
  selectedHero?: IHero;

  onSelect(hero: IHero): void {
    this.selectedHero = hero;
  }

  constructor(private heroService: HeroService) { }

  ngOnInit(): void {
    this.heroes = this.heroService.getHeroes();
  }
}
```

Per creare un service basta lanciare il comando:

```terminal
ng generate service [service-name]
```

# Observable

Uno dei casi per cui vengono usati i service, è per fare fetch di dati, per cui operazioni asincrone, non come nell'esempio precedente. Il metodo deve quindi avere una signature che ne indichi l'asincronicità.

Un [[Observable]] è una delle classi chiave della libreria `RxJS`, che sta alla base di [Angular](Angular).

```ts
getHeroes(): Observable<Hero[]> {
  const heroes = of(HEROES);
  return heroes;
}
```

Il metodo `of` ritorna un observable (una sorta di Promise) a cui l'utilizzatore deve fare un `subscribe` (una sorta di `.then()`):

```ts
// Componente

getHeroes(): void {
  this.heroService.getHeroes()
      .subscribe(heroes => this.heroes = heroes);
}

ngOnInit(): void {
    getHeroes();
}
```