# Testing changes for GitHub Pages hosted project documentation

The first option makes it easy to link to rendered changes for code review but
is slower for rapid changes or iteration where the second option is faster.

## Option 1: Deploying to your own GitHub Pages sub domain

- Replace `coreos` with your GitHub username in `docs/_config.yml` on top of
  your other changes:
  ```
  docs/_config.yml | 2 +-
   1 file changed, 1 insertion(+), 1 deletion(-)

  diff --git a/docs/_config.yml b/docs/_config.yml
  index 3ab720a0..801cbb9d 100644
  --- a/docs/_config.yml
  +++ b/docs/_config.yml
  @@ -1,7 +1,7 @@
   title: coreos/coreos-installer
   description: CoreOS Installer documentation
   baseurl: "/coreos-installer"
  -url: "https://coreos.github.io"
  +url: "https://your_github_username.github.io"
   # Comment above and use below for local development
   # url: "http://localhost:4000"
   permalink: /:title/
  ```
- Push the full changes to the main branch of your GitHub repo fork
- Enable GitHub Pages for the main branch, using `/` as root
- Wait for approximately 1 min for the changes to be deployed
- Access the rendered pages under your username as domain:
  <https://your_github_username.github.io/coreos-installer/>

## Option 2: Local testing

- In `docs/_config.yml`, replace the line
  ```
  url: "https://coreos.github.io"
  ```
  by
  ```
  url: "http://localhost:4000"
  ```
- Use the following commands to install the Ruby gems and start a local
  development server:
  ```
  export JEKYLL_ENV="production"
  bundle install --path=./vendor/gems/
  bundle exec jekyll serve --livereload --strict_front_matter
  ```
- Access the documentaion by pointing your browser to
  <http://localhost:4000/project-name/>
