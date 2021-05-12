# galenrice.com

[![GitHub license](https://img.shields.io/github/license/griceturrble/galenrice.com.svg)](https://github.com/griceturrble/galenrice.com/blob/main/LICENSE)

*If it ain't broke, it's legacy.*

A development-focused blog by Galen Rice, software engineer in NJ, USA, who may or may not know what [he/him] is talking about.

Site is built with GitHub Pages using the [Chirpy][chirpy_theme_github] Jekyll theme (if you like it, go check it out and say thanks!).

## Development

To develop locally, I recommend using [Docker][docker_site] to easily generate an environment.

First, pull the Docker Jekyll image from Docker Hub:

```shell
$ docker pull jekyll/jekyll:latest
```

Then, you can run the project from the root directory of the project...

...using Bash:

```bash
$ docker run --rm -it --volume="$PWD:/srv/jekyll" -p 4000:4000 jekyll/jekyll bash tools/run.sh --docker
```

...using PowerShell (slightly different syntax for `pwd` needed):

```powershell
docker run --rm -it --volume="$(pwd):/srv/jekyll" -p 4000:4000 jekyll/jekyll bash tools/run.sh --docker
```

> WARNING: when developing on Windows, be sure your files have been checked out with LF line endings, or the bash scripts will fail outright. Set `git config --global core.autocrlf false` before cloning to prevent the files from getting CRLF line endings.

Builds take a while to complete, and auto-refresh may not work properly on a Windows machine, but it works.

### In CodeSpace

In a GitHub codespace, development is much simpler:

```bash
bundle install
bundle exec jekyll s
```

Now there is a running server, which you should find running at `http://127.0.0.1:4000`.

## Contributing

For suggestions regarding content on the site, feel free to comment using the site tools or contact me directly.

If you wish to contribute to the site layout and infrastructure, however, please check the [contributing guidelines][contributing_guidelines] here on GitHub.

## License

This work is published under [MIT](https://github.com/griceturrble/galenrice.com/blob/main/LICENSE) License.

[chirpy_theme_github]: https://github.com/cotes2020/jekyll-theme-chirpy
[contributing_guidelines]: .github/CONTRIBUTING.md
[docker_site]: https://www.docker.com/
