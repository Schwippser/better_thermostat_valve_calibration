# Thermostat Calibration Watchdog


<a href="https://www.buymeacoffee.com/tomreim0s" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me a Coffee" style="height: 60px !important;width: 217px !important;" ></a>


[🇬🇧 English](README.md) | 🇩🇪 Deutsch

Ein Home-Assistant-Blueprint, der automatisch eine **Ventil-Neukalibrierung** von Heizkörperthermostaten startet, wenn diese ihre Kalibrierung verloren haben.

[![Öffne deine Home-Assistant-Instanz und zeige den Blueprint-Import-Dialog.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Schwippser/better_thermostat_valve_calibration/blob/main/thermostat_calibration_watchdog.yaml)

> **Hinweis:** Dieser Blueprint funktioniert nur mit Thermostaten, die eine Ventilkalibrierung unterstützen und bei denen sich diese aus Home Assistant über einen Button starten lässt.

## Wozu dieser Blueprint?

Manche Thermostate, zum Beispiel der **Sonoff TRVZB**, können ihre Ventilkalibrierung verlieren, oft wegen der Heizungsanlage selbst. Das Ventil meldet dann eine Position von 0 %, ist tatsächlich aber offen, und der Raum heizt über die Zieltemperatur hinaus. Die gemeldete Ventilposition ist deshalb **nicht brauchbar**, um das Problem zu erkennen.

Dieser Blueprint erkennt stattdessen das Symptom: Der Raum liegt lange deutlich über der Solltemperatur und kühlt nicht ab. In diesem Fall drückt er den Kalibrierungs-Button der Thermostate dieses Raums.

## So funktioniert es

1. **Auslöser:** Die Isttemperatur liegt **ununterbrochen** mindestens *X* K über der Solltemperatur (Standard 2,0 K) und das für eine bestimmte Zeit (Standard 1 Stunde). Die Climate-Entity darf nicht `off` sein.
2. **Trendprüfung:** Die Automation merkt sich die Isttemperatur, wartet (Standard 30 Minuten) und prüft erneut.
3. **Entscheidung:** Die Kalibrierung startet nur, wenn der Raum **immer noch** mindestens *X* K zu warm ist **und** die Temperatur nicht um mehr als die Toleranz gefallen ist (Standard 0,2 K). Das filtert das normale, langsame Abkühlen nach dem Absenken der Solltemperatur heraus.
4. **Aktion:** Eine optionale Push-Nachricht wird gesendet und alle gewählten Kalibrierungs-Buttons werden gedrückt.

Nach einer Kalibrierung löst die Automation erst wieder aus, wenn die Differenz mindestens einmal unter die Schwelle gefallen ist und sie danach erneut überschreitet. Das verhindert endlose Kalibrierungsschleifen.

## Voraussetzungen

- Home Assistant **2024.10.0** oder neuer
- Eine `climate`-Entity pro Raum mit den Attributen `current_temperature` und `temperature`
- Ein oder mehrere `button`-Entities, die die Ventilkalibrierung der Thermostate starten (**das Thermostat muss eine Kalibrierung unterstützen** und dafür eine `button`-Entität bereitstellen, sonst ist der Blueprint nicht nutzbar)
- Optional: die Home-Assistant-Companion-App für Push-Nachrichten

Entwickelt und getestet mit **Sonoff-TRVZB**-Thermostaten (Zigbee2MQTT) in Kombination mit **Better Thermostat**. Better Thermostat kann mehrere Thermostate in einer Climate-Entity zusammenfassen, deshalb akzeptiert der Blueprint mehrere Kalibrierungs-Buttons. Andere Kombinationen sollten funktionieren, wenn die Voraussetzungen erfüllt sind, wurden vom Autor aber nicht getestet.

## Installation

**Import-Button:** Klicke oben auf das Badge.

**Manuell:**
1. Kopiere `blueprints/automation/thermostat_calibration_watchdog.yaml` nach `/config/blueprints/automation/<dein_ordner>/` in deinem Home Assistant.
2. Wird der Blueprint nicht in der Liste angezeigt, lade die Automationen neu (Entwicklerwerkzeuge → YAML → *Automationen neu laden*) oder starte Home Assistant neu.

## Verwendung

Lege **pro Raum eine Automation** an: Einstellungen → Automationen & Szenen → Automation erstellen → Blueprint verwenden → *Thermostat Calibration Watchdog*.

### Einstellungen

| Eingabe | Beschreibung | Standard |
|---|---|---|
| **Climate entity (room)** | Climate-Entity mit Ist- und Solltemperatur (z. B. die Better-Thermostat-Entität des Raums). | – |
| **Calibration button(s)** | Ein oder mehrere Buttons, die die Ventilkalibrierung starten. Alle werden gedrückt. | – |
| **Temperature difference** | Mindestdifferenz *Ist − Soll* in Kelvin. | 2,0 K |
| **Minimum duration above threshold** | So lange muss die Differenz ununterbrochen bestehen. | 1 h |
| **Trend check delay** | Wartezeit bis zur zweiten Prüfung. | 30 min |
| **Trend tolerance** | Erlaubter Temperaturabfall während der Wartezeit, der noch als „nicht sinkend" gilt. | 0,2 K |
| **Mobile app device** | Gerät der Companion-App für die Push-Nachricht. Leer = keine Benachrichtigung. | leer |
| **Notification title / message** | Texte der Push-Nachricht. Die Nachricht darf Templates enthalten, die Variable `climate_entity` steht zur Verfügung. | englischer Text |

Die Bezeichnungen in der Oberfläche sind englisch, damit der Blueprint international nutzbar ist.

### Beispiel (Raum mit zwei Thermostaten)

- Climate entity: `climate.heizung_badezimmer`
- Calibration buttons: `button.thermostat_bad_1_valve_travel_calibration`, `button.thermostat_bad_2_valve_travel_calibration`

### Deutscher Benachrichtigungstext

Standardmäßig ist die Nachricht englisch. Für Deutsch trägst du bei *Notification message* zum Beispiel ein:

```
Neukalibrierung gestartet für {{ state_attr(climate_entity, 'friendly_name') }}. Ist {{ state_attr(climate_entity, 'current_temperature') }} °C, Soll {{ state_attr(climate_entity, 'temperature') }} °C.
```

## Schwelle wählen

| Variante | Differenz | Dauer | Einsatz |
|---|---|---|---|
| Empfindlich | 1,5 K | 45 min | Schnelle Reaktion, mehr Fehlalarme möglich |
| **Standard** | **2,0 K** | **60 min** | Guter Kompromiss |
| Vorsichtig | 3,0 K | 90 min | Räume mit viel Sonne oder anderen Wärmequellen |

Ein normales Überschwingen eines Thermostats liegt bei etwa 0,5 bis 1 K. Direkte Sonne, ein Kamin oder ein Backofen können einen Raum lange über dem Sollwert halten und eine unnötige Kalibrierung auslösen. Das ist harmlos, lässt sich aber mit einer höheren Schwelle verringern.

## Hinweise

- Die Automation läuft im Modus `single` mit stillem `max_exceeded`: Wartet ein Durchlauf auf die Trendprüfung, werden weitere Auslöser desselben Raums ignoriert.
- Die Kalibrierung selbst führen die Thermostate aus, sie kann einige Minuten dauern, in denen sich die Ventile bewegen.
- Bleibt der Raum nach einer Kalibrierung weiterhin zu warm, liegt die Ursache vermutlich nicht in der Ventilkalibrierung.

## Haftungsausschluss

Nutzung auf eigene Gefahr. Der Autor übernimmt keine Verantwortung für Schäden oder Fehlfunktionen an Heizungsanlagen oder Geräten.

## Lizenz

Dieses Projekt steht unter der [MIT-Lizenz](LICENSE).
