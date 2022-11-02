# Modal

Modal er et annet navn for "meldingsboks". For å kunne ta i bruk dette trenger vi ng-bootstrap som er Angular sin versjon av bootstrap. Det kan installeres via terminal og npm ved å bla til ClientApp mappen i løsningen og kjøre følgende kommando:

```
npm install --save @ng-bootstrap/ng-bootstrap@5
```

Her er det versjon 5 som er valgt - det er mulig det finnes nyere versjoner. Sjekk nettsiden til ng bootstrap for å se hva som er nyeste versjon og hvilken versjon av Angular som støtter hva. Hvis man får noe lignende det under som melding etter å ha kjørt kommandoen så er installasjonen sannsynligvis vellykket:

```
+ @ng-bootstrap/ng-bootstrap@5.3.1
added 1 package from 1 contributor
```

ng-bootstrap må importeres i app.module.ts

```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { NgbModule } from '@ng-bootstrap/ng-bootstrap';

import { AppComponent } from './app.component';
import { Modal } from './modal';

@NgModule({
  declarations: [
    AppComponent,
    Modal
  ],
  imports: [
    BrowserModule,
    FormsModule,
    NgbModule
  ],
  providers: [],
  bootstrap: [AppComponent],
  entryComponents: [Modal] // merk!  
})
export class AppModule { }
```

- Merk at man under declarations må ta med Modal
- entryComponents er det andre som er nytt her i forhold til det vi har sett tidligere

(Jeg er ganske sikker på at FormsModule ikke brukes i dette eksempelet)

Funksjonaliteten finner vi i app.component.ts:

```ts
import { Component} from '@angular/core';
import { NgbModal} from '@ng-bootstrap/ng-bootstrap';
import { Modal } from './modal';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html'
})
export class AppComponent {
  
  constructor(private modalService: NgbModal) { }

  visModal() {
    const modalRef = this.modalService.open(Modal, {
      backdrop: 'static',
      // betyr at man ikke kan klikke vekk modalen ved å trykke andre steder

      keyboard: false
      // betyr at man ikke kan klikke vekk modalen med ESC
    });

    modalRef.componentInstance.navn = "Per Hansen";

    modalRef.result.then(retur => {
      console.log('Lukket med:'+ retur);
    });
  }
}
```

Det injiseres en NgbModal i konstruktøren.

app.component.html har bare en knapp som kaller visModal()

Selve modal.ts ser slik ut. Den gjør ikke annet enn å injisere en NgbActiveModal, som tas som parameter i en NgbModal sin open metode.

```ts
import { Component} from '@angular/core';
import { NgbActiveModal } from '@ng-bootstrap/ng-bootstrap';

@Component({
  templateUrl: 'modal.html'
})
export class Modal {
  constructor(public modal: NgbActiveModal) { }
}
```

Og html-en til modal.ts, modal.html, ser slik ut:

```html
<div class="modal-content">
    <div class="modal-header">
        <h4 class="modal-title" id="modal-basic-title">Ønsker du å slette</h4>
    </div>
    <div class="modal-body">
        {{navn}} ?
    </div>
    <div class="modal-footer">
        <!-- den første knappen vil bli "default" dvs. kjøres når "enter" trykkes -->
        <button type="button" class="btn btn-default" (click)="modal.close('Lukk klikk')">Lukk</button>
        <button type="button" class="btn btn-danger" (click)="modal.close('Slett klikk')">Slett</button>
    </div>
</div>
```