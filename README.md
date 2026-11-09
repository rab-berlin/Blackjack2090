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

## Zufall

RND liest am echten Gerät einen Zähler (+3 je Befehl, im Leerlauf etwa 205 Schritte/s). Zufällig ist also nur der Zeitpunkt der Tastendrücke.

Das Programm mischt deshalb jeden Zählerstand in einen Speicher (M7), der über alle Runden bleibt. Nach jeder Karte wartet es eine zufällige Zahl von Befehlen, so hängen die Karten ohne Tastendruck dazwischen nicht voneinander ab.

Im [Emulator2090](https://github.com/rab-berlin/Emulator2090) entspricht der RND-Modus `mic` dem Gerät.

## Einstellen

Die Pausenlänge (etwa 2,7 s) steht an Adresse EB: `11F` (MOVI 1,F). Mit 0 dauert sie 1,3 s, mit 2 etwa 4 s.
