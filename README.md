# This repository explains the notation, data, and ways to use WoC Analytics

# Maps/tables and their locations 

Data are available in two forms:

   - text file (gzipped and sorted by the first column) and located on da0:/data/basemaps/gz 
   - random-access (tch) databases located on on da0:/data/basemaps/


The notation of these databases consist of

   - The prefix describing the type of data,
   - Followed by completness (Full is complete while absence means only the updated data for that version)
   - The last letter denotes version (in alphabetical
    order). Currently the last version is N
	- Number  0-31 (or 0-127): the part of the database

For example, c2pFullN0.s for flat file and c2pFullN.0.tch for the
random access database

The notation in database type names is as follows:
```
a - author
c - commit (cc - child commit)
f - file
b - blob
p - project (P - after fork normaliation)
t - time 
m - module
```

# How to get the data out via a shell script
These are rando-access examples, where relatively few items from the entire set are queried. For large numbers of items a sweep may be much faster, see next section

1. Get a list of projects for an author
```
echo "Audris Mockus <audris@utk.edu>" | /da3_data/lookup/Prj2FileShow.perl /da0_data/basemaps/a2pFullN 1 32
```

2. Get a list of commits for an author
```
echo "Audris Mockus <audris@utk.edu>" | /da3_data/lookup/Prj2CmtShow.perl /da0_data/basemaps/a2cFullN 1 32
```

3. Get a list of commits for a blob
```
echo 05fe634ca4c8386349ac519f899145c75fff4169 | /da3_data/lookup/Cmt2BlobShow.perl /da0_data/basemaps/b2cFullM 1 32
```

4. Get a list of projects for a commit
```
echo e4af89166a17785c1d741b8b1d5775f3223f510f | /da3_data/lookup/Cmt2PrjShow.perl /data/basemaps/c2pFullN 1 32
```

6. Get list of commits for a file (a very large number of results)
```
echo main.c | /da3_data/lookup/Prj2CmtShow.perl /d0_data/basemaps/f2cFullM 1 32
```

7. Get list of child commits for a commit
```
echo af36164b147909db1c10e9fa036482dd3e885d86 | /da3_data/lookup/Cmt2BlobShow.perl /da0_data/basemaps/c2ccFullN 1 32
```

8. Get commit attributes (there are various options)
```
echo af36164b147909db1c10e9fa036482dd3e885d86 | /da3_data/lookup/showCmt.perl
```

9. Get blob content (only on da4)
```
echo 05fe634ca4c8386349ac519f899145c75fff4169 | /da3_data/lookup/showBlob.perl
```

10. Get content of a tree (only on da4)
```
echo f1b66dcca490b5c4455af319bc961a34f69c72c2 | perl ~audris/bin/showTree.perl
```

11. Various technical dependencies: e.g., for Go language in /da0_data/play/GothruMaps/m2nPNGo.s 
Details for Go, for example, are in c2bPtaPkgMGo.{0..31}.gz
see lookup/grepNew.pbs for exact details.

# How to get the data out via a shell script (sweep)

1. Get all commits modifying files with .c extension
```
for j in {0..31}; do zcat /da0_data/basemaps/gz/c2fFullN$j.s; done | grep '\.c$'
```
2. Get all authors containing string 'audris'
```
zcat /da0_data/basemaps/gz/asN.s | grep -i 'audris'
```

# Some data is extracted and stored in MongoDB to facilitate more accurate search
```
mongo --host=da1
use WoC
db.authors.find({ "email" : { $regex: /^andrey/i}} )
//or
db.authors.find({$text:{$search:"mockus"}})
```

# Creating services so that the maps can be accessed via socket
For example, we want to have author-to-project map on port 9999
```
./tcServer.perl /da0_data/basemaps/a2pFullN 32 9999 str2str
```

To query, we send authors to get the list of projects they committed
to:
```
echo 'Audris Mockus <audris@mockus.org>' | ./clientS2S.perl

E;5;Audris Mockus <audris@mockus.org>;AmreenS_Homework0;AmreenS_students;BIGJosher_Homework0;CaptainEmerson_chapters;CodyJae_students;Curtis017_students;Justa-ghost_students;MBenkhayal_students;...
```

If we want to have author-to-commit map on port 9998
```
./tcServer.perl /da0_data/basemaps/a2cFullN 32 9998 s2h
```

To query:
```
echo 'Audris Mockus <audris@mockus.org>' | ./clientS2H.perl
```


# Related repositories

[Project overview](https://bitbucket.org/swsc/overview)

[Python wrappers](https://github.com/ssc-oscar/oscar.py)

[Repository discovery](https://github.com/ssc-oscar/gather)

[Repository data extraction](https://github.com/ssc-oscar/libgit2)
