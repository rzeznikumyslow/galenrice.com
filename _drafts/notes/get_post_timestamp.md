# Get post timestamp

Grab the post timestamp using the following:

In bash:

```bash
date +"%Y-%m-%d %H:%M %z"
```

In Powershell:

```powershell
Get-Date -UFormat '%Y-%m-%d %R %Z00'
```

Both will output in the format of, for example, `2020-10-14 20:30 -0400` (Oct 14, 2020, 8:30 pm US/Eastern with correct offset for DST). This is the proper format for a post date.

Obviously, adjust the post date as needed. No need to be precise to the minute here, depending on needs.
