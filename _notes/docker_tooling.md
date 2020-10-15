# Running with Docker

In Bash:

```powershell
docker run --rm -it --volume="$PWD:/srv/jekyll" -p 4000:4000 jekyll/jekyll bash tools/run.sh --docker
```

In Powershell:

```powershell
docker run --rm -it --volume="$(pwd):/srv/jekyll" -p 4000:4000 jekyll/jekyll bash tools/run.sh --docker
```

**Do not build first**.

Running alone should work fine.
