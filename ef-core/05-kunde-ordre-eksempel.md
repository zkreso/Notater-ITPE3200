# Kunde Ordre eksempel

I dette eksempelet skal vi oppprette er mer komplisert databasestruktur, med fire forskjellige tabeller:

**Kunde**
- Id (PK)
- Navn

**Ordre**
- Id (PK)
- Dato
- Kunde_Id (FK)

**OrdreLinje**
- Id (PK)
- Antall
- Ordre_Id (FK)
- Vare-Id (FK)

**Vare**
- Id (PK)
- Navn
- Pris

## Controller

Vi begynner med controlleren. I dette eksempelet skal vi ikke kode muligheter for å endre på data, så vi lager en kontroller med kun en enkel metode for å hente ut de dataene som ligger lagret. Metoden heter index og blir mappet til "home/index"

**HomeController.cs**

```cs
namespace KundeOrdre.Controllers
{
    [Route("[controller]/[action]")]
    public class HomeController : ControllerBase
    {
        private readonly DB _db;

        public HomeController(DB db)
        {
            _db = db;
        }

        public List<Kunde> index()
        {
            return _db.Kunder.ToList();
        }
    }
}
```

## Startup.cs

I Startup så legger vi til de "vanlige" linjene;

- `services.AddDbContext<DB>(options => options.UseSqlite("Data Source=Kunde.db"));` i ConfigureServices() metoden
- `app.UseStaticFiles();` i Configure() metoden
- Vi erstatter nok en gang endpointet i UseEndpoints metoden med `endpoints.MapControllers();`

Det nye her blir at vi benytter en annen pakke for å konvertere til json enn den som er installert som standard. Json-strukturen blir litt for komplisert for standard pakken. Vi skriver følgende i ConfigureServices for å ta i bruk denne:

```cs
services.AddControllers().AddNewtonsoftJson(options =>
        options.SerializerSettings.ReferenceLoopHandling = Newtonsoft.Json.ReferenceLoopHandling.Ignore
);
```

Pakken vi trenger heter **Microsoft.AspNet.Mvc.Newtonsoftjson**

**Startup.cs**

```cs
namespace KundeOrdre
{
    public class Startup
    {
        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers().AddNewtonsoftJson(options =>
                    options.SerializerSettings.ReferenceLoopHandling = Newtonsoft.Json.ReferenceLoopHandling.Ignore
            );
            services.AddDbContext<DB>(options => options.UseSqlite("Data Source=Kunde.db"));
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
                DBInit.init(app);
            }

            app.UseRouting();

            app.UseStaticFiles();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllers();
            });
        }
    }
}
```

## Tabeller

Vi oppretter en klasse pr. tabell i databsen. Vi tar også med atributter som ikke er i den tilhørende tabellen i databasen når vi ønsker at det skal være mulig å gå fra en tabell til den andre. F.eks. så ønsker vi å gå fra en kunde til denne kundens ordrer, så tar vi med `List<Ordre>` i Kunde klassen. Da må vi bruke virtual kodeordet.

**Kunde.cs**

```cs
namespace KundeOrdre.Models
{
    public class Kunde
    {
        public int Id { get; set; }
        public string Navn { get; set; }
        public virtual List<Ordre> Ordre { get; set; }
    }
}
```

**Ordre.cs**

```cs
namespace KundeOrdre.Models
{
    public class Ordre
    {
        public int Id { get; set; }
        public string Dato { get; set; }
        public virtual Kunde Kunde { get; set; }
        public virtual List<OrdreLinje> OrdreLinjer { get; set; }
    }
}
```

**OrdreLinje.cs**

```cs
namespace KundeOrdre.Models
{
    public class OrdreLinje
    {
        public int Id { get; set; }
        public int Antall { get; set; }
        public virtual Vare Vare { get; set; }
        public virtual Ordre Ordre { get; set; }
    }
}
```

**Vare.cs**

```cs
namespace KundeOrdre.Models
{
    public class Vare
    {
        public int Id { get; set; }
        public string Navn { get; set; }
        public double Pris { get; set; }
        public virtual List<OrdreLinje> OrdreLinjer { get; set; }
    }
}
```

## Database

Databasen tar i bruk lazy loading, så vi bruker samme kode som tidligere for å få dette til.

**DB.cs**

```cs
namespace KundeOrdre.Models
{
    public class DB : DbContext
    {
        public DB(DbContextOptions<DB> options) : base(options)
        {
            Database.EnsureCreated();
        }

        public virtual DbSet<Vare> Varer { get; set; }
        public virtual DbSet<Kunde> Kunder { get; set; }
        public virtual DbSet<Ordre> Ordre { get; set; }
        public virtual DbSet<OrdreLinje> OrdreLinjer { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseLazyLoadingProxies();
        }
    }
}
```

Vi ser at database-klassen er forholdsvis enkel. Det er hvordan vi koder klassene som representerer tabellene som bestemmer hvordan databasen blir seende ut, og relasjonen mellom de ulike tabellene.

Til slutt så lager vi en init fil med testdata slik at vi får testet løsningen

**DBInit.cs**

```cs
namespace KundeOrdre.Models
{
    public class DBInit
    {
        public static void init(IApplicationBuilder app)
        {
            using (var serviceScope = app.ApplicationServices.CreateScope())
            {
                var context = serviceScope.ServiceProvider.GetService<DB>();

                context.Database.EnsureDeleted();
                context.Database.EnsureCreated();

                var nyKunde = new Kunde { Navn = "Ole Hansen" };
                var nyOrdre = new Ordre { Dato = "23.05.2017" };
                var nyVare1 = new Vare { Pris = 2.34, Navn = "Mutter 3mm" };
                var nyVare2 = new Vare { Pris = 3.34, Navn = "Mutter 4mm" };
                var nyOrdreLinje1 = new OrdreLinje { Vare = nyVare1, Antall = 100 };
                var nyOrdreLinje2 = new OrdreLinje { Vare = nyVare2, Antall = 50 };
                
                var nyeOrdreLinjer = new List<OrdreLinje>();
                nyeOrdreLinjer.Add(nyOrdreLinje1);
                nyeOrdreLinjer.Add(nyOrdreLinje2);

                nyOrdre.OrdreLinjer = nyeOrdreLinjer;

                var nyeOrdre = new List<Ordre>();
                nyeOrdre.Add(nyOrdre);
                nyKunde.Ordre = nyeOrdre;

                context.Kunder.Add(nyKunde);
                context.SaveChanges();
            }
        }
    }
}
```

## Klientsiden

Klientsiden består kun en funksjon for å skrive ut mottatt data:

**index.js**

```js
$(function () {
    hentAlleKunder();
})

function hentAlleKunder() {
    $.get("home/index", function (kunder) {
        skrivUt(kunder);
    });
}

function skrivUt(kunder) {
    for (let kunde of kunder) {
        document.write("Kundenavn: " + kunde.navn + "<br>");
        for (let ordre of kunde.ordre) {
            document.write("Ordre-dato: " + ordre.dato + "<br>");
            for (let ordreLinje of ordre.ordreLinjer) {
                document.write("Ordrelinje-antall: " + ordreLinje.antall + "<br>");
                document.write("Vare-navn: " + ordreLinje.vare.navn + "<br>");
            }
        }
    }
}
```

**index.html**

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>KundeOrdre</title>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
    <script src="js/index.js"></script>
</head>
<body>

</body>
</html>
```

I tillegg må som vanlig launchurl settes til index.html i application settings.