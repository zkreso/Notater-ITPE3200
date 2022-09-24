# Data Access Layer

Vi ønsker å flytte implementeringen av kommunikasjonen mellom tjener og database vekk fra Controller klassen. Vi abstraherer denne vekk ved å legge til et nytt lag - et data access layer. Vi flytter implementasjonen inn i en ny klasse som vi kaller KundeRepository. Vi oppretter et interface for metodene vi ønsker å ha og ender Controlleren fra å ta imot et KundeContext objekt til å ta imot interfacet vi laget.

Se utgangspunkt for dette eksempelet i eget dokument: <a href="../master/06.1 - Utgangspunkt for DAL.md">Link</a>

## Endringer i filstruktur og nye filer

Lager ny mappe som vi kaller **DAL**. Dit flyttes det som har med databasen å gjøre; DBInit.cs og KundeContext.cs. I samme mappe oppreter vi to nye filer: KundeRepository.cs. og IKundeRepository.cs

Filstrukturen ser slik ut etter endringer:

- 📁 Controllers
    - KundeController.cs
- 📁 DAL
    - DBInit.cs
    - KundeRepository.cs
    - KundeContext.cs
    - IKundeRepository.cs
- 📁 Models
    - Kunde.cs
    - Poststed.cs

(I tillegg til filer for frontend; wwwroot osv.)

- **IKundeRepository.cs** skal være grensesnitt og inneholde alle de vanlige ønskede metodene (CRUD)
- **KundeRepository.cs** skal være en klasse som implementerer grensesnittet (implementasjonen lå tidligere i KundeController)
- **KundeController.cs** skal nå ta imot klasser som implementerer grensesnittet. Implementasjonen av metodene blir delegert til den klassen det gjelder. 

**IKundeRepository.cs**

Grensesnittet skal inneholde alle de vanlige metodene (CRUD).
```cs
using KundeAppDAL.Models;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace KundeAppDAL.DAL
{
    public interface IKundeRepository
    {
        Task<bool> Lagre(Kunde innKunde);
        Task<bool> Endre(Kunde endreKunde);
        Task<bool> Slett(int id);
        Task<List<Kunde>> HentAlle();
        Task<Kunde> HentEn(int id);
    }
}
```

**KundeRepository.cs** 

KundeRepository skal være en klasse som implementerer grensesnittet vi laget.
```cs
using KundeAppDAL.Models;
using Microsoft.EntityFrameworkCore;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace KundeAppDAL.DAL
{
    public class KundeRepository : IKundeRepository
    {
        private readonly KundeContext _db;

        public KundeRepository(KundeContext kundeDb)
        {
            _db = kundeDb;
        }

        public async Task<bool> Lagre(Kunde innKunde)
        {
            try
            {
                var nyKundeRad = new Kunde();
                nyKundeRad.Fornavn = innKunde.Fornavn;
                nyKundeRad.Etternavn = innKunde.Etternavn;
                nyKundeRad.Adresse = innKunde.Adresse;

                var sjekkPostSted = _db.Poststeder.Find(innKunde.Poststed.Postnr);
                if (sjekkPostSted == null)
                {
                    var nyPoststedsRad = new Poststed();
                    nyPoststedsRad.Postnr = innKunde.Poststed.Postnr;
                    nyPoststedsRad.Postnavn = innKunde.Poststed.Postnavn;
                    nyKundeRad.Poststed = nyPoststedsRad;
                }
                else
                {
                    nyKundeRad.Poststed = sjekkPostSted;
                }
                _db.Kunder.Add(nyKundeRad);
                await _db.SaveChangesAsync();
                return true;
            }
            catch
            {
                return false;
            }
        }

        public async Task<bool> Endre(Kunde endreKunde)
        {
            try
            {
                Kunde enKunde = await _db.Kunder.FindAsync(endreKunde.id);

                // hvis postnummeret er endret
                if (enKunde.Poststed.Postnr != endreKunde.Poststed.Postnr)
                {
                    // hvis postnummeret ikke finnes
                    var sjekkPostSted = _db.Poststeder.Find(endreKunde.Poststed.Postnr);
                    if (sjekkPostSted == null)
                    {
                        var nyPoststedsRad = new Poststed();
                        nyPoststedsRad.Postnr = endreKunde.Poststed.Postnr;
                        nyPoststedsRad.Postnavn = endreKunde.Poststed.Postnavn;
                        enKunde.Poststed = nyPoststedsRad;
                    }
                    // hvis postnummeret finnes
                    else
                    {
                        enKunde.Poststed = sjekkPostSted;
                    }
                }

                enKunde.Fornavn = endreKunde.Fornavn;
                enKunde.Etternavn = endreKunde.Etternavn;
                enKunde.Adresse = endreKunde.Adresse;

                await _db.SaveChangesAsync();
                return true;
            }
            catch
            {
                return false;
            }
        }

        public async Task<bool> Slett(int id)
        {
            try
            {
                Kunde enKunde = await _db.Kunder.FindAsync(id);
                _db.Kunder.Remove(enKunde);
                _db.SaveChanges();
                return true;
            }
            catch
            {
                return false;
            }
        }
        public async Task<List<Kunde>> HentAlle()
        {
            try
            {
                List<Kunde> alleKunder = await _db.Kunder.Select(k => new Kunde
                {
                    id = k.id,
                    Fornavn = k.Fornavn,
                    Etternavn = k.Etternavn,
                    Adresse = k.Adresse,
                    Poststed = k.Poststed
                }).ToListAsync();

                return alleKunder;
            }
            catch
            {
                return null;
            }
        }

        public async Task<Kunde> HentEn(int id)
        {
            try
            {
                Kunde enKunde = await _db.Kunder.FindAsync(id);
                var hentetKunde = new Kunde()
                {
                    id = enKunde.id,
                    Fornavn = enKunde.Fornavn,
                    Etternavn = enKunde.Etternavn,
                    Adresse = enKunde.Adresse,
                    Poststed = enKunde.Poststed
                };
                return hentetKunde;
            }
            catch
            {
                return null;
            }
        }
    }
}
```

**KundeController.cs**

KundeController skal ta imot klasser som implementerer grensesnittet vi laget
```cs
using KundeAppDAL.DAL;
using KundeAppDAL.Models;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace KundeApp3.Controller
{
    [Route("[controller]/[action]")]
    public class KundeController : ControllerBase
    {
        private readonly IKundeRepository _db;
        public KundeController(IKundeRepository db)
        {
            _db = db;
        }

        public async Task<bool> Lagre(Kunde innKunde)
        {
            return await _db.Lagre(innKunde);
        }
        
        public async Task<List<Kunde>> HentAlle()
        {
            return await _db.HentAlle();
        }

        public async Task<bool> Slett(int id)
        {
            return await _db.Slett(id);
        }

        public async Task<Kunde> HentEn(int id)
        {
            return await _db.HentEn(id);
        }

        public async Task<bool> Endre(Kunde endreKunde)
        {
            return await _db.Endre(endreKunde);
        }
    }
}
```

**Startup.cs**

Til slutt må det legges til enda en service i Startup, unded ConfigureServices:
```cs
services.AddScoped<IKundeRepository, KundeRepository>();
```