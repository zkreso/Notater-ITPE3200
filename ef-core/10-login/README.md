# Login

Først setter vi opp muligheten til å ha brukere. Vi ønsker å lagre brukerne i databasen med krypterte passord. Men vi mottar passordet i plaintext når brukeren forsøker å logge inn. Det innebærer at vi må ha en klasse for å motta login-informasjon og en klasse for å lagre brukerne. Slik det er gjort i eksempelet så har klassen for å motta login-informasjon en egen fil under Models, mens klassen for å lagre brukere er i DbContext filen. Det går imot tidligere eksempler å ha klasser i DbContext filen, så her må man bestemme litt selv hva man gjør. Jeg hadde laget klassen for login i en DTO mappe og gitt den navnet BrukerDTO, så jeg bruker det videre i eksemplene under.

## Oppsett på server

Klassen for å ta imot login kan se slik ut:

```cs
public class BrukerDTO {
    [RegularExpression(@"^[a-zA-ZæøåÆØÅ. \-]{2,20}$")]
    public string Brukernavn { get; set; }
    [RegularExpression(@"^(?=.*[a-zA-Z])(?=.*\d)[a-zA-Z\d]{6,}$")]
    public string Passord { get; set; }
}
```

Entity klassen til brukertabellen må ha følgende atributter:

```cs
public class Bruker {
    public int Id { get; set; }
    public string Brukernavn { get; set; }
    public byte[] Passord { get; set; }
    public byte[] Salt { get; set; }
}
```

I tillegg må man selvfølgelig ha med et DbSet for denne siste klassen i DbContext;

```cs
public DbSet<Bruker> Brukere { get; set; }
```

Man trenger også en metode for å sjekke om brukernavn og passord er riktig. Implementasjonen gjøres i Repository:

```cs
public async Task<bool> LoggInn(BrukerDTO bruker) {
    try
    {
        Brukere funnetBruker = await _db.Brukere.FirstOrDefaultAsync(b => b.Brukernavn == bruker.Brukernavn);
        // sjekk passordet
        byte[] hash = LagHash(bruker.Passord, funnetBruker.Salt);
        bool ok = hash.SequenceEqual(funnetBruker.passord);
        if (ok)
        {
            return true;
        }
        return false;
    }
    catch (Exception e)
    {
        _log.LogInformation(e.Message);
        return false;
    }
}
```

Metoden bruker en hjelpemetode for å kalkulere hash'et. Denne kan godt ligge i samme Repository.

```cs
public static byte[] LagHash(string passord, byte[] salt) {
    return KeyDerivation.Pbkdf2(
        password: passord,
        salt: salt,
        prf: KeyDerivationPrf.HMACSHA512,
        iterationCount: 1000,
        numBytesRequested: 32
    );
}
```

I tillegg trenger man en metode for å generere et tildelig salt. Denne benyttes når brukeren registreres for første gang. Ved sjekk av login så hentes saltet ut fra databasen. Saltet er der for å hindre bruk av rainbow tables, så det trenger ikke være kryptert.

```cs
public static byte[] LagSalt() {
    var csp = new RNGCryptoServiceProvider();
    var salt = new byte[24];
    csp.GetBytes(salt);
    return salt;
}
```

For å teste løsningen kan det legges inn en bruker i DBInit.

I Controlleren så kaller vi login metoden og legger på inputvalidering og feilhåndtering:

```cs
public async Task<ActionrResult> LoggInn(BrukerDTO bruker) {
    if (ModelState.IsValid) {
        bool returnOK = await _db.LoggInn(bruker);
        if (!returnOK) {
            _log.LogInformation("Innloggingen feilet for bruker " + bruker.Brukernavn);
            return Ok(false);
        }
        return Ok(true);
    }
    _log.LogInformation("Feil i inputvalidering");
    return BadRequest("Feil i inputvalidering på server");
}
```

Vi returnerer altså HTTP status ok når brukernavn og passord er feil, men med boolean false som innhold.

## Oppsett på klient

På klienten kan login funksjonen se f.eks. slik ut:

```js
function loggInn() {
    const brukernavnOK = validerBrukernavn($("#brukernavn").val());
    const passordOK = validerPassord($("#passord").val());

    if (brukernavnOK && passordOK) {
        const bruker = {
            brukernavn: $("#brukernavn").val(),
            passord: $("#passord").val()
        }
        $.post("Kunde/LoggInn", bruker, function(OK) {
            if (OK) {
                window.location.href = "index.html";
            }
            else {
                $("#feil").html("Feil brukernavn eller passord");
            }
        })
        .fail(function() {
            $("#feil").html("Feil på server - prøv igjen senere");
        })
    }
}
```

Det er ikke noe som hindrer noen å bare nå index.html direkte. Til det må man ta i bruk sessions.