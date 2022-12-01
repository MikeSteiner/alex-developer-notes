https://stackoverflow.com/questions/64996452/how-to-list-all-pages-of-a-section-in-hugo

#Hugo  

```
{{ range (where .Site.Pages "Section" "programming") }}
   {{ range .Pages }}
      {{ .Title }}
   {{ end }}
{{ end }}
```

