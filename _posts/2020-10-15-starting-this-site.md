---
title: How This Blog Was built
author: Galen Rice
date: 2020-10-15 12:00:00 -0400
categories: [Blogging, Meta]
tags: [meta, site building, github pages, jekyll, cloudflare, dns]
---

If you're wondering how I put this site together (or if you're me from the future who totally forgot about it), then you're in luck.

I'm not a fan of long intros, so let's just get started, shall we?

## Site stack

The stack is relatively simple: it's a static site generated using [Jekyll][jekyll_site] and served on [GitHub Pages][gh_pages_site] (you can see the repo [here][gh_this_site]). As such, I don't run my own middleware, database layers, or anything else, really. So, hosting is basically free, though posting updates can require some code changes that eat up my personal time.

The domain is managed on [Cloudflare][cloudflare_registrar_products_page] directly, which also provides full SSL/TLS encryption and my DNS records (we'll get into that, too).

Outside the Cloudflare setup and some local tooling I use in development and writing posts, everything you see here runs through GitHub, including the CI/CD workflows (which I did not author, though I may tweak them).

## Starting from a Jekyll theme

You can spend quite a while browsing through [themes for Jekyll][jekyll_docs_themes] trying to find the perfect one (I know I did). Ultimately, I settled on the [Chirpy theme][gh_chirpy_theme] you see here. For my taste, it struck a good middle ground between features, styling, and extensibility; without also steamrolling Jekyll's other config options. Thank you kindly for your hard work there, folks!

You can check the Chirpy repo for details on getting started, but for the basic steps I followed:

1. Fork the Chirpy theme repository, creating your own copy of the project on your GitHub account.
2. `git clone` your forked version on your local system.
3. Adjust the `_config.yml` settings to your liking. This takes some time, so patience is required.
4. Make some other initial tweaks, such as:
    - Generating favicons using [favicon-generator.org][favicon_generator] and a nice little picture of yours truly.
    - Changing the permalink style for posts to include `year/month/day`
    - Tweaks to `tools/run.sh` so that Docker builds can run properly (giving it write permissions that Windows tends not to).
    - Removing a GitHub Actions workflow dedicated to cleaning stale issues. That may work for the Chirpy team, but I have no need for it, personally.
5. Commit, push, and off it goes!

### Detaching the fork

You'll notice that my forked version of Chirpy doesn't link back to Chirpy as a fork. Reason being I don't intend to contribute back to Chirpy from this site's repo directly, so having any potential PRs trying to merge themselves to the upstream is a bit of a hassle.

The reasons may be personal, but the method is straightforward, actually:

1. Ensure all your files are up-to-date locally. You might also consider backing them up to a different location.
2. **Delete your repository** from GitHub (done through the repo settings, down in the Danger Zone at the bottom).
    - Oh don't worry, you'll get it back!
3. Create a new repository with the same name as the original, creating a blank repo that isn't a fork of anything.
4. Clone your new repository to your local machine.
5. Copy all files from the old local repo (minus the `.git/` directory) to the new repo.
6. Commit and push this full change.

It's a bit of a shame there isn't a more direct way to detach from an upstream repo in GitHub other than this "burn it down and start over" approach (outside of involving GitHub support), but there it is.

The other option, of course, would be for the upstream repository to be changed into a [template repository][gh_template_repo_blog] *before* we try to fork it, but that's not something most of us can control when we're the repo's consumers.

Anyway, moving on.

## Setting up GitHub Pages

This part's relatively straightforward. Most guides out there (including [GitHub's][gh_guide_github_pages]) will tell you to name your repo `username.github.io`, replacing `username` with your username. This *works*, but is not actually required.

In fact, the naming convention of `username.github.io` just tells GitHub to turn on GitHub Pages options automatically when the repo is first created.

Whether you name your repo `username.github.io` or `myawesomesite.example.com.bork`, the GH Pages settings can be found in the repository settings, by scrolling down until you see this section:

![My GitHub Pages Settings](/assets/img/my-gh-pages-settings.png)
_My GitHub Pages settings for this site, as of the time of this post. Don't worry, HTTPS is still enforced by Cloudflare!_

These settings can be enabled in any repo of your choosing, even an existing one. The only difference will be the URL that serves the pages by default:

- A repo named `username.github.io` will be served from `http://username.github.io`.
- Any other repo will be served from `http://username.github.io/reponame`, where `reponame` is... well, the name of the repo.

### What's that "Theme Chooser" setting?

*Pay no attention to the man behind the curtain*- kidding, kidding.

This ties into the earlier section about [choosing your starter theme](#starting-from-a-jekyll-theme), so let's back things up a bit.

You can use built-in themes for any Jekyll site (or even remote themes) just by adding a line or two in `_config.yml`. That way, the entire site gets built with that theme's settings, layouts, and other doodads, without you needing to own all that code. Technically, your whole site can be a single page with all the info you want to convey, just by using the default settings and a well-written `README` file.

The "Theme Chooser" option (shown above) gives you easy access to those built-in themes from the browser, updating your config to match your selection. You can change the chosen theme just by changing the config, either through the "Chooser" or changing `_config.yml`.

For this site, with all that I wanted to do besides a simple static page, I found it made more sense to bring the full theme into the repo. That's what the Chirpy folks recommended, anyway, but I also wanted the ability to alter the site internals without always referring back to a remote theme or trying to hammer in some hacky overrides.

If you just want a simple site with minimal coding, by all means, use the Theme Chooser or go with Jekyll's `minima` theme or some remote theme or whatever you like.

## Deployment steps

Is it too early to talk about deployment? Not at all! In fact, I find it best to get the full build chain up and running first, even if the finished product is just "Hello, World!" with a fancy font and dark theme (for those thinking Agile, this would be your minimum viable product).

When I started building the site, I figured I'd have GitHub Pages build off a single `main` branch and call it a day, with no need for a branching strategy unless I had big changes in mind. After all, they use Jekyll to build pages, and this is a Jekyll site, so... match made in heaven, right?

Well, the Chirpy theme in particular takes a different approach. You can check the [deployment workflow][site_deploy_workflow_file] for details, but essentially they've written their own test and build scripts, like their own category and tag collection logic, applying a "last modified" date to posts using the latest Git commit to touch the file, and other neat stuff.

Thanks to that tooling, the site's build chain looks like this:

1. I use `main` as the primary development branch. Pushes to this branch on GitHub trigger a CI workflow that builds and tests the site, looking for problems that might prevent a deployment later.
2. When I'm ready to deploy changes to the live site, I merge them into a `production` branch. Personal preference is to use local `git` commands to accomplish that:

   ```shell
   $ git checkout production  # checkout my local version of the prod branch
   $ git pull                 # getting latest from the remote version of this branch, just in case
   $ git pull origin main     # fast-forward merge the changes from main into production
   $ git push                 # push the production changes up to GitHub
   $ git checkout main        # get back into main to avoid problems later!
   ```

   The same can be done from GitHub using Pull Requests, merging `main` into `production`. That can slow things down, especially when you're just one person, but sometimes slower is better.

3. Pushes to the `production` branch trigger the deployment workflow, which again builds and tests the site, then runs a `deploy.sh` script within the GitHub Actions VM.
4. That script commits and pushes the site files into the `gh-pages` branch, setting a GitHub Actions bot account as the committer of the change.
5. The push to `gh-pages` triggers GitHub Pages to rebuild the live site.

From my viewpoint, all I need to do is stage changes to `main` while testing things out, then push those changes to `production` to trigger a deployment. Step away, get a coffee, come back 10 minutes later, *find a typo in the live post*, start over... You get the idea.

## Setting a custom domain

At this point, we've got a live site served from the easy-to-remember `http://username.github.io/reponame`. Now it's time to put a custom domain on it.

There's a lot of material in GitHub docs [about custom domains and GitHub Pages][gh_guide_custom_domains], so I'll spare some details. Specific to this site, it works like so:

- GitHub knows the custom domain for the repo by the contents of a [CNAME file][gh_this_site_cname_file] in the repo itself. The file is changed automatically when you change "Custom domain" in the repo settings (see figure [here](#setting-up-github-pages)).
- In Cloudflare (which I use as domain registrar), I have DNS records set to the following:

  ![My DNS Settings](/assets/img/my-gh-pages-dns.png)

  GitHub specifies (in the [Managing a custom domain guide][gh_guide_managing_custom_domain]) that configuring an apex domain (like `galenrice.com`, no subdomain) requires `A` records that point to these IP addresses:

  ```
  185.199.108.153
  185.199.109.153
  185.199.110.153
  185.199.111.153
  ```

  Further, handling the `www` subdomain is often a good idea, even if you don't want to use it. So a `CNAME` record in the DNS points back to `galenrice.com`, automatically redirecting traffic to `www` to the apex domain.

And... that's it, really. Tell GitHub the domain that traffic will be coming from and tell your DNS provider how to reach GitHub Pages, and you should be all set.

> Note in Jekyll there are settings in `_config.yml` that relate to the URL, which help it build links and RSS feed content and so on.

> The relevant settings for this site are:

> ```yaml
> # The full URL here
> url: "https://galenrice.com"
>
> # an empty string here
> # If the Jekyll site were served under, for instance, "galenrice.com/blog",
> # this setting would be "/blog", instead.
> baseurl: ""
> ```

## HTTPS enforcement

While GitHub Pages setting has an option to "Enforce HTTPS", it's not entirely relevant in this case so long as Cloudflare does the job for us.

> **Note**: I have a paid plan on Cloudflare to give me some extra options, so the following options may be limited if you have a free plan on your domain.

> And, of course, if you don't use Cloudflare at all, none of this is relevant. I'm just documenting my own process here, not trying to add an opinion on which tooling you *should* use.

In Cloudflare's SSL/TLS settings, I have:

- In **Overview**, SSL/TLS encryption mode is set to **Full**. "Full (strict)" may work, though I haven't tried it, and I don't find it necessary at the moment to try.
- In **Edge Certificates**, the "Always Use HTTPS" setting is turned on, which is basically essential. I also have "Opportunistic Encryption", "TLS 1.3", and "Automatic HTTPS Rewrites"; all of which are likely optional in most cases, but nice to have.

Additionally, I have a **Page Rule** applying "Always Use HTTPS" to `galenrice.com/*`, just to be certain it enforces HTTPS on the entire site.

When all's said and done, the site resolves automatically to `https://galenrice.com`, with that sweet lock icon showing a valid cert from Cloudflare.

## Summary

So, how'd we get here?

- Started from a Jekyll theme, forked it, tweaked it, and put it up on my GitHub as a new repo.
- Ran the full build chain using the `main -> production -> gh-pages` branch pipeline via GitHub Actions workflows.
- Activated GitHub Pages on the repo, targeting the `gh-pages` branch.
- Told the repo which custom domain it was going to receive traffic from.
- Adjusted DNS and SSL settings in Cloudflare to reach the site and enforce HTTPS.
- Wrote this whole post on the process for future reference and maybe to help others along the way.
- Had a meal and a coffee at some point in the past 12 hours (allegedly).
- Updated this post's post date and laughed at my optimism as to when I thought I would be done writing it when I started.
- Briefly considered the meaning of life and my place in the universe.
- Got a reminder of a work meeting starting very soon and frantically wrapped things up here so I could go prepare.

That's about it. Have fun out there!

**-G**

[cloudflare_registrar_products_page]: https://www.cloudflare.com/products/registrar/
[favicon_generator]: https://www.favicon-generator.org/
[gh_chirpy_theme]: https://github.com/cotes2020/jekyll-theme-chirpy/
[gh_guide_custom_domains]: https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/about-custom-domains-and-github-pages
[gh_guide_github_pages]: https://guides.github.com/features/pages/
[gh_guide_managing_custom_domain]: https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site
[gh_pages_site]: https://pages.github.com/
[gh_template_repo_blog]: https://github.blog/2019-06-06-generate-new-repositories-with-repository-templates/
[gh_this_site_cname_file]: https://github.com/GriceTurrble/galenrice.com/blob/main/CNAME
[gh_this_site]: https://github.com/griceturrble/galenrice.com
[jekyll_docs_themes]: https://jekyllrb.com/docs/themes/
[jekyll_site]: https://jekyllrb.com/
[site_deploy_sh_file]: https://github.com/GriceTurrble/galenrice.com/blob/main/tools/deploy.sh
[site_deploy_workflow_file]: https://github.com/GriceTurrble/galenrice.com/blob/main/.github/workflows/pages-deploy.yml
