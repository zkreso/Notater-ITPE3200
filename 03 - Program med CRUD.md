# Program med CRUD

Vi skal nå utvide programmet vårt slik at vi kan manipulere databasen direkte fra nettleseren. Vi tar utgangspunkt i programmet med database fra forrige del. Repetisjon av hva som er gjort for å opprette kode hittil:

1. Opprette nytt prosjekt ut i fra en tom mal: **ASP.NET Core Empty**
2. Opprette modeller (Kunde.cs og KundeDB.cs) og kontroller (KundeController.cs) i tilhørende mapper ( henholdsvis mappen Models og mappen Controllers )
3. Opprette index.html og index.js og tilhørende mapper ( wwwroot og wwwroot/js )
4. Installert følgende packages:
    - Microsoft.EntityFrameworkCore 
    - Microsoft.EntityFrameworkCore.Sqlite
4. I Startup.cs:
    - Legge til `services.AddControllers();` i ConfigureSerices metoden
    - Legge til `services.AddDbContext<KundeDB>(options => options.UseSqlite("Data source=Kunde.db"));` i ConfigureServices metoden
    - Legge til `app.UseStaticFiles();` i Configure metoden
    - Erstatte koden i app.UseEndpoints kallet med `endpoints.MapControllers();`
5. Sette launchUrl til index.html i Properties/launchSettings.json

## Nye metoder på tjenersiden

Vi må opprette metoder for å lagre, endre og slette data. Dette gjør vi i **KundeController.cs**:

```cs
public bool Lagre(Kunde innKunde) {
    try
    {
        _kundeDB.Kunder.Add(innKunde);
        _kundeDB.SaveChanges();
        return true;
    }
    catch
    {
        return false;
    }
}

public bool Slett(int id)
{
    try
    {
        Kunde enKunde = _kundeDB.Kunder.Find(id);
        _kundeDB.Kunder.Remove(enKunde);
        _kundeDB.SaveChanges();
        return true;
    }
    catch
    {
        return false;
    }
}

public Kunde HentEn(int id)
{
    try
    {
        Kunde enKunde = _kundeDB.Kunder.Find(id);
        return enKunde;
    }
    catch
    {
        return null;
    }
}

public bool Endre(Kunde endreKunde)
{
    try
    {
        Kunde enKunde = _kundeDB.Kunder.Find(endreKunde.id);
        enKunde.navn = endreKunde.navn;
        enKunde.adresse = endreKunde.adresse;
        _kundeDB.SaveChanges();
        return true;
    }
    catch
    {
        return false;
    }
}
```


## Nye filer på klientsiden

Vi kan så kalle disse metodene fra nettleseren. Vi trenger et brukergrensesnitt og oppretter derfor følgende filer:

- lagre.html og endre.html ( i wwwroot mappen )
- lagre.js og endre.js ( i wwwroot/js mappen )

I tillegg så legger vi til en slett metode i den eksisterende index.js filen

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
                <label>Navn</label>
                <input type="text" id="navn" />
            </div>
            <div class="form-group">
                <label>Adresse</label>
                <input type="text" id="adresse" />
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
                <label>Navn</label>
                <input type="text" id="navn" />
            </div>
            <div class="form-group">
                <label>Adresse</label>
                <input type="text" id="adresse" />
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

**lagre.js**

```js
function lagreKunde() {
    const kunde = {
        navn: $("#navn").val(),
        adresse: $("#adresse").val()
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

**endre.js**

```js
$(function () {
    // hent kunden med kunde-id fra url og vis denne i skjemaet

    const id = window.location.search.substring(1);
    const url = "Kunde/HentEn?" + id;
    $.get(url, function (kunde) {
        $("#id").val(kunde.id); // må ha med id inn skjemaet, hidden i html
        $("#navn").val(kunde.navn);
        $("#adresse").val(kunde.adresse);
    });
});

function endreKunde() {
    const kunde = {
        id: $("#id").val(), // må ha med denne som ikke har blitt endret for å vite hvilken kunde som skal endres
        navn: $("#navn").val(),
        adresse: $("#adresse").val()
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

**ny metode i index.js**

```js
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
