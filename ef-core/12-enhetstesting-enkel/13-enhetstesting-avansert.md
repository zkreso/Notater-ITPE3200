# Enhetstesting (avansert)

## Oppsett

Skal vi teste en applikasjon med innlogging og feilhåndtering så krever det litt mer oppsett. Ved innlogging har en session, så vi må "fake" eller "mocke" en HttpSession. I tillegg har vi flere utfall i metodene når vi har login - de kan nå bl.a. returnere forskjellige statuskoder, null, booleans i ulike kombinasjoner, basert på login-status og om koden ble kjørt uten feil eller ikke. Ved feil vil metodene også skrive til loggen, noe som også bør testes.

### Felles variabler

Det er flere variabler som går igjen i hver av testene. Vi definerer derfor disse som atributter i toppen av testklassen.

```cs
private const string _loggetInn = "loggetInn";
private const string _ikkeLoggetInn = "";

private readonly Mock<IKundeRepository> mockRep = new Mock<IKundeRepository>();
private readonly Mock<ILogger<KundeController>> mockLog = new Mock<ILogger<KundeController>>();

private readonly Mock<HttpContext> mockHttpContext = new Mock<HttpContext>();
private readonly MockHttpSession mockSession = new MockHttpSession();
```

### Mock HttpSession

Vi må "fake" eller "mocke" en HttpSession. Vi lager en egen klasse for dette. Innholdet ble ikke forklart i videoen, klassen kan bare hentes ned fra nettet iflg. foreleser.

**MockHttpSession.cs**

```cs
namespace KundeAppTest
{
    public class MockHttpSession : ISession
    {
        Dictionary<string, object> sessionStorage = new Dictionary<string, object>();

        public object this[string name]
        {
            get { return sessionStorage[name]; }
            set { sessionStorage[name] = value; }
        }

        void ISession.Set(string key, byte[] value)
        {
            sessionStorage[key] = value;
        }

        bool ISession.TryGetValue(string key, out byte[] value)
        {
            if (sessionStorage[key] != null)
            {
                value = Encoding.ASCII.GetBytes(sessionStorage[key].ToString());
                return true;
            }
            else
            {
                value = null;
                return false;
            }
        }
        // det er noen flere metoder, men disse er ikke nødvendige iflg. foreleser
    }
}
```

### Eksempler

```cs
[Fact]
public async Task HentAlleLoggetInnOK()
{
    // Arrange
    var kunde1 = new Kunde {Id = 1,Fornavn = "Per",Etternavn = "Hansen",Adresse = "Askerveien 82",
                            Postnr = "1370",Poststed = "Asker"};
    var kunde2 = new Kunde {Id = 2,Fornavn = "Ole",Etternavn = "Olsen",Adresse = "Osloveien 82",
                            Postnr = "0270",Poststed = "Oslo"};
    var kunde3 = new Kunde {Id = 1,Fornavn = "Finn",Etternavn = "Finnsen",Adresse = "Bergensveien 82",
                            Postnr = "5000",Poststed = "Bergen"};

    var kundeListe = new List<Kunde>();
    kundeListe.Add(kunde1);
    kundeListe.Add(kunde2);
    kundeListe.Add(kunde3);

    mockRep.Setup(k => k.HentAlle()).ReturnsAsync(kundeListe);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _loggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.HentAlle() as OkObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.OK,resultat.StatusCode);
    Assert.Equal<List<Kunde>>((List<Kunde>)resultat.Value, kundeListe);
}

[Fact]
public async Task HentAlleIkkeLoggetInn()
{
    // Arrange

    //var tomListe = new List<Kunde>();

    mockRep.Setup(k => k.HentAlle()).ReturnsAsync(It.IsAny<List<Kunde>>());

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _ikkeLoggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.HentAlle() as UnauthorizedObjectResult;
    
    // Assert 
    Assert.Equal((int)HttpStatusCode.Unauthorized, resultat.StatusCode);
    Assert.Equal("Ikke logget inn", resultat.Value);
}

[Fact]
public async Task LagreLoggetInnOK()
{

    /*  Kan unngå denne med It.IsAny<Kunde>
    var kunde1 = new Kunde {Id = 1,Fornavn = "Per",Etternavn = "Hansen",Adresse = "Askerveien 82",
                            Postnr = "1370",Poststed = "Asker"};
    */

    // Arrange

    mockRep.Setup(k => k.Lagre(It.IsAny<Kunde>())).ReturnsAsync(true);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _loggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.Lagre(It.IsAny<Kunde>()) as OkObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.OK, resultat.StatusCode);
    Assert.Equal("Kunde lagret", resultat.Value);
}

[Fact]
public async Task LagreLoggetInnIkkeOK()
{
    // Arrange

    mockRep.Setup(k => k.Lagre(It.IsAny<Kunde>())).ReturnsAsync(false);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _loggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.Lagre(It.IsAny<Kunde>()) as BadRequestObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.BadRequest, resultat.StatusCode);
    Assert.Equal("Kunden kunne ikke lagres", resultat.Value);
}

[Fact]
public async Task LagreLoggetInnFeilModel()
{
    // Arrange
    // Kunden er indikert feil med tomt fornavn her.
    // det har ikke noe å si, det er introduksjonen med ModelError under som tvinger frem feilen
    // kunnde også her brukt It.IsAny<Kunde>
    var kunde1 = new Kunde {Id = 1,Fornavn = "",Etternavn = "Hansen",Adresse = "Askerveien 82",
                            Postnr = "1370",Poststed = "Asker"};

    mockRep.Setup(k => k.Lagre(kunde1)).ReturnsAsync(true);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    kundeController.ModelState.AddModelError("Fornavn", "Feil i inputvalidering på server");

    mockSession[_loggetInn] = _loggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.Lagre(kunde1) as BadRequestObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.BadRequest, resultat.StatusCode);
    Assert.Equal("Feil i inputvalidering på server", resultat.Value);
}

[Fact]
public async Task LagreIkkeLoggetInn()
{
    mockRep.Setup(k => k.Lagre(It.IsAny<Kunde>())).ReturnsAsync(true);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _ikkeLoggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.Lagre(It.IsAny<Kunde>()) as UnauthorizedObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.Unauthorized, resultat.StatusCode);
    Assert.Equal("Ikke logget inn", resultat.Value);
}

[Fact]
public async Task SlettLoggetInnOK()
{
    // Arrange

    mockRep.Setup(k => k.Slett(It.IsAny<int>())).ReturnsAsync(true);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _loggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.Slett(It.IsAny<int>()) as OkObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.OK, resultat.StatusCode);
    Assert.Equal("Kunde slettet", resultat.Value);
}

[Fact]
public async Task SlettLoggetInnIkkeOK()
{
    // Arrange

    mockRep.Setup(k => k.Slett(It.IsAny<int>())).ReturnsAsync(false);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _loggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.Slett(It.IsAny<int>()) as NotFoundObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.NotFound, resultat.StatusCode);
    Assert.Equal("Sletting av Kunden ble ikke utført", resultat.Value);
}

[Fact]
public async Task SletteIkkeLoggetInn()
{
    mockRep.Setup(k => k.Slett(It.IsAny<int>())).ReturnsAsync(true);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _ikkeLoggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.Slett(It.IsAny<int>()) as UnauthorizedObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.Unauthorized, resultat.StatusCode);
    Assert.Equal("Ikke logget inn", resultat.Value);
}

[Fact]
public async Task HentEnLoggetInnOK()
{
    // Arrange
    var kunde1 = new Kunde
    {
        Id = 1,
        Fornavn = "Per",
        Etternavn = "Hansen",
        Adresse = "Askerveien 82",
        Postnr = "1370",
        Poststed = "Asker"
    };

    mockRep.Setup(k => k.HentEn(It.IsAny<int>())).ReturnsAsync(kunde1);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _loggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.HentEn(It.IsAny<int>()) as OkObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.OK, resultat.StatusCode);
    Assert.Equal<Kunde>(kunde1,(Kunde)resultat.Value);
}

[Fact]
public async Task HentEnLoggetInnIkkeOK()
{
    // Arrange

    mockRep.Setup(k => k.HentEn(It.IsAny<int>())).ReturnsAsync(()=>null); // merk denne null setting!

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _loggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.HentEn(It.IsAny<int>()) as NotFoundObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.NotFound, resultat.StatusCode);
    Assert.Equal("Fant ikke kunden", resultat.Value);
}

[Fact]
public async Task HentEnIkkeLoggetInn()
{
    mockRep.Setup(k => k.HentEn(It.IsAny<int>())).ReturnsAsync(()=>null);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _ikkeLoggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.HentEn(It.IsAny<int>()) as UnauthorizedObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.Unauthorized, resultat.StatusCode);
    Assert.Equal("Ikke logget inn", resultat.Value);
}

[Fact]
public async Task EndreLoggetInnOK()
{
    // Arrange

    mockRep.Setup(k => k.Endre(It.IsAny<Kunde>())).ReturnsAsync(true);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _loggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.Endre(It.IsAny<Kunde>()) as OkObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.OK, resultat.StatusCode);
    Assert.Equal("Kunde endret", resultat.Value);
}

[Fact]
public async Task EndreLoggetInnIkkeOK()
{
    // Arrange

    mockRep.Setup(k => k.Lagre(It.IsAny<Kunde>())).ReturnsAsync(false);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _loggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.Endre(It.IsAny<Kunde>()) as NotFoundObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.NotFound, resultat.StatusCode);
    Assert.Equal("Endringen av kunden kunne ikke utføres", resultat.Value);
}

[Fact]
public async Task EndreLoggetInnFeilModel()
{
    // Arrange
    // Kunden er indikert feil med tomt fornavn her.
    // det har ikke noe å si, det er introduksjonen med ModelError under som tvinger frem feilen
    // kunnde også her brukt It.IsAny<Kunde>
    var kunde1 = new Kunde
    {
        Id = 1,
        Fornavn = "",
        Etternavn = "Hansen",
        Adresse = "Askerveien 82",
        Postnr = "1370",
        Poststed = "Asker"
    };

    mockRep.Setup(k => k.Endre(kunde1)).ReturnsAsync(true);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    kundeController.ModelState.AddModelError("Fornavn", "Feil i inputvalidering på server");

    mockSession[_loggetInn] = _loggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.Endre(kunde1) as BadRequestObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.BadRequest, resultat.StatusCode);
    Assert.Equal("Feil i inputvalidering på server", resultat.Value);
}

[Fact]
public async Task EndreIkkeLoggetInn()
{
    mockRep.Setup(k => k.Endre(It.IsAny<Kunde>())).ReturnsAsync(true);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _ikkeLoggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.Endre(It.IsAny<Kunde>()) as UnauthorizedObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.Unauthorized, resultat.StatusCode);
    Assert.Equal("Ikke logget inn", resultat.Value);
}

[Fact]
public async Task LoggInnOK()
{
    mockRep.Setup(k => k.LoggInn(It.IsAny<Bruker>())).ReturnsAsync(true);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _loggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.LoggInn(It.IsAny<Bruker>()) as OkObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.OK, resultat.StatusCode);
    Assert.True((bool)resultat.Value);
}

[Fact]
public async Task LoggInnFeilPassordEllerBruker()
{
    mockRep.Setup(k => k.LoggInn(It.IsAny<Bruker>())).ReturnsAsync(false);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    mockSession[_loggetInn] = _ikkeLoggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.LoggInn(It.IsAny<Bruker>()) as OkObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.OK, resultat.StatusCode);
    Assert.False((bool)resultat.Value);
}

[Fact]
public async Task LoggInnInputFeil()
{
    mockRep.Setup(k => k.LoggInn(It.IsAny<Bruker>())).ReturnsAsync(true);

    var kundeController = new KundeController(mockRep.Object, mockLog.Object);

    kundeController.ModelState.AddModelError("Brukernavn", "Feil i inputvalidering på server");

    mockSession[_loggetInn] = _loggetInn;
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;

    // Act
    var resultat = await kundeController.LoggInn(It.IsAny<Bruker>()) as BadRequestObjectResult;

    // Assert 
    Assert.Equal((int)HttpStatusCode.BadRequest, resultat.StatusCode);
    Assert.Equal("Feil i inputvalidering på server", resultat.Value);
}

[Fact]
public void LoggUt()
{
    var kundeController = new KundeController(mockRep.Object, mockLog.Object);
    
    mockHttpContext.Setup(s => s.Session).Returns(mockSession);
    mockSession[_loggetInn] = _loggetInn;
    kundeController.ControllerContext.HttpContext = mockHttpContext.Object;
    
    // Act
    kundeController.LoggUt();

    // Assert
    Assert.Equal(_ikkeLoggetInn,mockSession[_loggetInn]);
}
```