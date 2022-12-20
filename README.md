# r4tio.dev
A personal encyclopedia to servers, security and privacy.
---

[![Netlify Status](https://api.netlify.com/api/v1/badges/d4ff8d90-226f-4fb4-968a-34f415eb4cb3/deploy-status)](https://app.netlify.com/sites/r4tio/deploys)

## Local Development

1. [Install Hugo](https://gohugo.io/getting-started/installing/)
1. Clone this repository: `git clone --recurse-submodules https://github.com/s0up4200/r4tio.dev`
1. Run `hugo serve` to start the local development server at (by default) `http://localhost:1313`
   - Alternatively run `hugo` to simply build the site into the `/public` directory

- Run `git submodule update --merge` to update the [WonderMod theme](https://github.com/Wonderfall/hugo-WonderMod) to the version specified in this repo
  - Run `git submodule update --remote --merge` to update to the upstream master branch of WonderMod
- Run `./external-blogs.sh` to pull the latest versions of the articles from [wonderfall.dev](https://wonderfall.dev) and place them in `/content`
