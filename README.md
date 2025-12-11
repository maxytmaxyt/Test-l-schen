# 📌 Ticket-System Funktionsliste (Bestätigung für Entwicklerteam)

Bitte bestätigt, ob ALLE folgenden Features im finalen System enthalten sein sollen:

---

## 1. Ticket Panel
- [ ] Bot sendet Ticket-Panel automatisch beim Start  
- [ ] Panel wird aktualisiert, nicht dupliziert  
- [ ] Kategorien als Dropdown-Menü  
- [ ] Kategorien haben Emojis  
- [ ] Panel-Embed schön gestaltet  
- [ ] Panel-Daten werden gespeichert (Restart-sicher)

---

## 2. Ticket Erstellung
- [ ] Tickets werden in EINER festen Kategorie erstellt  
- [ ] Ticketname: `ticket-username`  
- [ ] User wird direkt gesperrt (kein Schreiben erlaubt)  
- [ ] Automatische Nachricht:  
      „Zurzeit ist kein Support verfügbar. Bitte warten…“  
- [ ] Ticket-Daten werden in JSON gespeichert  
- [ ] User kann erst schreiben, wenn Supporter übernimmt  

---

## 3. Claim / Unclaim System
- [ ] Button „Ticket übernehmen“  
- [ ] Nach Claim: User darf wieder schreiben  
- [ ] Nachricht: „Supporter X hat das Ticket übernommen“  
- [ ] Claim-Button verwandelt sich in „Unclaim“-Button  
- [ ] Unclaim sperrt User wieder  
- [ ] Claim/Unclaim Daten werden gespeichert (Restart-sicher)  
- [ ] Buttons funktionieren auch nach Bot-Neustart  

---

## 4. Ticket Weitergeben / Transfer
- [ ] Supporter können Tickets weitergeben  
- [ ] Slash-Command `/transfer @Supporter`  
- [ ] Nur Supporter dürfen den Command nutzen  
- [ ] Neuer Supporter erhält Schreibrechte  
- [ ] Übergabe wird im Ticket angezeigt  
- [ ] Ticket-Daten werden aktualisiert  

---

## 5. Owner Override
- [ ] Owner darf ALLES tun, ungeachtet von Rollen  
- [ ] Owner kann claimen, unclaimen, closen, Commands nutzen  
- [ ] Owner-Berechtigungen in config einstellbar  

---

## 6. Inaktivitätssystem
- [ ] Timer startet automatisch für jeden User  
- [ ] Wenn User zu lange nicht schreibt → Warn-Nachricht  
- [ ] Warn-Nachricht enthält Discord Relative-Time: `<t:xxx:R>`  
- [ ] Wenn User antwortet → Timer wird vollständig zurückgesetzt  
- [ ] Wenn User weiterhin inaktiv bleibt → Ticket wird automatisch geschlossen  

---

## 7. Ticket Closing
- [ ] Button „Ticket schließen“  
- [ ] Nur Supporter & Owner dürfen schließen  
- [ ] User kann nicht schließen  
- [ ] Closing-System restart-sicher  

---

## 8. Transcript-System
- [ ] Alle Nachrichten werden protokolliert  
- [ ] Beim Schließen wird Transcript generiert  
- [ ] Transcript wird in Transcript-Channel gesendet  
- [ ] User erhält Transcript zusätzlich per DM  
- [ ] Transcript wird in JSON gespeichert  

---

## 9. JSON-Datenbank
- [ ] Automatische Ordnerstruktur (`ticket_data/`)  
- [ ] Pro Ticket eine JSON  
- [ ] Sicheres Snowflake-Handling (IDs immer Strings)  
- [ ] Datenverlustfreie Speicherung  
- [ ] Restart-sicheres Laden aller Tickets  
- [ ] Nach Restart funktionieren alle Interaktionen weiter  

---

## 10. Startup-System
- [ ] Panel wird beim Start geladen / neu erstellt  
- [ ] Laufende Tickets werden geladen  
- [ ] Claim-Status wiederhergestellt  
- [ ] Alle Timer wieder aktiviert  
- [ ] Buttons wieder funktionsfähig  

---

Wenn alle Punkte stimmen → „Bestätigt“  
Dann wird die komplette finale Datei (alle Features in 1 Datei) erstellt.
