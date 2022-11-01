# Databinding

Ta utgangspunkt i et "tomt hallo verden" prosjekt. Vi skal ha en inputboks og utskriften. Da må vi endre app.component.html til følgende:

```ts
<div>
    Navnet er {{navn}}<br />
    Skriv inn navnet her: <input type="text" [(ngModel)]="navn" />
</div>
```

Navnet på variabelen som skal skrives ut må være det samme som vi "binder" i input boksen.

Den litt spesielle konstruksjonen med både firkantparanteser og vanlige paranteser skyldes at det her er en "two-way data binding". Dette er unikt for Angular (i forhold til React eller Vue). Firkantparantesene brukes ellers for property binding og vanlige paranteser for event binding i Angular. Når vi kombinerer dem får vi det som heter "two-way data binding".

I tillegg må vi importere igjen FormsModule som vi fjernet sist gang fra app.module.ts. Den importeres fra `'@angular\forms`. Husk også å legge den til under imports: i @NgModule funksjonen.

For å gi variabelen navn en verdi ved oppstart kan vi bruke ngOnInit. Da trenger vi (iflg. videoen) å importere OnInit fra angular/core (men hos meg fungerte det uten. VS sier at OnInit ikke er brukt noe sted).

```ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html'
})
export class AppComponent {
    public navn: string; // definerer variabelen

    ngOnInit() { // ngOnInit brukes i stedet for konstruktør
        this.navn = "Ole Hansen";
    }
}
```