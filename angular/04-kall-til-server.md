# Kall til server

For å gjøre kall til server trenger i en kontroller. Vi bruker dekoratøren `"api/[controller]"`. Dette gir oss et REST api. Vi trenger selvfølgelig ikke bruke et REST api, da kan vi f.eks. bruke den type dekoratør som vi hadde i del 1 i kurset `"[controller]/[action]"`

NB: `[ApiController]` dekoratøren må også være med. Han nevner ikke det i videoen.

I et REST api har http metoden betydning. Det er ikke likegyldig om man bruker GET eller POST osv. Vi må bestemme hvilken http metode funksjonene i controlleren skal kunne nås med. For en metode som f.eks. returnerer en liste med kunder vil det være naturlig at denne kan nås ved å bruke GET. Det gjøres ved å bruke dekoratøren `[HttpGet]` foran metoden **og/eller** så må metoden hete noe med "Get". Bortsett fra det kan kontrolleren utformes hvordan man ønsker. Limer ikke inn noe eksempel på det her.

**app.component.html** kan f.eks. se slik ut:

```html
<div>
    <h1>Kunder</h1>
    <button (click)="hentAlleKunder()">Hent kunder</button> {{laster}}
    <ul>
        <li *ngFor="let kunde of alleKunder">
            {{kunde.id}} : {{kunde.fornavn}} {{kunde.etternavn}}
            {{kunde.adresse}} {{kunde.postmummer}} {{kunde.poststed}}
        </li>
    </ul>
</div>
```

Vi må ha en **kunde.ts** som tilsvarer kunde modellen på server:

```ts
export class kunde {
    id: number;
    fornavn: string;
    etternavn: string;
    adresse: string;
    postnummer: string;
    poststed: string;   
}
```

I **app.module.ts** må vi ha med FormsModule og HttpClientModule:

```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpClientModule } from '@angular/common/http';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent,
  ],
  imports: [
      BrowserModule.withServerTransition({ appId: 'ng-cli-universal' }),
      FormsModule,
      HttpClientModule,
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

I **app.component.ts** koder vi funksjonaliteten:

```ts
import { Component } from '@angular/core';
import { kunde } from './kunde';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html'
})
export class AppComponent {
    public alleKunder: Array<kunde>;
    public laster: string;

    constructor(private _http: HttpClient) { }

    hentAlleKunder() {
        this.laster = "vennligst vent";
        this._http.get<kunde[]>("api/Kunde/")
            .subscribe( data => {
                this.alleKunder = data;
                this.laster = "";
            },
            error => alert(error),
            () => console.log("ferdig get-/kunde")
        );
    }
}
```

Merk følgende:
- Vi må importere kunde klassen
- Vi trenger en HttpClient. Vi injiserer denne i konstruktøren (dependency injection)
- Vi må oppgi hva vi forventer å få som respons ved get kallet. Det gjør vi i hakeparanteser etter .get
- .subscribe utføres når api kallet er ferdig
- `api/Kunde` tilsvarer `api/[controller]` som vi spesifiserte i controlleren på server.