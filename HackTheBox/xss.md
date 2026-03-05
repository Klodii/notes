# XSS - Cross Site Scripting

Vulnerabilita' Front End, quando non viene fatto bene l'escape dei caratteri
nelle form.


## Reflected XSS

Tipo di XSS non immagazzinato, utilizzato tipicamente per scoprire se viene
fatta una validazione dell'input da parte del FE.

Quindi si scrive in un field il codice di injection, come questo:

```
<script>alert('xss')</script>
```

E se dopo l'invio compare un Alert, allora vuol dire che quel field e'
vulnerabile.

## Stored XSS

In questo caso lo script iniettato viene salvato da qualche parte, come un db, e
poi viene visualizzato da tutti gli utenti che accedono a una certa pagina web.

Per esempio una pagina web in cui vengono mostrati tutti i commenti che sono
scritti puo' essere un buon target. Infatti se non viene fatta alcuna
validazione, si puo' inserire uno script maligno che poi verra visiualizzato e
usato da tutti gli utenti.


## DOM Based XSS

Non l'ho ancora capito bene.


## Esempi di attacchi

Gli esempi scritti qua sotto si possono inserire sia nelle field di una form, che
nei quiry parameters di un URL.

Prima di provare a caso, sarebbe utilie guardare il codice html e javascript,
per vedere come e dove vengono utilizzate i valori inseriti.

### <scipt>..</script>

```
<script>alert('xss')</script>
```

Questo e' il caso piu' semplice, in cui il FE non fa alcuna validazione
sull'input.


### onerror

```
<img src='invalid_url' onerror="alert('xss')"/>
```

In alcuni casi la parola chiave `script` viene validata e quindi non e' piu'
possibile utilizzare il primo metodo.

Nel caso in cui il contenuto della field viene poi mostrato in una pagina in cui
il codice html viene utilizzato, si puo' usufruire di questo altro metodo.

### inject script dentro un altro javascript

Per esempio dato questa riga di codice presente nel html del sito web

```
    <img src="/static/loading.gif" onload="startTimer('{{ timer }}');" />
```

Dove `timer` e' una variabile che viene passata dall'utente tramite field.
Nessuna validazione viene fatta su di essa.

In questo caso possiamo pensare di iniettare il nostro codice all'interno della
variabile `timer`.

Siamo fortunati che siamo gia' all'interno di codice javascript, quindi non
dobbiamo usare `<scirpt>` o `onerror`.

Quindi possiamo scrivere direttamente il nostro codice maligno, per esempio

```
alert('xss');
```

pero' non possiamo scrivere solo questo, altrimenti il codice andra' in errore.

Quindi dobbiamo ragionare in un modo che non faccia rompere il codice.
Possiamo passare il valore che la funzione si aspetta e poi accodare il nostro
comando di alert.

L'input sara':

```
3');alert('XSS');//
```

1. `3');` serve per far eseguire il comando `startTimer` senza rompere il codice
2. `alert('XSS');` e' il nostro codice maligno che viene accodato
3. `//` serve per commentare il resto del codice, altrimenti riceveremmo un
   errore e non verrebbe eseguito nulla


Quindi il risultato finale e' questo

```
<img src="/static/loading.gif" onload="startTimer('3');alert('XSS');//');" />
```

come si puo' vedere `//` e' una parte chiave, perche' commenta il codice `');`
che avrebbe rotto il javascript.

### javascript:alert(1)

`javascript:` e' un comando che permette di eseguire codice javascript se
inserito nella barra del brwoser oppure dentro a un `<a href
src="javascript:alert(1)">`, che quando cliccato eseguira' il codice javascript.

## Note

Invece di usare `alert(1)`, utilizza `alert(document.domain)`, di piu' qui
https://bughunters.google.com/learn/invalid-reports/web-platform/xss/5108550411747328/when-reporting-xss-don-t-use-alert-1.
