---
title: "{{ replace .Name "-" " " | title }}"
type: "issue"
date: {{ .Date }}
draft: true
---

Summary Here.

{{/* Get all `verse` type pages in exact last 7 days. */ -}}
{{/* Run `hugo new issue/{seq}.md` every week. Better with cronjob. */ -}}

{{ $sinceTimestamp := sub now.Unix 604800 -}}
{{ $versePages := where (where ( where .Site.Pages "Type" "verse" ) "Kind" "page") "Date.Unix" "gt" $sinceTimestamp -}}
{{ range $versePages.GroupByParam "subject" -}}
### {{ .Key }}

{{ range .Pages.ByDate -}}
* [{{ .Title }}]({{ .Permalink }})

{{ end -}}
{{ end -}}

