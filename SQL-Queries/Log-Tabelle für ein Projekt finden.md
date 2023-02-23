# Log-Tabelle für ein Projekt finden

## Funktion
Gibt den Namen der Log-Tabelle aus, in die die Log-Nachrichten für ein bestimmtes Projekt geschrieben werden.

## Abfrage
```SQL
SELECT `log_event_table`
  FROM `redcap_projects` 
  WHERE `project_id` = 1 -- Projekt-ID entsprechend anpassen!
```

## Erklärung

Bis REDCap 10 gab es nur eine Tabelle, in der sämtliche Log-Einträge geschrieben wurden (_redcap_log_event_). Weil diese Tabelle für große Instanzen zu Performanceproblmemen führte, wurden ab Version 11 weitere Log-Tabellen angelegt. Jedes neue Projekt wird bei der Erzeugung einer Log-Tabelle zugewiesen, was in der Tabelle _redcap_projects_ vermerkt wird, in der Spalte `log_event_table`. Die Abfrage gibt den Namen dieser Tabelle für die angegebene Projekt-ID aus.

[Zurück](../README.md)