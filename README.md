# Blackjack2090

Blackjack für den Busch Microtronic 2090: ohne Zusatzhardware, nur mit der 6-stelligen Anzeige, der Hex-Tastatur und den 4 LEDs an den Ausgängen.

- 254 Programmschritte (00-FD), 2 frei
- Datei: [blackjack.mic](blackjack.mic) (Version 17)

## Start

Nur nach dem Einschalten oder nach RESET starten, dann `HALT` `NEXT` `00` `RUN`. Das Startkonto beträgt 20 Chips.

## Regeln

- Unendliches Deck. Die Karten sind 1 (Ass, zählt 1 oder 11), 2-9, A (Zehn), b (Bube), C (Dame) und d (König).
- Einsatz immer 2 Chips.
- **Blackjack** zahlt 3:2.
- **Versicherung** kostet 1 Chip und zahlt 2:1.
- **Double** auf zwei beliebige Karten.
- **Split** bei gleichen Karten oder zwei Zehnerwerten, höchstens 3 Hände. Split-Asse bekommen nur eine Karte.
- **Dealer:** Er steht auf 17, auch auf weich 17. Wie am Tisch sieht man nur seine offene Karte, bis alle Spielerhände fertig sind; dann zieht er sichtbar Karte für Karte.

## Bedienung

| Taste | Wirkung |
|---|---|
| 0 | Rest (stehen bleiben) |
| 1 | Karte |
| 2 | Double |
| 3 | Split |
| 4 | Versicherung |
| E | am Kontostand: Spiel beenden (Cash in) |

Die LEDs zeigen binär die höchste Taste, die gerade erlaubt ist.

## Anzeige

| Bild | Bedeutung |
|---|---|
| `KKK` | Kontostand vor jeder Runde (dezimal); weiter mit einer beliebigen Taste |
| `E D K K S S` | Spielbild: E = du bist dran, D = offene Dealer-Karte, K K = deine ersten zwei Karten, S S = Summe; ab der 3. Karte `E D K 0 S S` mit nur der neuen Karte |
| `F 0 K 0 S S` | Dealer zieht (die verdeckte Karte erscheint an der Stelle der 0) |
| `D D S S 0 G` | Ergebnis je Hand: Dealer-Summe, deine Summe (2b = Blackjack), Gewinn bzw. Verlust in Chips; LED 1 = Gewinn, alle LEDs = Verlust |

Mit Ass wird die niedrige Summe angezeigt, außer die weiche Summe ist 19, 20 oder 21.

## Technik: die Anzeige ist das Registerfeld

Das Programm hat keine Anzeigeroutine. Der Befehl `DISP 6,0` zeigt die sechs Register R5 R4 R3 R2 R1 R0 als sechs Ziffern, und `DISP 6,C` zeigt R1 R0 RF RE RD RC. Die Register sind deshalb so verteilt, dass das gewünschte Bild einfach im Registerfeld steht:

| Register | Bedeutung |
|---|---|
| R0 / R1 | angezeigte Summe (Einer / Zehner) |
| R2 / R3 | die letzten zwei Karten, die neuere in R3 (im Display links) |
| R4 | offene Dealer-Karte |
| R5 | E = du bist dran, F = Dealer |
| R6 | verdeckte Dealer-Karte |
| R7 | Zehner der harten Summe |
| R8 | Ass-Merker (1 = Ass in der Hand, 2 = weich, aber hart angezeigt) |
| R9 / RA | Splits dieser Runde / wartende Hände |
| RC | höchste erlaubte Taste, sonst Hilfsregister |
| RD RE RF | Kontostand (hexadezimal, in der Bank), beim Würfeln der Zählerstand |

Ein Ergebnisbild wie `D D S S 0 G` entsteht, indem das Programm vor dem `DISP` nur die Register mit den richtigen Werten füllt.

## Zufall

RND liest am echten Gerät einen Zähler (+3 je Befehl, im Leerlauf etwa 205 Schritte/s). Zufällig ist also nur der Zeitpunkt der Tastendrücke.

Das Programm mischt deshalb jeden Zählerstand in einen Speicher (M7), der über alle Runden bleibt. Nach jeder Karte wartet es eine zufällige Zahl von Befehlen, so hängen die Karten ohne Tastendruck dazwischen nicht voneinander ab.

Im [Emulator2090](https://github.com/rab-berlin/Emulator2090) entspricht der RND-Modus `mic` dem Gerät.

## Einstellen

Die Pausenlänge (etwa 2,7 s) steht an Adresse EB: `11F` (MOVI 1,F). Mit 0 dauert sie 1,3 s, mit 2 etwa 4 s.

## Wie es gemacht ist

Der 2090 hat 16 Register mit 4 Bit, keine Zeiger und nur ein Rücksprungregister. Blackjack passt trotzdem in 254 Schritte:

### Dezimal addieren durch Subtrahieren

Die Summe der Hand liegt dezimal in Zehner (R7) und Einer (R0). Eine Karte mit dem Wert w wird nicht addiert, sondern R0 − (10 − w) gerechnet: Gibt es keinen Borger (Carry 0), ist der Einer gleich Einer + w − 10, und der Zehner steigt um 1. Gibt es einen Borger, ergänzt `ADDI A` die 10 wieder, und der Zehner bleibt. So entsteht die Dezimalsumme ohne Umrechnung und ohne `DZHX`. Die Karten Zehn, Bube, Dame und König zählen alle 10: Sie haben den Wert 0 in 10 − w.

### Karten aus einem Zählerstand

`RND` liest am echten Gerät einen Zähler, der in jedem Befehl um 3 weiterläuft. Zufällig ist also nur der Zeitpunkt, an dem das Programm ihn liest. Das Programm macht daraus in drei Schritten gute Karten:

1. **Sammeln:** Jeder Zählerstand (RD, RE, RF) wird auf einen Speicher X in M7 addiert, der über alle Runden bleibt.
2. **Warten:** Nach jeder Ziehung läuft eine Schleife RE + 1 Durchläufe, damit der Zähler bis zur nächsten Karte zufällig weit gelaufen ist.
3. **Verwerfen:** Ein Wert über 12 wird neu gezogen (`CMPI C,D`, `BRC`), der Rest plus 1 ergibt die Karten 1-13. So sind alle 13 Karten gleich wahrscheinlich.

Zwischen der Taste am Kontostand und dem Geben steht kein Befehl, der den Zähler setzt. `HXDZ` und `DZHX` benutzen den Zähler als Arbeitsregister, und `DZHX` würde ihn auf 999 setzen. Das Programm rechnet deshalb nur einmal mit `HXDZ` für die Anzeige um und sichert das Konto vorher hexadezimal mit `MAS` in der Bank. Ein `DZHX` ist nicht nötig, der Zähler läuft nach dem Tastendruck frei weiter, und der Tastendruck zählt als Zufall.

### Eine Taste, ein Vergleich

Die LEDs zeigen binär die höchste erlaubte Taste (3 mit Split, 2 mit Double, 1 sonst). Dieselbe Zahl in RC ist auch die Prüfung: Nach `KIN` heißt `CMP C,D` mit Carry „Taste größer als erlaubt“, und die Taste wird ignoriert. Ein einziger Wert erledigt LED-Anzeige und Tastenfreigabe. Nach einer gezogenen Karte fällt RC auf 1, denn Double und Split sind danach nicht mehr möglich.

### Hände im Ring statt im Index

Bis zu 3 Hände (2 Splits) liegen in M0-M5, je 2 Register: Zehner mit Double-Merker und Einer. Ohne indirekte Adressierung gibt es keinen Zugriff „Hand n“. Stattdessen schiebt `ROT` den ganzen Speicher um eine Hand weiter: Die älteste Hand kommt oben heraus in RF RE, die gerade fertige wandert unten hinein. Die Abrechnung ruft `ROT` auf, bis die älteste Hand herauskommt, und füllt dabei unten mit 0 nach, die Ablage leert sich also beim Lesen. Ein leerer Platz fällt an der 0 im oberen Register auf und wird übersprungen.

### Das Ergebnis aus drei Werten

Für jede Hand bildet das Programm R7 = F / 0 / 1 für Verlust / unentschieden / Gewinn. R7 startet bei −1 (F), und zwei hintereinanderliegende `ADDI 1,7` zählen hoch: Wer in `WIN` einspringt, läuft durch beide und kommt bei 1 an. Wer in `PUSH` einspringt, nur durch eine und kommt bei 0 an. Ein Verlust springt über beide. Daraus ergibt sich alles andere:

- **Die LEDs:** `DOT 7` schaltet R7 direkt auf die Ausgänge: F = alle LEDs (Verlust), 0 = keine, 1 = LED 1 (Gewinn).
- **Die Auszahlung:** 2 · (R7 + 1) Chips kommen aufs Konto zurück (0, 2 oder 4), bei Double doppelt so viel.

### Ein Konto in drei Hex-Ziffern, in der Bank

Der Kontostand liegt in RD-RF hexadezimal (bis 999) und wird mit `EXRM` in M8-MF geparkt, solange die Spielregister gebraucht werden. `BET` zieht 2 Chips ab: `SUB`, `SUBC`, `SUBC` über die drei Ziffern. Ein Carry am Ende heißt „zu wenig“, und `PAY` bucht zurück. Dasselbe Unterprogramm zahlt die Gewinne aus (`ADD`, `ADC`, `ADC`).

### Die Pause mit Zählern

Die Pause zwischen den Bildern ist eine Zählschleife über `DOT E`: Das Register E zählt auf den LEDs von F bis 0 herunter, zweimal. Der Spieler sieht damit, wie lange das Bild noch steht. Die Länge steht im Startwert von F (siehe „Einstellen“).

### Nach dem Einschalten ist RF = F

Nach Einschalten oder RESET steht RF = F, alle anderen Register 0. Der Kontostand beginnt deshalb mit `MOVI 0,F` an Adresse 00: 014h = 20 Chips, nicht F14h.
