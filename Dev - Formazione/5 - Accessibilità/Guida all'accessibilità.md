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

Un altro pregio di strutturare correttamente l'HTML è che è più facile gestirne lo stile [CSS](CSS), o manipolarlo con JS.

### Utilizzo di linguaggio chiaro

Anche il linguaggio utilizzato può inficiare l'accessibilità:

- Non usare dash se possibile, invece di "5-7" è meglio "5 to 7".
- Non usare abbreviazioni, invece di "Jan", scrivi "January".
- Espandi gli acronimi, almeno una 2 volte, e poi usa `<abbr>` per descriverli.

## Layout della pagina

Una volta si aveva la cattiva abitudine di utilizzare le `table` per creare i layout della pagina. Questa non è una buona idea perché confonde gli screen reader, specialmente se complesso.
Questo è un retaggio di quando il [CSS](CSS) non era largamente supportato dai browser.

Nel creare layout bisognerebbe utilizzare la semantica [HTML](HTML.md) (verdi [content sectioning](https://developer.mozilla.org/en-US/docs/Web/HTML/Element#content_sectioning)) cercando di evitare dove possibile generici `div`, quindi `nav` per la navigazione principale, `footer`, `article` per le unità di contenuto che si ripetono, ecc.

## UI controls

Per UI controls intendiamo le parti della UI con cui gli utenti interagiscono, es. bottoni, link e form controls.
Un aspetto chiave che riguarda l'accessibilità degli UI controls è che di default i browser permettono di manipolarli tramite tastiera.

### Labels

Le label degli UI controls sono molto importanti per tutti gli utenti, specialmente per quelli con disabilità.
Bisogna fare in modo che le label dei bottoni o link siano comprensibili e distintivi, non bisogna usare "click here" o cose di questo genere, poiché gli screen reader creerebbero una lista di bottoni non distinguibili l'uno dall'altro.

```html
// === YES ===
<p>
  Whales are really awesome creatures.
  <a href="whales.html">Find out more about whales</a>.
</p>

// === NO ===
<p>
  Whales are really awesome creatures. To find out more about whales,
  <a href="whales.html">click here</a>.
</p>
```

Anche le label dei form sono molto importanti:

```html
// === NO ===
Fill in your name: <input type="text" id="name" name="name" />

// === YES ===
<div>
  <label for="name">Fill in your name:</label>
  <input type="text" id="name" name="name" />
</div>
```

## Tabelle

Una tabella base può essere scritta come segue:

```html
<table>
  <tr>
    <td>Name</td>
    <td>Age</td>
    <td>Pronouns</td>
  </tr>
  <tr>
    <td>Gabriel</td>
    <td>13</td>
    <td>he/him</td>
  </tr>
  <tr>
    <td>Elva</td>
    <td>8</td>
    <td>she/her</td>
  </tr>
  <tr>
    <td>Freida</td>
    <td>5</td>
    <td>she/her</td>
  </tr>
</table>
```

Ma scritta così presenta un problema, non c'è modo per uno screen reader di associare fra loro righe e colonne. Per fare ciò è necessario sapere quali sono le righe dell'header e se stanno intestando righe, colonne, ecc.
Per questo è importante usare gli `scope tag`e i tag corretti, come `<thead>`, `<tbody>`, `<tfoot>`, `<th>`.

## Oltre al testo

Di per se i contenuti testuali sono accessibili, ma la stessa cosa non si può necessariamente dire per quelli multimediali, per quello bisogna utilizzare gli strumenti che ci vengono messi a disposizione dai tag html e gli attributi AREA:

```html
// === NO ===
<img src="dinosaur.png" />

// === YES ===
<img
  src="dinosaur.png"
  alt="A red Tyrannosaurus Rex: A two legged dinosaur standing upright like a human, with small arms, and a large head with lots of sharp teeth." />

// === YES ===
<img
  src="dinosaur.png"
  alt="A red Tyrannosaurus Rex: A two legged dinosaur standing upright like a human, with small arms, and a large head with lots of sharp teeth."
  title="The Mozilla red dinosaur" />

// === YES ===
// Usefull if we need a label and to reutilize it around the page
<img src="dinosaur.png" aria-labelledby="dino-label" />

<p id="dino-label">
  The Mozilla red Tyrannosaurus Rex: A two legged dinosaur standing upright like a human, with small arms, and a large head with lots of sharp teeth.
</p>

```

### Figure e didascalie

L'[HTML](HTML.md) include 2 elementi, `<figure>` e `<figcaption>`, che associano una figura di qualsivoglia tipo (non necessariamente un'immagine) con una didascalia:

```html
<figure>
  <img
    src="dinosaur.png"
    alt="The Mozilla Tyrannosaurus"
    aria-describedby="dinodescr" />
  <figcaption id="dinodescr">
    A red Tyrannosaurus Rex: A two legged dinosaur standing upright like a
    human, with small arms, and a large head with lots of sharp teeth.
  </figcaption>
</figure>
```

Nonostante vi sia un supporto misto da parte degli screen reader per l'associazione fra didascalia e figura, includere `aria-labelledby` o `aria-describedby` crea l'associazione se non è presente.
Detto ciò, la struttura degli elementi vista è utile per lo styling [CSS](CSS), e fornisce un modo per aggiungere una descrizione dell'immagine accanto ad essa nel codice sorgente. 

### Attributi `alt` vuoti

```html
<h3>
  <img src="article-icon.png" alt="" />
  Tyrannosaurus Rex: the king of the dinosaurs
</h3>
```

Ci sono casi in cui un'immagine viene inclusa nella pagina, ma il suo scopo è solo decorativo.
In quel caso è corretto valorizzare l'attributo `alt` con una stringa vuota.
Questo perché se non mettessimo l'attributo lo screen reader leggerebbe tutto l'url dell'immagine, creando solo confusione.
Un'alternativa è quella di usare `role` come `role="presentation"`, anche in questo caso il reader smetterà di leggerne il contenuto che non avrebbe nessuna utilità di contenuto.



**CONTINUES....**