# Firecrawl Scraper – KI-gestützter Web-Scraping-Workspace

Ein Arbeitsbereich für **Claude Code**, in dem ein KI-Agent Daten aus dem Web holt: einzelne Seiten scrapen, strukturierte Daten extrahieren, Screenshots erstellen, ganze Websites crawlen und Seitenstrukturen kartieren.

Statt einzelne Scraper-Skripte von Hand zu schreiben, steuert der Agent den **[Firecrawl](https://firecrawl.dev) MCP-Server** an. Damit funktionieren auch JavaScript-lastige Seiten, paginierte Listen und Seiten mit Login oder Formularen.

## Was das Projekt zeigt

- **Agenten-Konfiguration statt Einzelskripte:** [`CLAUDE.md`](CLAUDE.md) legt fest, wie der Agent in diesem Projekt arbeitet: welches Tool er für Web-Aufgaben nutzt, welche nicht, und wo die Zugangsdaten liegen.
- **Werkzeugwahl nach Aufgabentyp:** [`FIRECRAWL_CHEATSHEET.md`](FIRECRAWL_CHEATSHEET.md) ordnet jedem Aufgabentyp das passende Firecrawl-Tool und die wichtigsten Parameter zu. Der Agent schlägt dort nach, statt zu raten.
- **Praxiswissen aus echten Einsätzen:** Das Cheat Sheet enthält Stolperfallen aus der Praxis, z. B. dass eine Startseite oft nur Teaser zeigt und die eigentliche Liste auf einer Unterseite liegt, oder wie man mit Bot-Schutz und Caching umgeht.
- **Sicherer Umgang mit Zugangsdaten:** Der API-Key liegt in einer lokalen `.env`, die nicht ins Repository gelangt. [`.env.example`](.env.example) zeigt, welche Variable gebraucht wird.

## Welche Aufgabe mit welchem Tool

| Aufgabe | Tool |
|---|---|
| Inhalt einer bekannten URL | `firecrawl_scrape` |
| Screenshot einer Seite | `firecrawl_scrape` mit `formats: ["screenshot"]` |
| Seite unbekannt, nur die Domain | `firecrawl_map`, danach `firecrawl_scrape` |
| Viele oder alle Seiten einer Website | `firecrawl_crawl` |
| Offene Websuche | `firecrawl_search` |
| Recherche über mehrere Quellen | `firecrawl_agent` |
| Klicks, Login, Formulare, Paginierung | `firecrawl_interact` |
| Lokale Dateien (PDF, DOCX, XLSX …) | `firecrawl_parse` |
| Seiten dauerhaft auf Änderungen überwachen | `firecrawl_monitor_*` |

Details und Parameter stehen im [Cheat Sheet](FIRECRAWL_CHEATSHEET.md).

## Einrichtung

1. [Claude Code](https://claude.com/claude-code) installieren.
2. Den Firecrawl MCP-Server verbinden und prüfen, ob er läuft:
   ```bash
   claude mcp list   # "firecrawl" sollte als Connected erscheinen
   ```
3. Für Skripte, die die Firecrawl-API direkt aufrufen: `.env.example` nach `.env` kopieren und den eigenen API-Key eintragen. Die MCP-Verbindung selbst meldet sich per OAuth an und braucht den Key nicht.
4. Claude Code in diesem Ordner starten und die Aufgabe beschreiben, z. B.:
   > „Hol mir alle offenen Stellen von omr.com als Tabelle mit Titel, Firma und Ort.“

## Tech-Stack

- **Claude Code** als KI-Agent
- **Firecrawl MCP** für Scraping, Crawling, Extraktion und Monitoring
- **Model Context Protocol (MCP)** zur Anbindung externer Tools an den Agenten
