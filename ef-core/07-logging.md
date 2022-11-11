# Logging

## Oppsett

Logging til fil er ikke innebygd i EF Core. Vi kan bruke pakker, bruker her:

**Serilog.Extensions.Logging.File**

Klasser som skal bruke denne må importere `using Microsoft.Extensions.Logging`.

### Oppsett i startup.cs

Configure metoden skal ta et ILoggerFactory objekt. I tillegg må vi konfigurere hvilken fil den skal bruke

```cs
public void Configure( ... , ILoggerFactory loggerFactory ) {
    ...
    loggerFactory.AddFile("Logs/Kundelog.txt");
    ...
}
```

### Oppsett i Controller

Loggeren må injiseres i Controller klassen. Det opprettes en variabel og den legges til i konstruktøren slik at den blir opprettet ved instansiering

```cs
public class KundeController : base {
    ...
    private readonly ILogger<KundeController> _log;
    ...
    public KundeController( ... , ILogger<KundeController> log) {
        ...
        log = _log;
    }
    ...
}
```
## Bruk

Med oppsettet ovenfor så vil det lages en fil pr. dato i Logs mappen (denne vil bli opprettet hvis den ikke er det til å begynne med). Den vil inneholde all informasjon som logges.

Man kan teste loggingen ved å skrive

```cs
_log.LogInformation("Hallo loggen!");
```