# WHAT

Per accessibilità si intende la pratica di rendere un applicativo web utilizzabile dal maggior numero possibile di persone. Può sembrare sia utile solo per persone con disabilità, ma in realtà ne beneficiano anche altri gruppi, come quelli che utilizzano dispositivi mobile, o quelli con bassa velocità di connessione.
Le principali disabilità da tenere in considerazione sono:

- **Difficoltà visive**
	- include persone con cecità, bassi livelli visivi, cecità ai colori. Molte di queste utilizzano ingranditori di schermi, sia fisici che software. Alcune utilizzano `screen-reader`. È molto utile a proposito installare e provare uno di questi in modo da poter testare il nostro applicativo e capirne meglio il funzionamento (es. [NVDA](https://www.nvaccess.org/)).
- **Difficoltà uditive**
	- Persone non udenti o con problemi di udito, a cui è importante dare alternative testuali.
- **Difficoltà motorie**
	- Queste possono avere difficoltà ad es. ad usare il mouse, altre potrebbero essere completamente paralizzate, avendo bisogno di un `head pointer` per interagire col computer. Alcune di queste difficoltà possono essere superate dando la possibilità di utilizzare solo la tastiera per interagire con la pagina.
- **Difficoltà cognitive**
	- In questo spettro rientrano diverse tipologie, da quelle intellettive alle malattie mentali. In questo caso per intervenire sull'accessibilità si possono adoperare diverse strategie:
		- Fornire il contenuto in più di una maniera, come ad es. tramite "text-to-speech" o video.
		- Contenuto facilmente comprensibile.
		- Focalizzando l'attenzione sul contenuto importante.
		- Riducendo le distrazioni, come contenuto inutile o pubblicità.
		- Layout della pagina e navigazione consistenti.
		- Elementi familiari, come link sottolineati in blue se non visitati e porpora se visitati.
		- Dividendo i processi passi logici e essenziali, con indicatori di progresso.
		- Autenticazione il più facile possibile senza compromettere la sicurezza.
		- Rendendo i form facili da compilare, ad es. con messaggi di errore chiari.

---

# HOW

Si pensa che per rendere un applicativo accessibile, si debba impiegare un effort eccessivo, causando perdite di tempo e complicazioni.
Se ciò potrebbe essere vero nel momento in cui devo rendere accessibile un applicativo già esistente, non è così se cerchiamo di renderlo accessibile da subito.
Ad es. ponendoci da subito certe domande come:

- è il mio datepicker utilizzabile tramite screen reader?
- se il contenuto si aggiorna dinamicamente, se ne accorgeranno anche persone con difficoltà visive?
- I miei bottoni sono accessibili sia tramite tastiera che da schermi touch?

Le linee guida su come dovrebbe essere un applicativo accessibile, sono state definite da W3C nelle [WCAG](https://www.w3.org/WAI/standards-guidelines/wcag/).

Inoltre i `browser` fanno utilizzo di speciali API per l'accessibilità che espongono informazioni utili alle tecnologie assistive.

>Dove le informazioni semantiche fornite nativamente dagli elementi dell'HTML non arrivano, possono essere supportati dalle feature delle specifiche [WAI-ARIA](https://www.w3.org/TR/wai-aria/), che aggiungono informazioni semantiche all'albero dell'accessibilità.

---

# Checklist test di accessibilità

- Assicurati che il tuo HTML sia il più semanticamente corretto possibile. [Validarlo](https://validator.w3.org/) è un buon inizio, così come usare uno [strumento di audit](https://wave.webaim.org/).
- Controlla che i tuoi contenuti abbiano senso anche quando il CSS è disattivato.
- Assicurati che le funzionalità siano accessibili tramite tastiera ([vedi controlli UI](https://www.w3.org/TR/WCAG21/#keyboard-accessible)). Testa usando Tab, Invio, ecc.
- Assicurati che i contenuti non testuali abbiano alternative testuali. Uno [strumento di audit](https://wave.webaim.org/) è utile per individuare problemi.
- Controlla che il contrasto dei colori del tuo sito sia adeguato usando un [tool apposito](https://webaim.org/resources/contrastchecker/).
- Assicurati che i contenuti nascosti siano visibili dai lettori di schermo.
- Garantisci che le funzionalità siano utilizzabili anche senza JavaScript, ove possibile.
- Usa [ARIA](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA) per migliorare l’accessibilità quando appropriato.
- Esegui un audit del tuo sito con uno [strumento di verifica](https://wave.webaim.org/).
- Testa il sito con un [lettore di schermo](https://www.nvaccess.org/download/).
- Includi una dichiarazione di accessibilità in una posizione facilmente individuabile sul tuo sito per spiegare le misure adottate.

Un altro strumento utile è `Lighthouse` dei `devTools`.

---

# HTML Accessibile

Uno dei modi più semplici per rendere una pagina web accessibile, è utilizzare correttamente gli elementi dell'[HTML](HTML.md), la semantica di questi.
Infatti non solo utilizzare gli giusti elementi ci darà dello styling a gratis, ma aggiungerà anche feature come la governabilità via tastiera. Ad es. l'elemento `button` permette all'utente di spostarsi su di esso tramite il tasto `Tab` e attivarli premendo `space` o `enter`.

Inoltre è più facile da rendere responsive e pesa meno rispetto a dello spaghetti code, e migliora la [SEO](SEO) della pagina.

## Text content

Uno dei migliori supporti per gli screen reader è fornire una buona struttura per il contenuto, con `headings`, paragrafi, liste ecc.:

```html
<h1>My heading</h1>

<p>This is the first section of my document.</p>

<p>I'll add another paragraph here too.</p>

<ol>
  <li>Here is</li>
  <li>a list for</li>
  <li>you to read</li>
</ol>

<h2>My subheading</h2>

<p>
  This is the first subsection of my document. I'd love people to be able to
  find this content!
</p>

<h2>My 2nd subheading</h2>

<p>
  This is the second subsection of my content, which I think is more interesting
  than the last one.
</p>
```

In una buona struttura come quella d'esempio uno screen reader potrà:

- leggere ogni header mentre procede col contenuto, informando di quale è un heading e quale un paragrafo ecc.;
- può fermarsi dopo ogni elemento, permettendo di procedere a qualsiasi ritmo si preferisce;
- è possibile saltare all'heading successivo/precedente;
- in alcuni screen reader, permette di fare una lista degli heading in modo da poterli usare come tabella dei contenuti.

