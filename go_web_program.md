> in go, you simply make ever file in the same directory of main package,and they'll be included
> 
> template file
    {{define "layout"}}
    {{template "navebar" .}}
    {{end}}

