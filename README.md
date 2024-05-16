# JU-TE-6K-Video-HW-Patch
Die hier vorgestellte Schaltung unterdrückt die Störungen im Bildschirm, die durch konkurrierende Zugriffe der Host- und Video-CPU auf den Bildwiederholspeicher entstehen. Die Video-CPU erhält nun Vorrang vor der Host-CPU. Dazu wird das Verfahren "CPU-Takt-Verzögerung" aus dem ZX-Spectrum verwendet, welches im [ULA-Chip](http://www.zxdesign.info/book/) implementiert wurde.

Die Schreib- und Lesezyklen der Host-CPU werden durch Verlängerung des T1-State der Host-CPU mit dem Lesezyklus der Video-CPU synchronisiert. Die Host-CPU erhält erst nach dem Laden der Video-Schieberegister Zugriff auf den Bildwiederholspeicher.

## Demo-Video
[Vergleich: OHNE und MIT Störunterdrückung](https://nextcloud-ext.peppermint.de/s/J6zgPi3iB5reWo2)


## Video-Timing

### Analyse
Die Ursache für die Störungen ist der asynchrone Zugriff der Host-CPU auf den Bildwiederholspeicher. Die Aktivierung des Signals /DS der Host-CPU bewirkt das Umschalten der Adressen des Bildwiederholspeichers von der Video-CPU auf die Host-CPU. 

Wenn diese Umschaltung vor dem Laden der Video-Schieberegister (DL299) innerhalb eines 8-Pixel-Zyklus geschieht, werden bei entsprechender Konstellation Daten vom Daten-Bus der Host-CPU in die Schieberegister geladen. Das ist als Störung sichtbar.

### Lösung
Die Host-CPU darf nicht asynchron zur Videosignal-Erzeugung durch die Video-CPU auf den Bildwiederholspeicher zugreifen. Dazu muss das Timing beider Prozesse synchronisiert werden. 

- Erkennen von Zugriffen der Host-CPU auf den Adressbereich 4000H-5FFFH (Bildwiederholspeicher)
- Erzeugen eines CPU_WAIT-Signals (START_CPU_WAIT und STOP_CPU_WAIT)
- Anhalten des Taktes der Host-CPU während CPU_WAIT aktiv ist
- Während der H- und V-Austastlücke keine Taktunterbrechung zulassen

Die folgende Grafik zeigt die zeitlichen Verlauf der Signale der Host-CPU für zwei ausgewählte Situationen im Verhältnis zum Timing der Video-CPU. Die Signalbezeichnungen sind den [Plänen von Bert](https://github.com/boert/JU-TE-Computer/tree/main/Tiny_6k) entnommen.

![Video-Timing](/Bilder/Video-Timing.png)

## Schaltung
Die folgende Schaltung erzeugt ein CPU_WAIT-Signal nach den o.g. Bedingungen und stoppt den Takt der Host-CPU für wenige Taktperioden, wenn ein Konflikt im Timing zwischen Host- und Video-CPU auftritt.   

![Schaltplan](/Bilder/Schaltplan.png)

## Messungen
gelb: CPU_WAIT

blau: XTAL_CPU

<br>

Host-CPU-Takt verzögert (Bsp. 1):

![Video-Timing](/Bilder/Timing-verzögert1-500.png)

<br>

Host-CPU-Takt verzögert (Bsp. 2):

![Video-Timing](/Bilder/Timing-verzögert2-500.png)

<br>

Host-CPU-Takt synchron:

![Video-Timing](/Bilder/Timing-synchron-500.png)


## Quellen

[^1]: [The ZX Spectrum ULA: How to design a microcomputer](http://www.zxdesign.info/book/)



Dieses Projekt nutzt Infos aus folgenden Quellen:

http://www.zxdesign.info/book/

https://github.com/boert/JU-TE-Computer/tree/main/Tiny_6k

https://www.zilog.com/docs/um0016.pdf
