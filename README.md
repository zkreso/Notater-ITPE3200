# Første prosjekt - Server siden

I Visual Studio: New Project -> **ASP.NET Core Empty**

## Nye filer og mapper

Opprett følgende mapper og filer:

- ./Models/
- ./Models/Kunde.cs
- /Controllers/
- /Controllers/KundeController.cs

**Kunde.cs**

```cs
namespace KundeApp1Empty.Models
{
    public class Kunde
    {
        public string navn { get; set; }
        public string adresse { get; set; }
    }
}
```

**KundeController.cs**

```cs
using KundeApp1Empty.Models;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;

namespace KundeApp1Empty.Controllers
{
    [Route("[controller]/[action]")]
    public class KundeController : ControllerBase
    {
        public List<Kunde> HentAlle()
        {
            var kundene = new List<Kunde>();

            var kunde1 = new Kunde();
            kunde1.navn = "Per Hansen";
            kunde1.adresse = "Osloveien 82";

            var kunde2 = new Kunde
            {
                navn = "Line Jensen",
                adresse = "Askerveien 82"
            };

            kundene.Add(kunde1);
            kundene.Add(kunde2);

            return kundene;
        }
    }
}
```

## Endringer i Startup.cs

I Startup.cs må vi ta vekk det endpointet som er mappet til "/" og erstatte med en mapping til controllerne. I tillegg må vi legge til controller i ConfigureServices. Det blir seende slik ut etter endringer:

```cs
...
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers(); # ny kode
        }
...
            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers(); # erstattet kode
            });
...
```

## Endringer i launchSettings.json

I filen ./Properties/launchSettings.json må vi legge til følgende linje under "profiles" (midterste "avsnitt"):

```json
"launchUrl": "kunde/HentAlle"
```

Hvis vi var på Mac måtte vi lagt det samme til under "KundeApp1" (siste "avsnitt")

## Alternativ fremgangsmåte: Med utgangspunkt i API-malen

Vi kan alternativt ta utgangspunkt i API-malen til Visual Studio. Da sparer vi litt arbeid når det gjelder Startup.cs, men må til gjengjeld slette eksisterende eksempler og fikse litt på mappestrukturen.

I Visual Studio: New Project -> **ASP.NET Core Web API**

Slett eksemplene WeatherForecast.cs og WeatherForecastController.cs. Modellen ligger i rot, ikke i egen mappe. Vi oppretter en Models mappe og plasserer vår modell og controller i henholdsvis Models og Controllers mappene.

I Properties/launchSettings.json må vi endre launchUrl til "kunde/HentAlle"

Det vi har "spart" på å bruke denne metoden er at vi slipper å endre på Startup.cs, og at vi i lanuchSettings.json kun kan endre launchUrl i stedet for å legge til en ny linje med denne. Til gjengjeld må vi slette eksisterende model og controller og opprette mappe for Models.

# Første prosjekt - Klient siden

## Nye filer og mapper

Ta utgangspunkt i et eksisterende prosjekt og opprett følgende mapper og filer:

- ./wwwroot/
- ./wwwroot/js/
- ./wwwroot/index.html
- ./wwwroot/js/index.js

Hvis prosjektet er opprettet ut i fra mal og ikke tomt prosjekt så kan det være at disse mappene allerede eksisterer og inneholder filer.

**index.html**

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title></title>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.3/css/bootstrap.min.css">
    <script src="js/index.js"></script>
</head>
<body>
    <h1>Her er kundene:</h1>
    <div id="kundene"></div>
</body>
</html>

```

**index.js**

```js
$(function () {
    hentAlleKunder();
});

function hentAlleKunder() {
    $.get("kunde/hentAlle", function (kunder) {
        formaterKunder(kunder);
    });
}

function formaterKunder(kunder) {
    let ut = "<table class='table table-striped'>" +
        "<tr>" +
        "<th>Navn</th><th>Adresse</th><th></th><th></th>" +
        "</tr>";
    for (let kunde of kunder) {
        ut += "<tr>" +
            "<td>" + kunde.navn + "</td>" +
            "<td>" + kunde.adresse + "</td>" +
            "</tr>";
    }
    ut += "</table>";
    $("#kundene").html(ut);
}
```

## Endringer i Startup.cs

I Startup.cs må vi legge til linjen

```
app.UseStaticFiles();
```

Den skal ligge i Configuration metoden ( `public void Configure(... `). Kan settes rett under app.useRouting(); for eksempel.

## Endringer i launchSettings.json

I launchSettings.json skal launchUrl endres til index.html