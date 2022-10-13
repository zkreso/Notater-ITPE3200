# Inputvalidering

Når vi skal gjøre inputvalidering må vi desverre ta i bruk Regular Expressions. 

### Repetisjon regexp:

```
^[a-zA-ZæøåÆØÅ. \-]{2,20}$
```

- ^ er starten av en streng
- $ er slutten av en streng
- Hakeparanteser brukes for å kombinere typer av innhold. Her: Alle store og små bokstaver fra det engelske alfabet samt æ, ø og å, punktum, mellomrom og bindestrek.
- Krøllparanteser brukes for å bestemme antall av foregående tegn. Her mellom 2 og 20. Er også mulig å ha ubestemt antall tegn, maksimalt antall, nøyaktig antall o.l.
- \ brukes som escape character for å få med bindestreken.
    - I JavaScript så må man også escape mellomrommet og punktummet!

## Oppsett på server

I eksempelet legges inputvalideringen i Entity klassen. Det gjøres ved hjelp av dekoratøren. Man setter en dekoratør foran hvert atributt som skal valideres.

```cs
[RegularExpression(@"xxx")]
```

Regular expression skal inn istedet for `xxx`. Dvs. skal omringes med anførselstegn og ha alfakrøll foran.

Kontrollen av om valideringen lykkes legges inn i metodene i Controlleren. Der bruker vi `ModelState.IsValid` som returnerer `true` dersom valideringen lykkes. Hvis ikke valid så logger vi at inputvalideringen feilet og sender en bad request til klienten.

## Oppsett på klient

På klienten så lager vi en ny fil, `valider.js` med en funksjon for validering av hvert felt. Vi trenger en funksjon pr. felt fordi:
- Vi ønsker å skrive ut inviduelle feilmeldinger for hvert felt som det er feil i,
- på forskjellig sted på siden (ved siden av hvert felt)
- I tillegg er det forskjellig regexp på noen av feltene

Funksjonene kan f.eks. se slik ut: 

```js
function validerFornavn(fornavn) {
    const regexp = /^[a-zA-ZæøåÆØÅ\.\ \-]{2,20}$/;
    const ok = regexp.test(fornavn);
    if (!ok) {
        $("#feilFornavn").html("Fornavnet må bestå av 2 til 20 bokstaver");
        return false;
    }
    else {
        $("#feilFornavn").html("");
        return true;
    }
}
```

Når vi så skal sende data som krever validering, f.eks. lagre en kunde, så gjør vi først valideringen før vi kaller funksjonen som skal sende data til server. F.eks.:

```js
function validerOgLagreKunde() {
    const fornavnOK = validerFornavn($("#fornavn").val());
    const etternavnOK = validerEtternavn($("#etternavn").val());
    // osv.
    if (fornavnOK && etternavnOK && adresseOK && postnrOK && poststedOK) {
        lagreKunde();
    }
}
```

`valider.js` må nødvendigvis importeres i html filen.

Lagre-knappen endres til den nye metoden, og vi kan også legge til de individuelle valideringsmetodene til inputfeltene når de endres med `onchange="validerFornavn(this.value)"` hvis vi ønsker fortløpende validering.