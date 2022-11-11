# Routing

Routing er måten å navigere mellom komponenter i Angular. På den måten får man single page application.

Vi lager en app-routing.module.ts i app mappen:

```ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { LagreComponent } from './lagre/lagre.component';
import { ListeComponent } from './liste/liste.component';

const appRoots: Routes = [
  { path: 'liste', component: ListeComponent },
  { path:'lagre', component: LagreComponent },
  { path:'', redirectTo:'/liste', pathMatch:'full' }
]

@NgModule({
  imports: [
    RouterModule.forRoot(appRoots)
  ],
  exports: [
    RouterModule
  ]
})
export class AppRoutingModule { }
```

- Vi må importere RouterModule og Routes modulene
- Vi må importere de komponentene det skal være mulig å navigere til.
- Vi definerer en konstant appRoots av typen Routes (som vi importerte). Routes er et array som inneholder path objekter. Typiske path objekter har en variabel path, type string, og en variabel component type komponent. Vi ser også at det er mulig å ha andre path objekter, f.eks. et objekt som redirecter til en annen path.
- I @NgModule funksjonen så importerer vi en RouterModule.forRoots() som tar det Routes arrayet vi laget.

I selve komponentene trenger vi ikke legge til noe spesielt for at dette skal fungere. Tvert imot så kan vi kommentere ut selector da dette ikke vil ha noen mening når vi bruker routing. Vi må passe på å ta me en exports slik at vi kan importere disse i app-routing.module.ts. Komponentene vil typisk se slik ut (dersom de ikke har noen annen funksjonalitet):

```ts
import { Component } from '@angular/core';

@Component({
  //selector: 'app-root', -denne gjør ikke noe da det er routing som gjelder
  templateUrl: './lagre.component.html'
})
export class LagreComponent {
  
}
```

Vi lager komponentene i egne mapper under /app.

I app.module.ts må vi importere det nye vi har laget. Det er altså de komponentene det skal kunne navigeres til, i tillegg til den routing modulen vi har laget. Vi har også laget en komponent for navbar som vi må importere. Bortsett fra det skal det ikke gjøres noen endringer her.

```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppRoutingModule } from './app-routing.module';

import { AppComponent } from './app.component';
import { LagreComponent } from './lagre/lagre.component';
import { ListeComponent } from './liste/liste.component';
import { NavMenuComponent } from './nav-meny/nav-meny.component';

@NgModule({
  declarations: [
    AppComponent,
    LagreComponent,
    ListeComponent,
    NavMenuComponent
  ],
  imports: [
    BrowserModule.withServerTransition({ appId: 'ng-cli-universal' }),
    AppRoutingModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

html filen til app komponenten blir litt annerledes. Vi må ha med en tag for navbaren og vi må ha med en tag der vi ønsker at det som vi router til skal skrives ut:

```html
<app-nav-meny></app-nav-meny>
<div class="container">
    <router-outlet></router-outlet>
</div>
``` 

Selve nav menyen skal skrives ut til app-nav-meny tag'en, så vi må skrive det i nav-meny.component.ts:

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-nav-meny',
  templateUrl: './nav-meny.component.html'
})
export class NavMenuComponent {
  isExpanded = false;

  collapse() {
    this.isExpanded = false;
  }

  toggle() {
    this.isExpanded = !this.isExpanded;
  }
}
```

Collapse og toggle har å gjøre med at det er en hamburgermeny. Men det sentrale her er hvordan man legger til routing links i navbaren. Det gjøres ved bruk av `routerLink`:

```html
<header>
    <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
        <div class="container">
            <a class="navbar-brand" routerLink="/">Routing</a>
            <button class="navbar-toggler"
                    type="button"
                    data-toggle="collapse"
                    data-target=".navbar-collapse"
                    aria-label="Toggle navigation"
                    [attr.aria-expanded]="isExpanded"
                    (click)="toggle()">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="navbar-collapse collapse d-sm-inline-flex flex-sm-row-reverse"
                 [ngClass]="{ show: isExpanded }">
                <ul class="navbar-nav flex-grow">
                    <li class="nav-item">
                        <a class="nav-link text-dark" routerLink="/">Home</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link text-dark" routerLink="/liste">Liste</a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link text-dark" routerLink="/lagre">Lagre</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
</header>
```

