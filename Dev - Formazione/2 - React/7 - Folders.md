## Public
La cartella #public viene resa pubblica dal server, per cui gli elementi al suo interno (immagini, icone ecc.) possono essere richieste dal browser come da altri file.
Per questo motivo ad es. il file [[HTML]] (come altri) possono accedere ad una risorsa in public senza bisogno di specificare il path:

```html
<img src="icon.jpg" ...> // YES
<img src="./icon.jpg" ...> // NOT needed
```

>In Linea di massima vanno messi in questa cartella tutti gli elementi che non vanno presi in carico dal builder, ad es. immagini utilizzati direttamente dal file [[HTML]] o favicon.
>Se vengono utilizzati da componenti, vanno messi al di fuori di #public  in modo da essere presi in carico dal builder. 