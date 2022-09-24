# Data Access Layer

## Utgangspunkt

Utgangspunktet er prosjektet fra "Kunde eksempel videre", men med følgende modifikasjoner:
- NewtonSOFT JSON brukes i stedet for innebygd JSON behandler
- Tabellene er separert i egne filer

Følgende packages må installeres:
- Microsoft.EntityFrameworkCore (3.1.4)
- Microsoft.EntityFrameworkCore.Sqlite (3.1.4)
- Microsoft.AspNet.Mvc.Newtonsoftjson (3.1.4?)
- Microsoft.EntityFrameworkCore.Proxies (3.1.4)

### Mappestruktur og filer:

- 📂 Controllers
    - KundeController.cs
- 📂 Models
    - DBInit.cs
    - Kunde.cs
    - Poststed.cs
    - KundeContext.cs
- 📂 wwwroot
    - index.html
    - lagre.html
    - endre.html
    - 📂 js
        - index.js
        - lagre.js
        - slett.js

### Tjenersiden

- **KundeController** inneholder alle metoder som skal snakke med databasen. Den tar imot et database objekt (KundeContext objekt)
- **DBInit** inneholder dummy data for å seede databsen ved oppstart
- **Kunde** og **Poststed** er klasser som representerer enkeltlinjer i Kunder og Poststeder tabellene
- **KundeContext** er "database"-klassen. Den oppretter Kunder-tabellen og Poststeder-tabellen ved bruk av lazy loading

**KundeController.cs**

```cs
using KundeAppDAL.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace KundeApp3.Controller
{
    [Route("[controller]/[action]")]
    public class KundeController : ControllerBase
    {
        private readonly KundeContext _db;
        public KundeController(KundeContext kundeDb)
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

**DBInit.cs**

```cs
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;

namespace KundeAppDAL.Models
{
    public class DBInit
    {
        public static void initialize(IApplicationBuilder app)
        {
            using (var serviceScope = app.ApplicationServices.CreateScope())
            {
                var context = serviceScope.ServiceProvider.GetService<KundeContext>();

                context.Database.EnsureDeleted();
                context.Database.EnsureCreated();

                var poststed1 = new Poststed { Postnr = "0270", Postnavn = "Oslo" };
                var poststed2 = new Poststed { Postnr = "1370", Postnavn = "Asker" };
                var poststed3 = new Poststed { Postnr = "0192", Postnavn = "Oslo" };

                var kunde1 = new Kunde { Fornavn = "Ole", Etternavn = "Hansen", Adresse = "Osloveien 82", Poststed = poststed1 };
                var kunde2 = new Kunde { Fornavn = "Line", Etternavn = "Jensen", Adresse = "Askerveien 72", Poststed = poststed2 };
                var kunde3 = new Kunde { Fornavn = "Zlatko", Etternavn = "Kreso", Adresse = "Alnagata 18", Poststed = poststed3 };

                context.Kunder.Add(kunde1);
                context.Kunder.Add(kunde2);
                context.Kunder.Add(kunde3);

                context.SaveChanges();
            }
        }
    }
}
```

**Kunde.cs**

```cs
namespace KundeAppDAL.Models
{
    public class Kunde
    {
        public int id { get; set; }
        public string Fornavn { get; set; }
        public string Etternavn { get; set; }
        public string Adresse { get; set; }
        public virtual Poststed Poststed { get; set; }
    }
}
```

**Poststed.cs**

```cs
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace KundeAppDAL.Models
{
    public class Poststed
    {
        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.None)]
        public string Postnr { get; set; }
        public string Postnavn { get; set; }
    }
}
```

**KundeContext.cs**

```cs
using Microsoft.EntityFrameworkCore;

namespace KundeAppDAL.Models
{
    public class KundeContext : DbContext
    {
        public KundeContext(DbContextOptions<KundeContext> options) : base(options)
        {
            Database.EnsureCreated();
        }
        public virtual DbSet<Kunde> Kunder { get; set; }

        public virtual DbSet<Poststed> Poststeder { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseLazyLoadingProxies();
        }
    }
}
```

### Klientsiden

**index.html**

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>KundeApp3</title>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.3/css/bootstrap.min.css">
    <script src="js/index.js"></script>
</head>
<body>
    <h1>Her er kundene:</h1>
    <div id="kundene"></div>
    <a href="lagre.html" class="btn btn-primary">Lagre ny kunde</a>
</body>
</html>
```

**endre.html**

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>Et skjema</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/2.3.1/css/bootstrap.min.css">
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
    <script src="js/endre.js"></script>
</head>
<body>
    <div class="container">
        <h1>Endre kunde</h1>
        <form class="form">
            <input type="hidden" id="id" />
            <div class="form-group">
                <label>Fornavn</label>
                <input type="text" id="fornavn" />
            </div>
            <div class="form-group">
                <label>Etternavn</label>
                <input type="text" id="etternavn" />
            </div>
            <div class="form-group">
                <label>Adresse</label>
                <input type="text" id="adresse" />
            </div>
            <div class="form-group">
                <label>Postnr</label>
                <input type="text" id="postnr" />
            </div>
            <div class="form-group">
                <label>Poststed</label>
                <input type="text" id="poststed" />
            </div>
            <div class="form-group">
                <input type="button" id="reg" value="Endre" class="btn btn-primary" onclick="endreKunde()" />
            </div>
            <div id="feil" style="color: red"></div>
        </form>
    </div>
</body>
</html>
```

**lagre.html** 

```html
<!DOCTYPE HTML>
<html>
<head>
    <title>Et skjema</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/2.3.1/css/bootstrap.min.css">
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
    <script src="js/lagre.js"></script>
</head>
<body>
    <div class="container">
        <h1>Lagre kunde</h1>
        <form class="form">
            <div class="form-group">
                <label>Fornavn</label>
                <input type="text" id="fornavn" />
            </div>
            <div class="form-group">
                <label>Etternavn</label>
                <input type="text" id="etternavn" />
            </div>
            <div class="form-group">
                <label>Adresse</label>
                <input type="text" id="adresse" />
            </div>
            <div class="form-group">
                <label>Postnr</label>
                <input type="text" id="postnr" />
            </div>
            <div class="form-group">
                <label>Poststed</label>
                <input type="text" id="poststed" />
            </div>
            <div class="form-group">
                <input type="button" id="reg" value="Registrer" onClick="lagreKunde()" class="btn btn-primary" />
            </div>
            <div id="feil" style="color: red"></div>
        </form>
    </div>
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
            "<td>" + kunde.fornavn + "</td>" +
            "<td>" + kunde.etternavn + "</td>" +
            "<td>" + kunde.adresse + "</td>" +
            "<td>" + kunde.poststed.postnr + "</td>" +
            "<td>" + kunde.poststed.postnavn + "</td>" +
            "<td> <a class='btn btn-primary' href='endre.html?id=" + kunde.id + "'>Endre</a></td>" +
            "<td> <button class='btn btn-danger' onclick='slettKunde(" + kunde.id + ")'>Slett</button></td>" +
            "</tr>";
    }
    ut += "</table>";
    $("#kundene").html(ut);
}

function slettKunde(id) {
    const url = "Kunde/Slett?id=" + id;
    $.get(url, function (OK) {
        if (OK) {
            window.location.href = 'index.html';
        }
        else {
            $("#feil").html("Feil i db - prøv igjen senere");
        }

    });
};
```

**endre.js**

```js
$(function () {
    // hent kunden med kunde-id fra url og vis denne i skjemaet

    const id = window.location.search.substring(1);
    const url = "Kunde/HentEn?" + id;
    $.get(url, function (kunde) {
        $("#id").val(kunde.id); // må ha med id inn skjemaet, hidden i html
        $("#fornavn").val(kunde.fornavn);
        $("#etternavn").val(kunde.etternavn);
        $("#adresse").val(kunde.adresse);
        $("#postnr").val(kunde.poststed.postnr);
        $("#poststed").val(kunde.poststed.postnavn);
    });
});

function endreKunde() {
    const poststed = {
        postnr: $("#postnr").val(),
        postnavn: $("#poststed").val()
    }
    const kunde = {
        id: $("#id").val(), // må ha med denne som ikke har blitt endret for å vite hvilken kunde som skal endres
        fornavn: $("#fornavn").val(),
        etternavn: $("#etternavn").val(),
        adresse: $("#adresse").val(),
        poststed: poststed
    }
    $.post("Kunde/Endre", kunde, function (OK) {
        if (OK) {
            window.location.href = 'index.html';
        }
        else {
            $("#feil").html("Feil i db - prøv igjen senere");
        }
    });
}
```

**lagre.js** 

```js
function lagreKunde() {
    const poststed = {
        postnr: $("#postnr").val(),
        postnavn: $("#poststed").val()
    }
    const kunde = {
        fornavn: $("#fornavn").val(),
        etternavn: $("#etternavn").val(),
        adresse: $("#adresse").val(),
        poststed: poststed
    }
    const url = "Kunde/Lagre";
    $.post(url, kunde, function (OK) {
        if (OK) {
            window.location.href = 'index.html';
        }
        else {
            $("#feil").html("Feil i db - prøv igjen senere");
        }
    });
};
```

### Endrede filer

**Startup.cs**

```cs
using KundeAppDAL.Models;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace KundeAppDAL
{
    public class Startup
    {
        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddDbContext<KundeContext>(options => options.UseSqlite("Data source=Kunde.db"));
            services.AddControllers().AddNewtonsoftJson(options =>
                options.SerializerSettings.ReferenceLoopHandling = Newtonsoft.Json.ReferenceLoopHandling.Ignore
            );
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
                DBInit.initialize(app);
            }

            app.UseStaticFiles();

            app.UseRouting();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}
```

I tillegg er `"launchUrl": "index.html"` lagt inn i Properties/launchSettings.json

## Endringer i filstruktur og nye filer

Lager ny mappe som vi kaller **DAL**. Dit flyttes det som har med databasen å gjøre; DBInit.cs og KundeContext.cs. I samme mappe oppreter vi to nye filer: KundeRepository.cs. og IKundeRepository.cs

Filstrukturen ser slik ut etter endringer:


- 📁 Controllers
    using KundeAppDAL.Models;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace KundeAppDAL
{
    public class Startup
    {
        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddDbContext<KundeContext>(options => options.UseSqlite("Data source=Kunde.db"));
            services.AddControllers().AddNewtonsoftJson(options =>
                options.SerializerSettings.ReferenceLoopHandling = Newtonsoft.Json.ReferenceLoopHandling.Ignore
            );
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
                DBInit.initialize(app);
            }

            app.UseStaticFiles();

            app.UseRouting();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}
- KundeController.cs

- 📁 DAL
    - DBInit.cs
    - KundeRepository.cs
    - KundeContext.cs
    - IKundeRepository.cs

- 📁 Models
    - Kunde.cs
    - Poststed.cs

I tillegg til filer for frontend; wwwroot osv.




**IKundeRepository.cs**

Grensesnittet skal inneholde alle de vanlige metodene (CRUD).

```

```


KundeRepository skal implementere grensesnittet. KundeController skrives om til å ta imot en variabel av type IKundeRepository. Kontrollerens metoder skal kalle på dette objektets metoder med samme navn.

**Startup.cs**

Til slutt må det legges til enda en service i Startup, unded ConfigureServices:

```cs
services.AddScoped<IKundeRepository, KundeRepository>();
```