* ### Compress folder
```bash
tar -zvfc archive_name.tar.gz directory_name
```
> * -z: Compress archive using gzip program
> * -v: Verbose i.e display progress while creating archive
> * -f: Archive file name
> * -c: Create archive

***

* ### Uncompress folder
```bash
tar –zvfx archive_name.tar.gz
```
> * -z: Compress archive using gzip program
> * -v: Verbose i.e display progress while creating archive
> * -f: Archive file name
> * -x: Extract to disk from the archive

* ### Timestamp convinient for file name format
```bash
date +%F_%T
```

* ### Set default rvm ruby version
```bash
rvm --default use 2.6.5
```

* ### Save file in remote server
```bash
cat dump.sql | curl -F 'f:1=<-' ix.io
```

* ### MacOS copy to clipboard from terminal
```bash
pbcopy < file.txt
```

* ### Fix ^M character when pressing enter
```bash
stty sane
```
