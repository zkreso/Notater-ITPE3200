# Hallo verden i Angular

Først må man installere Angular på PC'en

```
npm install -g @angular/cli
```

Lag så nytt prosjekt i Visual Studio. Velg Angular som template (kan velge standalone i dette eksempelet). Det vil ta litt tid å kjøre første gangen fordi man må laste ned en del filer. Malen inneholder linker til diverse Angular ressurser, filer for testing og litt annet diverse. Hvis man velger ASP.NET project med Angular i stedet så vises et eksempelprosjekt. Den inneholder litt motstridende færre filer enn hvis man velger standalone.

Vi fjerner eksempelfilene for å strippe koden ned til et hallo verden prosjekt. Vi fjerner:
- WeatherForecast C# filene. (Controlleren og WeatherForecast selv)
- Alle komponentene som ligger under app (hver mappe er et komponent)
- I app.module.ts så fjerner vi imports av de komponentene vi slettet
- Vi fjerner også imports av alle moduler bortsett fra BrowserModule og NgModule
- Fjern også referanser til modulene og komponentene i @NgModule() funksjonen
- Vi erstatter innholdet i app.component.html med hallo verden tekst

Forklaring av app.component.ts:

```ts
import { Component } from '@angular/core'; // Importerer muligheten til å bruke komponenter

@Component({
  selector: 'app-root', // Definerer hvilken "tag" som skal velges i ../index.html
  templateUrl: './app.component.html' // Definerer hva innholdet i den valgte "tagen" skal erstattes med
})
export class AppComponent {
  title = 'app';
}
```