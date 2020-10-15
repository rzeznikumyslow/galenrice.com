# Docker tooling

## Build

In Bash:

```bash
docker run --rm -it --volume="$PWD:/srv/jekyll" jekyll/jekyll bash tools/build.sh --docker
```

In Powershell:

```powershell
docker run --rm -it --volume="$(pwd):/srv/jekyll" jekyll/jekyll bash tools/build.sh --docker
```

## Run

In Bash:

```powershell
docker run --rm -it --volume="$PWD:/srv/jekyll" -p 4000:4000 jekyll/jekyll bash tools/run.sh --docker
```

In Powershell:

```powershell
docker run --rm -it --volume="$(pwd):/srv/jekyll" -p 4000:4000 jekyll/jekyll bash tools/run.sh --docker
```
