# Email-Liste der Nutzer:innen eines Projektes

Die folgende Abfrage erstellt für ein Projekt die Liste aller Nutzer:innen mit Rolle und - gegebenenfalls - 
den zugeordneten Data Access Groups. Dazu einfach `PROJECT_ID` am Ende der Abfrage ersetzen.

```SQL

select rui.user_email, rui.username, ruro.role_name,
(select group_concat(group_name)  from
        redcap_data_access_groups dag, redcap_data_access_groups_users dagu where
        dag.project_id=ruri.project_id and dag.group_id=dagu.group_id and
		dagu.username=rui.username) as dags
from redcap_user_rights ruri
left join redcap_user_information rui on (ruri.username=rui.username)
left join redcap_user_roles ruro on (ruri.role_id=ruro.role_id)  
where rui.username is not null and
ruri.project_id=PROJECT_ID

```
