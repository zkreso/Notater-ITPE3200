# Skjemaer (forms)

app.component.html:

```html
<div>
    <h2>Modellbasert Skjema </h2>
    <form [formGroup]="Skjema" (ngSubmit)="onSubmit()">
        <table>
            <tr>
                <td>
                    <label>Brukernavn:</label>
                </td>
                <td>
                    <input type="text" formControlName="brukernavn">
                </td>
                <td *ngIf="this.Skjema.controls.brukernavn.status=='INVALID'">
                    Obligatorisk
                </td>
            </tr>
            <tr>
                <td>
                    <label>Passord:</label>
                </td>
                <td>
                    <input type="password" formControlName="passord">
                </td>
                <td *ngIf="this.Skjema.controls.passord.status=='INVALID'">
                    Mellom 6 og 15 siffer
                </td>
            </tr>
            <tr>
                <td>
                    <button type="submit" [disabled]="!Skjema.valid">Register</button>
                </td>
                <td>
                </td>
            </tr>
        </table>
    </form>
</div>
```

- Med (ngSubmit) så bestemmer vi hva som skjer når vi trykker submit knappen. Vi kan kalle en funksjon med hvilket som helst navn, men (ngSubmit) forblir altså uendret.
- Med `[formGroup]="Skjema"` så sier vi at variabelnavnet er "Skjema" for denne form gruppen. (Husk at firkantparanteser er "property binding" mens vanlige paranteser er "event binding").
- Input boksene må giset formControlName for av vi skal kunne bruke disse ellers i koden.
- Her ser vi bruk av *ngIf for første gang. Sammenligne med *ngFor som vi har sett tidligere.

I app.component.ts finner vi funksjonaliteten:

```ts
import { Component } from "@angular/core";
import { FormGroup, FormControl, Validators, FormBuilder } from '@angular/forms';

@Component({
  selector: "app-root",
  templateUrl: "app.component.html"
})
export class AppComponent {

  Skjema: FormGroup;

  constructor(private fb: FormBuilder) {
    this.Skjema = fb.group({
      brukernavn: ["", Validators.required],
      passord: ["", Validators.pattern("[0-9]{6,15}")]
    });
  }

  onSubmit() {
    console.log("Modellbasert skjema submitted:");
    console.log(this.Skjema);
    console.log(this.Skjema.value.brukernavn);
    console.log(this.Skjema.touched);
  }
}
```

- Vi må importere FormGroup, FormControl, Validators og FormBuilder
- I konstruktøren injiserer vi en FormBuilder
- Formbuilder sin group() metode retunerer en FormGroup. Inne i group() kan vi oppgi navnene for variabler som skal med, og verdi.
    - Første variabel i verdi er "placeholder" tekst hvis vi ønsker det (her er det blankt)
    - Andre variabel i verdi er valideringsregelen. Vi ser at for brukernavn er eneste valideringsregel at det må være oppgitt noe som helst. For passord så er det regex.

I app.module.ts må ReactiveFormsModule importeres fra @angular/forms (og selvfølgelig legges til i imports)