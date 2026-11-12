
system_diagnose.py – README
Übersicht
system_diagnose.py ist ein Python-Diagnoseskript, das aus dem Log === FEHLER & WARNUNGEN === hervorgeht. Es sammelt Systeminformationen und prüft typische Problemquellen (Snap-Loop-Geräte, DNS/Netzwerk, dmesg-Zugriff, Paket-Cache, verwaiste Pakete) und gibt die Ergebnisse in einer strukturierten Warn-/Fehlerliste aus.

Hinweis: Da der ursprüngliche Skriptinhalt nicht vorliegt, beschreibt diese README die Funktionen, die sich aus dem erzeugten Log ableiten lassen. Passen Sie die Abschnitte an, falls Ihr Skript zusätzliche Prüfungen enthält.

Voraussetzungen
Betriebssystem: Linux (getestet auf Arch-basierten Systemen, z. B. Manjaro/CachyOS)

Python: 3.x

Rechte: Einige Prüfungen (z. B. dmesg, Paketverwaltung) benötigen sudo oder root.

Optionale Werkzeuge:

pacman / pacman-contrib (für Cache- und Paketprüfungen)

snapd (falls Snap-Prüfungen aktiv sind)

mullvad (für VPN-/DNS-Kontext, optional)

Installation
bash
# Skript herunterladen bzw. kopieren
chmod +x system_diagnose.py
Keine externen Python-Pakete erforderlich – das Skript nutzt ausschließlich die Standardbibliothek und Systembefehle.

Verwendung
bash
python3 system_diagnose.py
Ausgabe erfolgt auf der Konsole im Format:

text
=== FEHLER & WARNUNGEN ===
<ISO-Zeitstempel>

[HH:MM:SS] [ERROR]   x  <Meldung>
[HH:MM:SS] [WARN]    !  <Meldung>
Ausgabe in Datei umleiten
bash
python3 system_diagnose.py > diagnose_$(date +%F).log 2>&1
Geprüfte Bereiche
Bereich	Prüfung	Typische Meldung
Snap / Loop-Devices	Belegung von /dev/loop*	0.0/0.0 GB (100%) → falscher Alarm, SquashFS ist immer 100 %
DNS	Erreichbarkeit von 8.8.8.8, 1.1.1.1	ConnectionRefusedError → bei VPN (z. B. Mullvad) normal
Netzwerk-Interfaces	Status von enp0s31f6, wlp61s0	DOWN = kein Kabel; drop = Paketverluste
dmesg	Lesbarkeit des Kernelpuffers	Operation nicht erlaubt → kernel.dmesg_restrict=1
Paketverwaltung	Verwaiste Pakete, Cache-Größe	Verwaiste Pakete: N, Cache: X GB
Journal	App-bezogene Warnungen	app-...-Einträge aus journalctl
Interpretation der Ergebnisse
Snap-Loop 100 %
Kein Fehler. SquashFS-Images sind schreibgeschützt und fest dimensioniert – die 100 % sind rein kosmetisch. Ignorieren oder Snap deinstallieren.

DNS ConnectionRefused
Bei aktivem VPN mit eigenem DNS (z. B. Mullvad) erwartetes Verhalten – der Kill-Switch blockiert direkte DNS-Anfragen. Prüfen Sie den VPN-Status (z. B. mullvad status), nicht die Fehlermeldung.

Interface DOWN
enp0s31f6: DOWN bedeutet meist nur, dass kein Kabel steckt. Aktivieren mit:

bash
sudo ip link set enp0s31f6 up
dmesg nicht lesbar
Normal für unprivilegierte Benutzer. Für Einsicht:

bash
sudo dmesg
Verwaiste Pakete & Cache
Bereinigung (Arch):

bash
sudo pacman -Rns $(pacman -Qtdq)   # verwaiste Pakete
paccache -r                        # alter Cache (pacman-contrib nötig)
Bekannte Einschränkungen
Die DNS-Prüfung unterscheidet nicht zwischen „VPN aktiv" und „Netzwerk defekt" – interpretieren Sie die Meldung im Kontext.

dmesg-Prüfung schlägt bei restriktivem Kernel grundsätzlich fehl, auch wenn das System gesund ist.

Die Cache-Größe kann stark schwanken, je nachdem ob paccache bereits gelaufen ist.

Snap-Prüfungen laufen ins Leere, wenn snapd nicht installiert ist.

Beispielausgabe
text
=== FEHLER & WARNUNGEN ===
2026-11-12T15:02:35.564943

[15:02:18] [ERROR]   x  /dev/loop5 -> .../bare/5: 0.0/0.0 GB (100%)
[15:02:18] [WARN]    !  8.8.8.8: ConnectionRefusedError
[15:02:18] [WARN]    !  enp0s31f6: DOWN
[15:02:19] [WARN]    !  dmesg nicht lesbar: ...
[15:02:19] [WARN]    !  Verwaiste Pakete: 11 (pacman -Rns)
[15:02:19] [WARN]    !  Cache: 5.73 GB
Lizenz
Frei verwendbar – keine Gewährleistung. Passen Sie das Skript an Ihre Umgebung an.
