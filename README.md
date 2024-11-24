[English version](https://github-com.translate.goog/haykonus/JU-TE-6K-Video-HW-Patch?_x_tr_sl=de&_x_tr_tl=en&_x_tr_hl=de&_x_tr_pto=wapp)
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

Die Grafik zeigt die zeitlichen Verlauf der Signale der Host-CPU für zwei ausgewählte Situationen im Verhältnis zum Timing der Video-CPU. Die Signalbezeichnungen sind den [Plänen von Bert](https://github.com/boert/JU-TE-Computer/tree/main/Tiny_6k) entnommen.

![Video-Timing](/Bilder/Video-Timing.png)

## Schaltung
Die Schaltung erzeugt ein CPU_WAIT-Signal nach den o.g. Bedingungen und stoppt den Takt der Host-CPU für wenige Taktperioden, wenn ein Konflikt im Timing zwischen Host- und Video-CPU auftritt.   

![Schaltplan](/Bilder/Schaltplan.png)

## Leiterplatte

![Leiterplatte](/Bilder/Leiterplatte.png)

Andreas (Perser), aus dem [RT-Forum](https://www.robotrontechnik.de/html/forum/thwb/showtopic.php?threadid=22012), hat eine kleine Leiterplatte für diese Schaltung entwickelt. Er hat die Daten zur Verfügung gestellt und sie sind jetzt auch hier abgelegt.

> [!NOTE]
> Die Links unten anklicken und danach den Download-Button (Download raw file) im Github klicken, um die Datei zu laden.

Sprint Layout: [Tiny6kVideo-Entstoer1.lay6](/PCB/Tiny6kVideo-Entstoer1.lay6)

Gerber-Dateien: [Tiny6kVideo-Entstoer1.zip](/PCB/Tiny6kVideo-Entstoer1.zip)

## Messungen mit Logik-Analysator (24 MS/s)
Diese Messungen sind mit einem sehr preiswerten Logik-Analysator durchgeführt worden. Die max. Abtastrate beträgt 24 MS/s. Das bedeutet bei 8 MHz keine optimale Auflösung. Daher gibt es z.B. beim Systemtakt eine unsymmetrische Darstellung und manche Impulse haben eine leichte Ungenauigkeit. 

Aber die wesentlichen Abläufe sind sehr gut zu sehen und bestätigen die Annahmen. Damit wird auch deutlich, warum diese Lösung praktisch keine Verluste bei der Rechenzeit der Host-CPU verursacht und damit sehr effektiv ist. 

### 3 Video-Zeilen 
Das Bild zeigt den zeitlichen Verlauf der Signale für 3 Zeilen. Pro Zeile sind 40 Ladeimpulse für die Videoschieberegister (Y3,S1) zu sehen. In der zweiten und dritten Zeile gibt es jeweils 2 Zugriffe der Host-CPU auf den BWS (CPU_WAIT). Eine echte Taktunterbrechung gab es nur beim ersten Zugriff in Zeile 2. Es lief das Basic-Programm aus dem Demo-Video (s.o.).

![Video-Timing](/Bilder/LA_Timing_Zeile.png)

### Host-CPU-Takt verzögert

Marker 0: Extended Write Zyklus der Host-CPU (T1,T2,TX,T3)

Marker 4: Ladeimpuls der Videoschieberegister

Marker 3: Umschalten der BWS-Adressen auf Host-CPU

![Video-Timing](/Bilder/LA_Timing_verzögert.png)

### Host-CPU-Takt synchron

Marker 1: Extended Write Zyklus der Host-CPU (T1,T2,TX,T3)

Marker 2: Ladeimpuls der Videoschieberegister

Marker 5: Umschalten der BWS-Adressen auf Host-CPU

![Video-Timing](/Bilder/LA_Timing_synchron.png)

## Messungen mit Digital Oszilloskop (1 GS/s)

gelb: CPU_WAIT

blau: XTAL_CPU

### Host-CPU-Takt verzögert (Bsp. 1)

![Video-Timing](/Bilder/Timing-verzögert1.png)

### Host-CPU-Takt verzögert (Bsp. 2)

![Video-Timing](/Bilder/Timing-verzögert2.png)

### Host-CPU-Takt synchron

![Video-Timing](/Bilder/Timing-synchron.png)

## Quellen

[^1]: [The ZX Spectrum ULA: How to design a microcomputer](http://www.zxdesign.info/book/)



Dieses Projekt nutzt Infos aus folgenden Quellen:

http://www.zxdesign.info/book/

https://github.com/boert/JU-TE-Computer/tree/main/Tiny_6k

https://www.zilog.com/docs/um0016.pdf
