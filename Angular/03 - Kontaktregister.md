# Kontaktregister (på klient)

Eksempel med kontaktregister på klient (ingen kommunikasjon med server).

Her må vi ha med FormsModule i app.module.ts, siden vi bruker forms.

Vi definerer en klasse for kontakter, kontakt.ts:

```ts
export class kontakt {
    utData: string;

    constructor(navn: string, telefon: string) {
        this.utData = `${navn}  ${telefon}`;
    }
}
```

app.component.html blir slik:

```ts
<div>
    <h1>Kontaktliste</h1>
    <table class="table">
        <thead>
            <tr>
                <td>Navn og telefonnummer</td>
            </tr>
        </thead>
        <tbody class="col-md-4">
            <tr *ngFor="let kontakt of kontakter" style="margin-bottom: 10px;">
                <td>{{kontakt.utData}}</td>
                <td><button class="btn btn-danger btn-xs" (click)="slettKontakt(kontakt)">Slett kontakt</button></td>
            </tr>
        </tbody>
    </table>

    <div class="col-md-4">
        <input [(ngModel)]="navn" placeholder="navn" />
        <input [(ngModel)]="telefon" placeholder="telefon" />
        <button class="btn btn-primary btn-xs" (click)="leggTilKontakt()">Legg til kontakt</button>
    </div>
</div>
```

app.component.ts inneholder variablene og funksjonene som det refereres til i html filen. Den importerer kontakt.ts

```ts
import { Component, OnInit } from '@angular/core';
import { kontakt } from './kontakt';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html'
})
export class AppComponent {
    navn: string;
    telefon: string;

    kontakter: Array<kontakt> = [];

    leggTilKontakt(): void {
        const enKontakt = new kontakt(this.navn, this.telefon);
        this.kontakter.push(enKontakt);
        this.navn = "";
        this.telefon = "";
    }

    slettKontakt(enKontakt: kontakt): void {
        const indeks = this.kontakter.indexOf(enKontakt);
        this.kontakter.splice(indeks, 1);
    }
}
```