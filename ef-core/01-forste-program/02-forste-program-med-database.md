# Første program med database

Utgangspunktet er appen fra uke 1 med server og klient. Repetisjon av hvilke steg vi har tatt for å opprette denne:

1. Opprette nytt prosjekt ut i fra en tom mal: **ASP.NET Core Empty**
2. Opprette modell (Kunde.cs) og kontroller (KundeController.cs) i tilhørende mapper ( henholdsvis mappen Models og mappen Controllers )
3. Opprette index.html og index.js og tilhørende mapper ( wwwroot og wwwroot/js )
4. I Startup.cs:
    - Legge til `services.AddControllers();` i ConfigureSerices metoden
    - Legge til `app.UseStaticFiles();` i Configure metoden
    - Erstatte koden i app.UseEndpoints kallet med `endpoints.MapControllers();`
5. Sette launchUrl til index.html i Properties/launchSettings.json

## Installering av packages

Vi vil få bruk for to packages som vi må installere. Disse er:
- Microsoft.EntityFrameworkCore (3.1.4)
- Microsoft.EntityFrameworkCore.Sqlite (3.1.4)

Vi kan installere disse ved å gå inn på Projects -> Manage NuGet Packages. Versjonsnummeret til packages som er brukt i videoen står i parantes ovenfor. Disse var de seneste versjonene når videoen ble laget og er kompatible med 3.1, som er brukt i videoen.

*NB! Pass på å _ikke_ installere Microsoft.EntityFrameworkCore.Sqlite.Core. Vi skal altså ha den som slutter med .Sqlite, ikke den som slutter på .Core.*

## Endringer i Kunde.cs

Første steg for å gjøre programmet om til en database er å legge til en atributt id i kunde modellen (blir løpenummer i databasen). Når vi kaller atributten id eller Id med stor for bokstav så blir denne primary key i databasen og auto increment. (Merk at vi programmerer "code first" - dvs. vi skriver koden og oppretter databasen ut i fra denne)

**Kunde.cs**

```cs
namespace KundeApp1Empty.Models
{
    public class Kunde
    {
        public int id { get; set; }
        public string navn { get; set; }
        public string adresse { get; set; }

    }
}
```

## Ny klasse: KundeDB.cs

Vi oppretter klassen KundeDB.cs i Models mappen. Klassen skal knytte applikasjonen sammen med databasen.

**KundeDB.cs**

```cs
using Microsoft.EntityFrameworkCore;

namespace KundeApp1Empty.Models
{
    public class KundeDB : DbContext
    {
        public KundeDB (DbContextOptions<KundeDB> options) : base(options)
        {
            Database.EnsureCreated();
        }

        public DbSet<Kunde> Kunder { get; set; }
    }
}
```

## Endringer i Startup.cs

I Startup.cs må vi legge til følgende linje i ConfigureServices metoden:

```cs
services.AddDbContext<KundeDB>(options => options.UseSqlite("Data source=Kunde.db"));
```

Da må også Sqlite package importeres (kan gjøres ved å trykke på lyspæren)

## Endringer i KundeController.cs

Vi ønsker å endre KundeController til å ta i bruk database. Da må følgende endringer gjøres:

1. Introdusere en ny private og readonly variabel av typen KundeDB som vi kaller _kundeDB.
2. Opprette konstruktør for klassen som setter verdien på denne.
3. Endre HentAlle metoden til å hente data fra databasen.
   - Her må vi importere package System.Linq som er spørrespråket vi bruker til å kommunisere med databasen.

**KundeController.cs**

```cs
using KundeApp1Empty.Models;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Linq;

namespace KundeApp1Empty.Controllers
{
    [Route("[controller]/[action]")]
    public class KundeController : ControllerBase
    {
        private readonly KundeDB _kundeDB;

        public KundeController(KundeDB kundeDb)
        {
            _kundeDB = kundeDb;
        }

        public List<Kunde> HentAlle()
        {
            List<Kunde> alleKundene = _kundeDB.Kunder.ToList();
            return alleKundene;
        }
    }
}
```

For å legge til data må vi bruke DBbrowser. I neste steg lager vi metoder for å manipulere data direkte fra web appen vår, slik at vi ikke trenger å bruke DBbrowser.