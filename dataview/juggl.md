```juggl
workspace: testgraph
nodes:   
  - from: ""    
    where: type               # selects notes that have a `type` key    
    label: name    
    id: file.path    
    export:      
      - type                  # make `type` visible to the filter bar    
    style:      
      shape: ellipse      
      color: "#4A90E2" 
edges:   
  - from: ""    
    where: relations    
    flatten: relations    
    source: file.path    
    target: target    
    label: kind                # use the `kind` property as the edge label  
    style:      
      color: "#888888"      
      width: 2 
```

