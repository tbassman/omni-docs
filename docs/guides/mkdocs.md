# MkDocs

---

## Serving locally

```bash
mkdocs serve
```

Opens at [http://127.0.0.1:8000](http://127.0.0.1:8000) with live reload.

Changes to `docs/` are reflected automatically while the server is running. If edits stop appearing, the server has likely died — verify with:
```bash
curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:8000
```
A `200` means it's up; anything else means restart `mkdocs serve`.

**`OSError: [Errno 48] Address already in use`**
A previous server process is still holding port 8000. Kill it and restart:
```bash
kill -9 $(lsof -ti:8000)
mkdocs serve
```
`lsof -ti:8000` returns the PID of whatever is on that port; `-9` force-kills it.
