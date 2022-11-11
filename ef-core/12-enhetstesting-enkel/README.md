# Enhetstesting (enkel)

## Oppsett

### Lage testprosjekt

Når vi skal teste så lager vi et nytt prosjekt bare for testing. Prosjektet opprettes i samme solution, ved å høyreklikke på Solution (roten av treet) i Solution Explorer vinduet og velge **Add -> New Project...**

Vi velger malen "**xUnit Test Project (.NET Core)**". Malen inneholder en klasse som godt kan døpes om til noe mer passende, f.eks. KundeTest (hvis vi tester KundeApp).

### Legge til referanse

For å få tilgang til klassene i hovedprosjektet må vi legge til referanse til hovedprosjektet. Det kan gjøres ved å gå inn på **Project -> Add Reference**. Eventuelt så får man også forslag om dette under "show potential fixes" når vi prøver å bruke en klasse fra hovedprosjektet for første gang.

### Pakke for testing

I tillegg så trenger vi å installere en pakke for unit testing. Vi bruker pakken fra Moq:

- **Moq.EntityFrameWorkCore**

Som alltid - pass på å velge akkurat denne. Det er flere som heter noe som ligner.

## Design av testmetoder

Benytter testmetoden for opprettelse av ny kunde i KundeApp eksempelet som et eksempel:

**Lagre()**
```cs
namespace XUnitTestProject1
{
    public class KundeTest
    {
        [Fact]
        public async Task Lagre()
        {
            // Arrange
            var innKunde = new Kunde
            {
                Id = 1,
                Fornavn = "Per",
                Etternavn = "Hansen",
                Adresse = "Osloveien 82",
                Postnr = "1234",
                Poststed = "Oslo"
            };

            var mock = new Mock<IKundeRepository>();
            mock.Setup(k => k.Lagre(innKunde)).ReturnsAsync(true);
            var kundeController = new KundeController(mock.Object);

            // Act
            bool resultat = await kundeController.Lagre(innKunde);

            // Assert
            Assert.True(resultat);
        }
    }
}
```

Vi merker oss følgende:

- Testmetoder skal ha annotasjonen `[Fact]`
- Testmetoder skal ikke returnere noe. Men siden de er asynkrone så må man skrive async Task i stedet for void. Async task er det samme som void, bare for asynkrone metoder.
- Testen består av tre deler:
    - Arrange: Der vi setter opp testen
    - Act: Der testen utføres
        - Linjene for initialisering av mock og controller vil være de samme uansett utformingen av testen
        - Linjen for setup av mock vil variere. Vi må bestemme hva testen skal returnere. Det bør nødvendigvis være data av samme type som metoden vi tester returnerer, og med den verdien som vi forventer. Merk at return-kallet er asynkront.
        - Merk at return metoden kalles "etter" Setup() kallet, ikke inni.
    - Assert: Der resultatet av testen kontrolleres
        - Assert har metoder for å kontrollere flere typer av resultat. Det kan kontrollere nullresultat, sammenligne to resultater osv. Mer om det i senere eksempler.

### Flere eksempler

**HentAlle()**

```cs
[Fact]
public async Task HentAlle()
{
    var kunde1 = new Kunde
    {
        Id = 1,
        Fornavn = "Per",
        Etternavn = "Hansen",
        Adresse = "Askerveien 82",
        Postnr = "1370",
        Poststed = "Asker"
    };
    var kunde2 = new Kunde
    {
        Id = 2,
        Fornavn = "Ole",
        Etternavn = "Olsen",
        Adresse = "Osloveien 82",
        Postnr = "0270",
        Poststed = "Oslo"
    };
    var kunde3 = new Kunde
    {
        Id = 1,
        Fornavn = "Finn",
        Etternavn = "Finnsen",
        Adresse = "Bergensveien 82",
        Postnr = "5000",
        Poststed = "Bergen"
    };

    var kundeListe = new List<Kunde>();
    kundeListe.Add(kunde1);
    kundeListe.Add(kunde2);
    kundeListe.Add(kunde3);

    var mock = new Mock<IKundeRepository>();
    mock.Setup(k => k.HentAlle()).ReturnsAsync(kundeListe);
    var kundeController = new KundeController(mock.Object);

    // Act
    List<Kunde> resultat = await kundeController.HentAlle();

    // Assert
    Assert.Equal<List<Kunde>>(kundeListe,resultat);
}
```

Her ser vi et eksempel på hvordan setup metoden og resultat vil variere basert på hva vi tester.
- Vi angir under setup at HentAlle() skal returnere en type av `List<Kunde>`, med innhold de tre kundene vi har definert.
- Under "Act" så vet vi at resultatet vil være av typen `List<Kunde>` (men selvfølgelig ikke innholdet)
- Vi ser også et eksempel på hvordan Assert sin Equal metode brukes


**HentEn()**

```cs
[Fact]
public async Task HentEn()
{
    var returKunde = new Kunde
    {
        Id = 1,
        Fornavn = "Per",
        Etternavn = "Hansen",
        Adresse = "Askerveien 82",
        Postnr = "1370",
        Poststed = "Asker"
    };
    var mock = new Mock<IKundeRepository>();
    mock.Setup(k => k.HentEn(1)).ReturnsAsync(returKunde);
    var kundeController = new KundeController(mock.Object);
    
    // Act
    Kunde resultat = await kundeController.HentEn(1);
    
    // Assert
    Assert.Equal<Kunde>(returKunde, resultat);
}
```

Her ser vi et eksempel på at vi kan teste metoder som tar inn parametre.

**Slett()**

```cs
[Fact]
public async Task Slett()
{
    var mock = new Mock<IKundeRepository>();
    mock.Setup(k => k.Slett(1)).ReturnsAsync(true);
    var kundeController = new KundeController(mock.Object);

    //Act
    bool resultat = await kundeController.Slett(1);
    
    // Assert
    Assert.True(resultat);
}
```

Her ser vi et eksempel på at vi ikke trenger å opprette objekter for alle metoder vi tester. Slett metoden tar en parameter, men det er ikke nødvendig å opprette en kunde med id 1.

**Endre()**

```cs
[Fact]
public async Task Endre()
{
    var innKunde = new Kunde
    {
        Id = 1,
        Fornavn = "Per",
        Etternavn = "Hansen",
        Adresse = "Askerveien 82",
        Postnr = "1370",
        Poststed = "Asker"
    };
    var mock = new Mock<IKundeRepository>();
    mock.Setup(k => k.Endre(innKunde)).ReturnsAsync(true);
    var kundeController = new KundeController(mock.Object);

    // Act
    bool resultat = await kundeController.Endre(innKunde);

    // Assert
    Assert.True(resultat);
}
```

Det er ingen store forskjeller mellom Endre() og Lagre()

## Testing av alternative utfall

Metodene vi tester er ofte (og i alle fall i dette eksempelet) kodet slik at de vil kunne returnere flere verdier. F.eks. hvis noe går feil så vil ofte en metode som returnerer bool returnere false og ikke true. Metoder som returnerer objekter vil ofte være kodet slik at de returnerer null når noe går feil eller hvis det ikke finnes noe objekt som kan returneres. Vi bør teste for alle utfall. Dvs. at vi må lage en test pr. utfall.

### Eksempler med alternative utfall

**HentAlleTomListe()**

```cs
[Fact]
public async Task HentAlleTomListe()
{
    var kundeListe = new List<Kunde>();
    
    var mock = new Mock<IKundeRepository>();
    mock.Setup(k => k.HentAlle()).ReturnsAsync(()=>null);
    var kundeController = new KundeController(mock.Object);

    // Act
    List<Kunde> resultat = await kundeController.HentAlle();

    // Assert
    Assert.Null(resultat);
}
```
Her ser vi hvordan vi setter opp en test som skal returnere null.
- Man skulle kanskje tro at det holdt bare å skrive null i ReturnAsync(), men i Moq må man bruke denne litt spesielle konstruksjonen `() => null` i stedet.
- I tillegg ser vi at Assert har en egen metode for null.

**LagreIkkeOK()**

```cs
[Fact]
public async Task LagreIkkeOK()
{
    var innKunde = new Kunde
    {
        Id = 1,
        Fornavn = "Per",
        Etternavn = "Hansen",
        Adresse = "Askerveien 82",
        Postnr = "1370",
        Poststed = "Asker"
    };
    var mock = new Mock<IKundeRepository>();
    mock.Setup(k => k.Lagre(innKunde)).ReturnsAsync(false);
    var kundeController = new KundeController(mock.Object);

    // Act
    bool resultat = await kundeController.Lagre(innKunde);

    // Assert
    Assert.False(resultat);
}
```

Å teste for lagring som ikke gikk bra blir nesten det samme som å teste for lagring som gikk bra. Det eneste vi endrer på er mock.Setup, fordi vi forventer at resultatet skal være false. Vi ser også at Assert har en egen metode for False.

**HentEnIkkeOK()**

```cs
[Fact]
public async Task HentEnIkkeOK()
{
    var mock = new Mock<IKundeRepository>();
    mock.Setup(k => k.HentEn(1)).ReturnsAsync(()=>null);
    var kundeController = new KundeController(mock.Object);

    // Act
    Kunde resultat = await kundeController.HentEn(1);

    // Assert
    Assert.Null(resultat);
}
```

Hvis det ikke gikk bra å hente ut en kunde så returnerer metoden null. Vi bruker igjen samme konstruksjonen som vist tidligere. Vi ser igjen at vi ikke trenger å opprette objekter for å utføre alle tester, selv om metoden vi tester tar en parameter.

**SlettIkkeOK()**

```cs
[Fact]
public async Task SlettIkkeOK()
{
    var mock = new Mock<IKundeRepository>();
    mock.Setup(k => k.Slett(1)).ReturnsAsync(false);
    var kundeController = new KundeController(mock.Object);

    // Act
    bool resultat = await kundeController.Slett(1);

    // Assert
    Assert.False(resultat);
}
```

Det er ingen betydelige forskjeller mellom testen for vellykket sletting og sletting som ikke var vellykket.

**EndreIkkeOK()**

```cs
[Fact]
public async Task EndreIkkeOK()
{
    var innKunde = new Kunde
    {
        Id = 1,
        Fornavn = "Per",
        Etternavn = "Hansen",
        Adresse = "Askerveien 82",
        Postnr = "1370",
        Poststed = "Asker"
    };
    var mock = new Mock<IKundeRepository>();
    mock.Setup(k => k.Endre(innKunde)).ReturnsAsync(false);
    var kundeController = new KundeController(mock.Object);
    
    // Act
    bool resultat = await kundeController.Endre(innKunde);
    
    // Assert
    Assert.False(resultat);
}
```

Det er heller ingen betydelige forskjeller mellom testen for vellykket endring og endring som ikke var vellykket.