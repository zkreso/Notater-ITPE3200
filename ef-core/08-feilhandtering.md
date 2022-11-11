# Feilhåndtering

## Feilhåndtering på server

Vi begynner med å endre returtypen i metodene i Controller til `ActionResult`. ActionResult kan returnere hva som helst egentlig, men her skal vi returnere objekter av forskjellig type som tilsvarer HTTP statuser.

Siden vi benytter `Task` så blir det endret til `Task<ActionResult>`.

> I videoen setter foreleser metoden til `async` uten å forklare hvorfor eller om dette er nødvendig. Vi bruker derfor async videre i eksempelet

Så endrer vi på metodene slik at vi returnerer en action basert på om handlingen lykkes eller ikke:

```cs
public async Task<Actionresult> Lagre (Kunde innKunde) {
    bool returnOK = await _db.Lagre(innKunde);
    if (!returOK) {
        _log.LogInformation("Kunden ble ikke lagret");
        return BadRequest("Kunden ble ikke lagret");
    }
    return Ok("Kunde lagret");
}
```

Argumentet til objektene er meldinger som følger med HTTP statusmeldingen. Hvis man skal returnere data og ikke bare en statusmelding så bruker man bare dataene som argument:

```cs
public async Task<ActionResult> HentAlle() {
    List<Kunde> alleKunder = await _db.HentAlle();
    return Ok(alleKunder);
    // Denne metoden returnerer alltid OK
}
```

Andre statuser som kan brukes:

- `NotFound()` : Tilsvarer 404. Kan f.eks. brukes når en kunde som skulle slettes ikke ble funnet

## Feilhåndtering på klienten

På klienten så bruker vi `.fail` på slutten av et GET eller POST funksjonskall (alt som returner en HTTP status). Denne kjøres dersom HTTP statusen er av type "feil" - dvs. det kan være alle type feil, både på klient og server.

Eksempel:
```js
$.post("Kunde/Endre", kunde, function () {
    // kode som kjøres når vellykket
})
.fail(function () {
    // kode som kjøres når HTTP statusen er feilkode
});
```