# Sessions

For å hindre at man kan få tilgang til sider dersom man ikke er logget inn så innfører vi sessions. Et session varer så lenge man er inne på siden og foretar seg noe. Dersom man ikke foretar seg noe så avsluttes session etter et forhåndsbestemt tidsintervall.

## Oppsett på server

Først og fremst trenger vi en ny pakke:

**Microsoft.AspNetCore.Session**

### Startup.cs

Følgende linjer legges til i Startup.cs for å ta i bruk sessions:

**I ConfigureServices metoden:**

```cs
services.AddSession(options =>
{
       options.Cookie.Name = ".AdventureWorks.Session"; 
       options.IdleTimeout = TimeSpan.FromSeconds(1800); // 30 minutter
       options.Cookie.IsEssential = true;
});
// denne må også være medd
services.AddDistributedMemoryCache();
```

**Og i Configure metoden:**

```cs
app.UseSession();
```

### Controller

For å sjekke om man er logget inn så bruker vi getter og setter metodene til `Http.Context` sitt `.Session` atributt. Dersom dette er null eller en tom streng så er man ikke logget inn. Dersom det inneholder en streng med et hvilket som helst innhold så er man logget inn.

Siden vi skal sjekke denne så ofte så gir vi Controlleren et konstant atributt `private const string _loggetInn = "loggetInn";`. Ellers så kan skrivefeil fort føre til feil i autorisasjonen.

Først utvider vi login-metoden. Hvis innloggingen gikk bra så gir vi atributtet verdien "LoggetInn". Denne linjen skal med der login er vellykket:

```cs
HttpContext.Session.SetString(_loggetInn, "LoggetInn");
```

Dersom login feilet så setter vi verdien til en tom streng. Denne linjen skal med der login mislyktes:

```cs
HttpContext.Session.SetString(_loggetInn, "");
```

Vi oppretter også en logg ut metode som tar i bruk samme linjen.

Så må vi innføre en sjekk for om man er logget inn i starten av alle de andre metodene som krever innlogging. Dersom man ikke er logget inn returnerer vi Unauthorized og utfører ikke metoden. Det gjøres ved å sette disse linjene først i hver metode:

```cs
if (string.IsNullOrEmpty(HttpContext.Session.GetString(_loggetInn))) {
    return Unauthorized();
}
```

## Oppsett på klient

På klienten så kan vi endre litt på .fail metodene vi har brukt hittil. Vi utvider disse til å sjekke hva slags feil som har oppstått. Dersom HTTP status koden er 401 tilsvarer dette unauthorized, og vi sender brukeren tilbake til login-siden. Det kan se f.eks. slik ut:

```js
...
.fail(function(feil)) {
    if (feil.status == 401) {
        window.location.href = 'logginn.html';
    }
    else {
        $("#feil").html("Feil på server - prøv igjen senere");
    }
}
```