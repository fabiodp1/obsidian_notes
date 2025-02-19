In [Angular](Angular) il miglior modo per impostare la navigazione, è caricare e configurare il router in un modulo separato. Questo viene poi importato dell'`AppModule`.

Per convenzione il nome della classe del modulo è `AppRoutingModule` e si trova nel file `app-routing.module.ts` nella directory `app`.

Per creare il modulo per il [[routing]] va lanciato il comando:

```terminal
ng generate module app-routing --flat --module=app
```

- `--flat` crea il file in `src/app` piuttosto che nella sua directory.
- `--module=app` dice a `ng generate` di registrarlo nell'array degli `import` dell'`AppModule`.

