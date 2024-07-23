+++
title = 'Blog Setup'
date = 2024-07-22T15:10:28-07:00
draft = false
+++

I've used [hugo](https://gohugo.io) to generate this blog. It wasn't my first choice mostly because I knew I wanted to use Github Pages and they default to Jeckyll.

Here is how I created it.

# tl;dr
1. [Install Hugo:](#install-hugo) `snap install hugo`
2. [Create a Hugo site:](#create-new-site) 
    1. Go to the parent directory: `cd ~/workspace`
    2. Create the site: `hugo new site <repo-name> --format yaml`
    3. Switch directories: `cd <repo-name>`
    4. Update url to localhost: `sed -ie "s*baseURL: https://example.org/*baseURL: 'http://localhost:1313/'*" hugo.yaml`
    4. Set up config directory: `mkdir -p config/_default && mv hugo.yaml config/_default`
4. [Initialize a new module:](#module-init) `hugo mod init <unique identifier>`
5. [Theme](#theme)
    1. Configure the theme: 
```bash
cat << EOF >> config/_default/hugo.yaml
theme: 
  - github.com/xianmin/hugo-theme-jane

module:
  imports:
    - path: "github.com/xianmin/hugo-theme-jane"
      disable: false

EOF
```
    2. Install the theme: `hugo mod get`
6. [Vendor dependencies:](#vendor-modules) `hugo mod vendor`
7. Add to GitHub pages [or follow the appropriate instructions here](https://gohugo.io/categories/hosting-and-deployment/)
    1. Initialize the git repo in the hugo site you created: `git init`
    2. Go to your GitHub account and create a new repo `<your-account-name>.github.io` for GitHub Pages by clicking the plus (+)
    3. Follow the instructions to add this new location to the repo you've been working in. Something like `git remote add origin https://github.com/<your-account-name>/<your-account-name>.github.io.git`
    3. Go to `https://github.com/<your-account-name>/<your-account-name>.github.io/settings/pages`
    4. Change the `Source` dropdown to `GitHub Actions`
    5. [Follow steps 5-6 to set up GitHub Actions.](https://gohugo.io/hosting-and-deployment/hosting-on-github/) This will build and deploy your site to GitHub pages every time you push a change.
    6. Within your repo, follow the instructions GitHub gave you to assign the new repo as your


# Install Hugo {#install-hugo}
I'm using Ubuntu, so I was able to `snap install hugo`. There are a lot of different ways available to install both Go and Hugo, this was a single straightforward step. I may create an image to use in the future, but this is fine for now.

# Create a new site {#create-new-site }
I keep all my projects in `~/workspace`, so I started there.

```bash
cd ~/workspace
```

The simplest setup command to use is `hugo new site <site-name>`. This will create a new hugo site in the current directory with a single toml configuration file. I prefer to use a config directory and yaml instead of toml, so I did a little more work than this.

Change the `BLOG_URL` variable to what is suggested if you're using github pages. 

Other options:
- [GitHub Pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
- [GitLab Pages](https://gohugo.io/hosting-and-deployment/hosting-on-gitlab/)
- [Some other hosting](https://gohugo.io/categories/hosting-and-deployment/)

```bash
BLOG_URL="<my-github-account-name>.github.io"
hugo new site "${BLOG_URL}" --configDir config --format yaml
cd "${BLOG_URL}"
mkdir -p config/_default
mv hugo.yaml config/_default
```

Open the file `config/_default/hugo.yaml` with your preferred editor. Change the `baseURL` value from `https://example.org/` to `http://localhost:1313/` - *including* that last slash.

## Set the site up as a module {#module-init}
[Docs](https://gohugo.io/hugo-modules/use-modules/#initialize-a-new-module)

Rather than fiddling with nested git repos or submodules, I used Go's module functionality. Hugo provides this functionality.

The mod init command was a little confusing for me. The idea is to create a unique Go module name. The suggested module name `example.com/my-blog` is meant to be your actual site url as a _namespace_ with a subfolder as a _unique identifier_. For example, I used `tanglisha.gihub.io/blog` in my mod init. The subfolder can be whatever you want. If you aren't using Go aside from this, you are unlikely to run into a name conflict. The value you use will go into your [top level `go.mod` file.](https://github.com/tanglisha/tanglisha.github.io/blob/main/go.mod). 

I suggest _not_ leaving this as `example.com/my-blog`, if you create a new Hugo site later and do the same thing you could get a module name conflict in Go that may be confusing.

```bash
hugo mod init "${BLOG_URL}/blog"
```

# Theme {#theme}
[I looked through the themes listed on the Hugo website to find a simple one.](https://themes.gohugo.io/) I ended up choosing [Jane](https://themes.gohugo.io/themes/hugo-theme-jane/).

Because we're using Go modules, the theme can be added in the config file at `config/_default/hugo.yaml`. `module` and `theme` are top level keys, don't indent them.

[Docs](https://gohugo.io/hugo-modules/use-modules/#use-a-module-for-a-theme)
```yaml
theme:
  - github.com/xianmin/hugo-theme-jane

module:
  imports:
    - path: "github.com/xianmin/hugo-theme-jane"
```

Now we can download the theme back on the command line.

[Docs](https://gohugo.io/hugo-modules/use-modules/#update-modules)

```bash
hugo mod get
```

# Vendor modules {#vendor-modules}
[Docs](https://gohugo.io/hugo-modules/use-modules/#vendor-your-modules)

I highly recommend vendoring your modules. This will download your dependencies into a `_vendor` directory, which avoids issues later on.

- You won't need internet access in the future to build your site
- If a dependency disappears for any reason, you can still build your site. [A disappearing dependency is what caused the left-pad incident in 2016](https://en.wikipedia.org/wiki/Npm_left-pad_incident)
- Completely sidesteps a [dependency confusion attack](https://owasp.org/www-project-top-10-ci-cd-security-risks/CICD-SEC-03-Dependency-Chain-Abuse)

Be sure to include the `_vendor` directory in your repo.

```bash
hugo mod vendor
```

# Add to GitHub Pages
[Docs](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

Go to your GitHub account and create a new repo `<your-account-name>.github.io` for GitHub Pages by clicking the plus (+). It must be named using this pattern to work.

Look at the menu along the top of the screen. Click `Settings`. On the new page, click `Pages` in the menu along the left of the screen.

Change the `Source` dropdown to `GitHub Actions`.

[Follow steps 5-6 to set up GitHub Actions.](https://gohugo.io/hosting-and-deployment/hosting-on-github/) This will build and deploy your site to GitHub pages every time you push a change. I'm not pasting the yaml here because

1. I didn't write it / it's not mine
2. If there are changes, that page will be kept up to date

Go back to the main page of your repo. You can do this by clicking on the repo name at the top of the screen.

This page will give you instructions on adding this as a remote. It'll look something like `git remote add origin https://github.com/<your-account-name>/<your-account-name>.github.io.git`. Copy it from the website, then go back to he terminal/console you were working in before. Make sure you're still located in the hugo site you've created.

```bash
git init
```
Now paste the command you copied from Github.

Add the new directories and files to git.

```bash
git add _vendor .github archetypes assets config content datta i18n layouts public resources static themes .hugo_build.lock go.mod go.sum`
git commit -m 'set up website'
```

Before this will work in GitHub Pages, we need to add a new configuration file specific to production, where it will be published.

Make sure the baseURL used in this snipped begins with `https://` and ends with a slash.

```bash
mkdir -p config/production
echo "baseURL: \"${BLOG_URL}/\"" > config/production/hugo.yaml
git add config/production/hugo.yaml
git push
```

Give GitHub a few minutes to build your site, then visit your blog url. You should see a site that's styled, but doesn't yet have any content.