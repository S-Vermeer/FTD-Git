
```dataview
TABLE rel.target AS Target, rel.kind AS Kind
FROM ""
FLATTEN relations AS rel          
WHERE file.name = this.file.name
```


