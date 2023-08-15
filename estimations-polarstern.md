PowerMate (Polarstern)-Test | Offene ToDo's

Dieses Dokument beschreibt noch offene Punkte im PowerMate Projekt die *zwingend* angegangen werden müssen um einen erfolgreichen Test im Rahmen des Angebots von Polarsterns zu gewährleisten.

Rahmenbedingungen:
- [20,100] Installationen eines PowerMate Devices

# PowerMate Devices
Für den Test und dessen Erfolges wegen, sollte ein einziger Typ von PowerMate Device verwendet werden.

Für diesen Unterpunkt stellen sich folgende ToDo's:
- Welches Device soll im Test verwendet werden? ***1h***
- Update/Ausroll Prozess für Devices um diese *over-the-net* updaten zu können ***25h***
  - Hier ist noch erst keine Lösung in Diskussion, je nach Device kann das auch anders aussehen. Phone/Raspberry hätten Internetverbindung, ein ESP nicht.
- *Fool-Proof* Anleitung für den Installationsprozess ***5h***

# Datenverarbeitung

Abhängig davon, auf welches PowerMate Device man sich einigt, gibts es bezüglich dessen Datenverarbeitung der Mangetsensordaten andere ToDo's:

- ESP:
  - Kann keine Rotationskonvertierung, diese müsste noch implementiert werden ***20h***
- Raspberry:
  - Klären, welchen Einfluss Ausrichtung des Devices hat? ***5h***
- PaD:
  - Rotationskonvertierung nicht stabil ***10h***

# Authentifizierung

Um für den Test mehrere PowerMate Devices zu verwenden, ist es ratsam, Authentifizierung für User zu gewährleisten. Ohne hat der Test keine gute Aussagekraft wie gut sich mehrere Devices zugleich im PowerMate Projekt befinden können.

Während dem Projekt unter dem Semester wurde hieran schon gearbeitet, jedoch konnte kein fertiges System implementiert werden. Der Plan und eine Vision stünde bereits, müsste jedoch final implementiert werden.

Konkret sind folgende ToDo's offen:
- Device <-> User Assoziation ***25h***
  - Fakebar durch die App?
  - (Redis) Datenbank, Auth-Lambda und Policies
- Die PowerMate App muss auf die Device <-> User Assoziation abgestimmt werden ***25h***
