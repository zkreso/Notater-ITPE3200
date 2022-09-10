# Kunde eksempel videreført

Vi skal nå utvide kunde eksempelet ved å legge til poststed og postnummer. I tillegg ønsker vi at postnummer og poststed skal ligge i egen tabell, for å opprettholde 3. normalform på databasen.

## Tjenersiden

### Endringer i Kunde.cs

Vi starter med å legge til de nye attributtene Kunde skal ha i Kunde.cs:

```cs
public class Kunde
{
    public int id {  get; set; }
    public string Fornavn { get; set; }
    public string Etternavn { get; set; }
    public string Adresse { get; set; }
    public string Postnr { get; set; }
    public string Poststed { get; set; }
}
```

Kunde.cs er den "flate" strukturen som overføres mellom klient og tjener.

### Endringer i KundeDB / KundeContext

I KundeDB.cs (i videon er filnavnet endret til KundeContext.cs) så må vi gjøre en del endringer. Vi ønsker å ha en tabell for Kunde og en tabell for Poststed i databasen, så vi begynner med å lage to indre klasser for dette. (Klassene kunne også ha blitt plassert opprettet som egne filer, men det er vanlig å legge dem inn i Context klassen). I Kunder klassen bruker vi kodeordet virtual foran poststed. Dette gjør at vi automatisk får med poststedet når vi spør om en Kunde - såkalt "lazy loading"

```cs
public class Kunder
    {
        public int id { get; set; }
        public string Fornavn { get; set; }
        public string Etternavn { get; set; }
        public string Adresse { get; set; }
        virtual public Poststeder Poststed { get; set; }
    }
```

I Poststeder objektet så ønsker vi å bruke postnummer som nøkkel. Da må vi overstyre automatisk opprettelse av denne. Dette gjør vi ved å ta i bruk et par dekoratører. Disse krever at vi tar i bruk biblioteket System.ComponentModel.DataAnnotations.Schema;

```cs
using System.ComponentModel.DataAnnotations.Schema;
```

```cs
public class Poststeder
    {
        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.None)]
        public string Postnr { get; set; }
        public string Poststed { get; set; }
    }
```

Til slutt så må vi legge til en override i KundeContext klassen for å introdusere bruk av lazy loadingen som vi har i Kunder klassen. Vi må også opprette tabellen for poststeder og kunder i denne klassen. (Vi fjerner opprettelse av Kunde som å der tidligere, da denne klassen ikke står for opprettelse av databasen lenger.)

```cs
public class KundeContext : DbContext
    {
        public KundeContext (DbContextOptions<KundeContext> options) : base(options)
        {
            Database.EnsureCreated();
        }

        public DbSet<Kunder> Kunder { get; set; }

        public DbSet<Poststeder> Poststeder { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseLazyLoadingProxies();
        }
    }
```

Denne siste koden krever at vi installerer pakken **Microsoft.EntityFrameworkCore.Proxies**

### Oppdatering av metoder i KundeController

Vi må endre på alle metodene i kontrolleren, fordi vi nå forholder oss til to tabeller:

```cs
public async Task<bool> Lagre(Kunde innKunde)
{
    try
    {
        var nyKundeRad = new Kunder();
        nyKundeRad.Fornavn = innKunde.Fornavn;
        nyKundeRad.Etternavn = innKunde.Etternavn;
        nyKundeRad.Adresse = innKunde.Adresse;

        var sjekkPostSted = _db.Poststeder.Find(innKunde.Postnr);
        if (sjekkPostSted == null)
        {
            var nyPoststedsRad = new Poststeder();
            nyPoststedsRad.Postnr = innKunde.Postnr;
            nyPoststedsRad.Poststed = innKunde.Poststed;
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

public async Task<bool> Slett(int id)
{
    try
    {
        Kunder enKunde = await _db.Kunder.FindAsync(id);
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
            Postnr = k.Poststed.Postnr,
            Poststed = k.Poststed.Poststed
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
        Kunder enKunde = await _db.Kunder.FindAsync(id);
        var hentetKunde = new Kunde()
        {
            id = enKunde.id,
            Fornavn = enKunde.Fornavn,
            Etternavn = enKunde.Etternavn,
            Adresse = enKunde.Adresse,
            Postnr = enKunde.Poststed.Postnr,
            Poststed = enKunde.Poststed.Poststed
        };
        return hentetKunde;
    }
    catch
    {
        return null;
    }
}

public async Task<bool> Endre(Kunde endreKunde)
{
    try
    {
        Kunder enKunde = await _db.Kunder.FindAsync(endreKunde.id);

        // hvis postnummeret er endret
        if(enKunde.Poststed.Postnr != endreKunde.Postnr)
        {
            // hvis postnummeret ikke finnes
            var sjekkPostSted = _db.Poststeder.Find(endreKunde.Postnr);
            if (sjekkPostSted == null)
            {
                var nyPoststedsRad = new Poststeder();
                nyPoststedsRad.Postnr = endreKunde.Postnr;
                nyPoststedsRad.Poststed = endreKunde.Poststed;
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
```

## Klientsiden

**index.js** er utvidet med de nye atributtene:

```js
for (let kunde of kunder) {
        ut += "<tr>" +
            "<td>" + kunde.fornavn + "</td>" +
            "<td>" + kunde.etternavn + "</td>" +
            "<td>" + kunde.adresse + "</td>" +
            "<td>" + kunde.postnr + "</td>" +
            "<td>" + kunde.poststed + "</td>" +
            "<td> <a class='btn btn-primary' href='endre.html?id=" + kunde.id + "'>Endre</a></td>" +
            "<td> <button class='btn btn-danger' onclick='slettKunde(" + kunde.id + ")'>Slett</button></td>" +
            "</tr>";
    }
```

Det samme er lagre.js og endre.js:

**lagre.js**

```js
const kunde = {
    fornnavn: $("#fornavn").val(),
    etternavn: $("#etternavn").val(),
    adresse: $("#adresse").val(),
    postnr: $("#postnr").val(),
    poststed: $("#poststed").val()
}
```

**endre.js**

```js
$.get(url, function (kunde) {
        $("#id").val(kunde.id); // må ha med id inn skjemaet, hidden i html
        $("#fornavn").val(kunde.fornavn);
        $("#etternavn").val(kunde.etternavn);
        $("#adresse").val(kunde.adresse);
        $("#postnr").val(kunde.postnr);
        $("#poststed").val(kunde.poststed);
    });

...

const kunde = {
    id: $("#id").val(),
    fornavn: $("#fornavn").val(),
    etternavn: $("#etternavn").val(),
    adresse: $("#adresse").val(),
    postnr: $("#postnr").val(),
    poststed: $("#poststed").val()
}
```

**endre.html** og **lagre.html** er utvidet med de nye feltene:

```html
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
```

## Seeding av databasen

Nå skal vi vise hvordan vi kan starte med en database som inneholder data når den blir opprettet.

Vi oppretter en ny klasse, den kan hete hva som helst og ligge hvor som helst, men det er greit å kalle den noe som gjør at vi skjønner hva den gjør. Vi kaller den derfor DBInit og plasserer den under Model

**DBInit.cs**

```cs
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;

namespace KundeApp3.Model
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

                var poststed1 = new Poststeder { Postnr = "0270", Poststed = "Oslo" };
                var poststed2 = new Poststeder { Postnr = "1370", Poststed = "Asker" };

                var kunde1 = new Kunder { Fornavn = "Ole", Etternavn = "Hansen", Adresse = "Osloveien 82", Poststed = poststed1 };
                var kunde2 = new Kunder { Fornave = "Line", Etternavn = "Jensen", Adresse = "Askerveien 72", Poststed = poststed2 };

                context.Kunder.Add(kunde1);
                context.Kunder.Add(kunde2);

                context.SaveChanges();
            }
        }
    }
}
```

initialize metoden sletter databsen og lager ny database hver gang, hvor den legger inn verdiene og lagrer endringer. Det er ikke nødvendig med async da dette gjøres kun en gang og ved første kjøring av programmet.

I Startup.cs så må vi kaller på DBInit sin initialize. Det gjør vi i Configure metoden;

```cs
if (env.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    DBInit.initialize(app); // ny linje
}
```

Vi kan kommentere ut linjen dersom vi ikke ønsker at databasen skal overskrives hver gang.