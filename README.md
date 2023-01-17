# Example Github Workflows #

This repo has example Github workflows that you can use to get your Canopy project deploying to a free hosting service, or building electron apps.

## Provider requirements ##

The main difficulty in hosting a Canopy website on a static assets server is that Canopy paths use forward slashes that do not correspond to the directory structure on disk. For example, a Canopy URL might be `www.example.com/United_States/New_York/Manhattan`, which if served from a static assets server would cause a request for an `index.html` file in a `United_States/New_York/Manhattan` directory. However, all of these paths are merely cosmetic, and we always want the client to request the `index.html` at the site root, and by default this request will cause a 404.

So, Canopy provides several options for allowing these cosmetic paths without causing the browser to actually look for nested directories on the server that don't exist.

* Option 1: Use an application server. The Canopy CLI comes with an Express server that can be run using the `canopy serve` command. This server takes a request like `www.example.com/United_States/New_York/Manhattan` and ignores the request path, and just serves the same `index.html` file every time. The downside of this approach is that it can be harder to find free or low-cost application hosting providers than it is to find static asset hosting providers.

* Option 2: Use hash URLs. The `canopy build` command comes with the `--hash-urls` option which causes paths to be renders with a fragment ID, like `www.example.com/#/United_States/New_York/Manhattan`. This has the benefit of working with static asset servers, but the downside of adding a visible character to the URL. This is the only option of these four that is compatible with Github Pages.

* Option 3: Use symlinked directories. This option is a bit complicated and may be deprecated in future, but can be used for the time-being by running the `canopy build --symlinks` command. This option causes the creation of directories in the static assets root directory for every topic in the project, which will each contain symlinks to every other such directory. That means that if the user requests `www.example.com/United_States/New_York/Manhattan`, this will look for a `United_States` directory in the root directory, which it will find, then it will look for a `New_York` directory in the `United_States` directory, which it will find a symlink for, and then it will look in _that_ directory for a directory called `Manhattan`, which it will also find a symlink for. Every folder also includes the same `index.html` file. In this manner, we can create an infinite set of nested folders such that no Canopy path requests a non-existant directory, and the user always receives the same `index.html` asset. The downside of this approach is a slightly longer build process, and that not many static assets hosting providers will resolve symlinks in URL structure, which is necessary for this option to work.

* Option 4: Use a hosting provider that allows customized static assets hosting. Whereas option #1 causes us to ignore request paths at the server level, it is possible with some hosting services to ignore request paths at the provider level, creating a static assets server that only ever offers one asset regardless of the path requested. This approach allows us to have clean URLs (with no `#` character) while still using free static asset hosting. For this reason, the workflows in this repo follow this approach, and show the user how to configure Github to work with the Netlify hosting provider. By using a `netlify.toml` file, you can indicate to Netlify that you want all requests to be redirected to the root directory, causing the root `index.html` file to always be served regardless of the request path.

## Online Edit ##

This repo offers two Github workflows for building your Canopy project. The difference is the creation of an "online edit" branch. In the simpler set-up which can be found in the `plain` directory above, the Canopy user edits the project locally using the command line, commits their changes, and pushes them to the main branch on Github, causing a project build and a commit to the `build` branch, which Netlify should be watching for continuous deployment.

The second permutation, in the `online-edit` folder above, offers a solution for groups trying to collaboratively edit a Canopy project with team members who may not be familiar with the command line or git. The `online edit` workflow creates a branch called `online-edit` that has a file called `canopy_bulk_file`, which contains the contents of the project in an easy-to-edit format. When a user edits `canopy_bulk_file` using the online Github interface and commits their change to the `online-edit` branch, this triggers a parse of `canopy_bulk_file`, and a commit to the `main` branch, and then a corresponding commit to the `build` branch, triggering a deploy. The benefit of this feature is that non-technical team members can make live edits to a deployed project just by editing a natural-language text file. If the edit causes build errors, the new version will not be deployed, and the existing version will remain live.

## Setting up Netlify ##

These are instructions for how to set up automatic deployment of your Canopy project on Netlify.

1. Initialize a project locally using the `canopy init` command in a new empty directory.

2. Put the contents of one of the above workflow directories into a `.github/workflows` directory. You will need to edit these files to include the correct name, email, and github repo, replacing the all-caps placeholder strings in these workflow files.

3. Initialize a git repository in your project directory, make an initial commit, and push that commit to a Github repo.

4. Log in to Netlify using your Github credentials.

5. Choose the "Add new site" option under the "Sites" tab.

6. Pick your user or organization, then pick your repo. For the "Branch to deploy" option, select the `build` branch.

7. Now your Netlify website should deploy on any change to the `build` branch, which should update on any change to the topics files on the `main` branch.

## Electron ##

Use `electron/electron.yml` to build Electron apps from your project. You must have a 1024x1024 `png` icon file in your `project/assets` directory called `electron-icon.png`.

Warning: If you use online assets like images, these will not work in an offline electron app. All assets should be served locally by using a `assets` directory in the project directory, and then referencing assets with the `/_assets/` prefix from your `expl` file markdown.

Happy editing!
