# Cyber Dashboard

## Writeups by Wabisabi

```dataview
TABLE platform, difficulty, os, status
FROM "THM" OR "HTB"
WHERE status = "in-progress"
SORT file.mtime DESC
```

## Writeups Done

```dataview
TABLE platform, difficulty, os
FROM "THM" OR "HTB"
WHERE status = "done" OR status = "pwned"
SORT file.mtime DESC
```

## Notes

```dataview
TABLE topic, file.mtime
FROM "Notes"
SORT file.mtime DESC
LIMIT 15
```

## Tools

```dataview
TABLE topic, file.mtime
FROM "Notes/Tools"
SORT file.name ASC
```
