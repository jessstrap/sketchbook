teaching tool for sql. 

sqlite db. 
    holds current db state.
    for query execution 
query result display
    text mode
    grid mode
        headings specify type of column.
        headings optionally specify constraints. 
query entering window
    highlighting
    auto complete
bind var window 
    feeds into query auto complete
    reads from current query.
    includes dropdowns for type
query saving/loading/nameing
    includes current bind var window values
    ?reference saved query in other query?
history of all dml executed. 
    able to edit and get a new state. 
    app exports sqllite api that will add to history.
    also must record time of execution and non deterministic inputs.
ability to snapshot all the above and save/load snapshots to/from file.
    higher level sqlite db? does sqllite have a diff friendly, human readable, text export format
ability to diff snapshots
    save format should probably be text based. though could contain blob of current db state. 
    
should probably be written in rust and run on servotk. 
should be able to run in the browser and save to local filesystem or to a server
