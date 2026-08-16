# Cyber Dashboard

## Writeups by Wabisabi

```dataview
TABLE platform, difficulty, os, status
FROM "01 - THM" OR "02 - HTB"
WHERE status = "in-progress"
SORT file.mtime DESC
```

## Writeups Done

```dataview
TABLE platform, difficulty, os
FROM "01 - THM" OR "02 - HTB"
WHERE status = "done" OR status = "pwned"
SORT file.mtime DESC
```

## Notes

```dataview
TABLE topic, file.mtime
FROM "03 - Notes"
SORT file.mtime DESC
LIMIT 15
```

## Tools

```dataview
TABLE topic, file.mtime
FROM "03 - Notes/Tools"
SORT file.name ASC
```

