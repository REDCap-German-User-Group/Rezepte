# Wiederfinden von verlorenen Daten

*Christof Meigen (meigen@medizin.uni-leipzig.de)*

## Einleitung

In der internen REDCap-Datentabelle werden Daten bewußt **nicht** gelöscht,
wenn sie durch Änderung der Metadaten nicht mehr sichtbar / abrufbar
sind. Was auf den ersten Blick wie eine unverzeihliche Verletzung der
Forderung nach referentieller Integrität aussieht (ich habe Daten zu
einer Variable, die es gar nicht (mehr) gibt), erweist sich in der
Praxis als nützlich, wenn man Daten wiederherstellen will.

Viele der fatalen Änderungen, die zu einem derartigen scheinbaren
Datenverlust führen sind im Produktivmodus gar nicht oder nur durch
Bestätigung eines Admins möglich, aber einerseits sind Admins auch nicht
unfehlbar, und andererseits ist (leider) nicht jedes Projekt, das
Produktivdaten sammelt, auch im Produktivmodus.

Die Abfragen dienen einerseits zum Auffinden scheinbar
verlorengegangener Daten, anderereits ist es sicher auch nicht
verkehrt, sie zu Qualitätssicherungszwecken auszuführen und eventuell
Rückmeldungen an Nutzer:innen zu geben oder - mit großer Sorgfalt -
nicht mehr benötigte Daten auch wirklich aus `redcap_data` zu löschen.

## Zu welchen nicht-existenten Variablen existieren Daten?

Die Abfrage liefert die Projekt-ID, den Projekt-Namen, den Feldnamen
und die Anzahl erfasster Daten.  Das sind die Fälle, in denen Felder
angelegt und ausgefüllt, aber später gelöscht wurden. Entsprechend sind
(ausser in Backups) die Metadaten (Feldkommentar, Optionen, Branching Logic)
nicht mehr verfügbar.

```SQL
with missings as
   (select distinct project_id, field_name from redcap_data except
   select project_id, field_name from redcap_metadata)
select missings.project_id, redcap_projects.project_name, missings.field_name, count(*) as n
from missings, redcap_data, redcap_projects
where missings.project_id=redcap_projects.project_id and
missings.project_id=redcap_data.project_id and missings.field_name=redcap_data.field_name
group by missings.project_id, missings.field_name
order by missings.project_id, n desc, missings.field_name;
```

## Zu welchen nicht-existenten Events gibt es Daten?

Nicht nur Variablen, auch events können gelöscht werden. In diesen
Fällen sind die kompletten Daten und (Feld-)Metadaten noch vorhanden,
die Daten werden im REDCap aber nicht angezeigt, wenn das Event nicht
(mehr) existiert.

Die Lösung zur Datenrettung ist hier übrigens ganz einfach: Ein neues
Event anlegen, und in `redcap_data` alle Einträge mit der alten
(gelöschten) `event_id` auf die neue `event_id` setzen.

```SQL
with missings as
   (select distinct project_id, event_id from redcap_data except
   select rea.project_id, rem.event_id from redcap_events_arms rea, redcap_events_metadata rem
          where rea.arm_id=rem.arm_id)
select missings.project_id, redcap_projects.project_name,
		  missings.event_id, count(*) as n from missings, redcap_data, redcap_projects
		  where missings.project_id = redcap_data.project_id and
		  missings.event_id=redcap_data.event_id and
		  missings.project_id=redcap_projects.project_id;

```

## Zu welchen Instrumenten existieren Daten in Events, denen sie nicht (mehr) zugeordnet sind?

Hier sind weder Daten noch Metadaten verloren, sondern das Instrument
ist einfach nicht mehr dem Event zugeordnet, zu dem die Daten erfaßt
wurden.

```SQL
with missings as 
(select distinct event_id, field_name from redcap_data except
  select redcap_events_forms.event_id, redcap_metadata.field_name
from redcap_metadata, redcap_events_forms where
redcap_metadata.form_name=redcap_events_forms.form_name)
select redcap_projects.project_name, redcap_events_metadata.descrip as event_name, redcap_events_metadata.event_id,
redcap_metadata.field_name, redcap_metadata.form_name from
missings, redcap_projects, redcap_events_metadata, redcap_events_arms, redcap_metadata
where
missings.event_id=redcap_events_metadata.event_id and
missings.field_name=redcap_metadata.field_name and
redcap_metadata.project_id = redcap_events_arms.project_id and
redcap_events_metadata.event_id=redcap_events_arms.arm_id and
redcap_events_arms.project_id=redcap_projects.project_id ;
```
