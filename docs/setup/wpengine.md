# WPEngine
Intro to WPEngine

## Launch Setup
To launch a site hosted on WPEngine, follow the below steps:

1. Confirm the site URL with the program (ex. chinapower.csis.org)
2. Contact Mary Van Engelen to request a DNS change
3. In WPEngine, go to the site's install -> Domains
4. Click "Add Domain" and enter the new URL
5. Click "Add redirect" on the [INSTALL].wpengine.com domain
6. Go to the install's phpMyAdmin
7. Navigate to the `wp_options` table
8. Set both the `siteurl` and `home` options to the new URL

---

## Setting up SSL
1. In WPEngine, go to the site's install -> "SSL"
2. Click "Add Certificate"
3. Choose the "Let's Encrypt" free certificate
4. Check the WP-Login & WP-Admin boxes and save
5. When the SSL certificate is no longer pending, expand the options and select the "Secure all URLs" option
6. Save Changes

---

## TravisCI Setup
Using TravisCI, it is possible to automate the deployment of code from GitHub to WPEngine. The instructions below outline the process and include sample scripts.

### Create an SSH Key
1. Follow the steps for [Generating a new SSH key](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#generating-a-new-ssh-key)
2. Do not set up a passphrase.
4. Copy your public key with `pbcopy < ~/.ssh/id_rsa.pub`

### Setup Git Push on WPEngine
1. Login to WPEngine and navigate to your install -> Git push
2. Fill out the developer name and the SSH public key. The public key is the contents you just copied from your terminal.

### Create Encrypted File for Deployment (NAME_travis.enc)
1. Navigate to the repo directory on your machine
2. Follow the steps for [Automated Encryption](https://docs.travis-ci.com/user/encrypting-files/#Automated-Encryption)
  1. Install Travis CI CLI if you haven't already
  2. Login using `travis login`
  3. Run `travis encrypt-file ~/.ssh/id_rsa`
  4. Copy the resulting `openssl aes-256-cbc -K $encrypted_0a6446eb3ae3_key -iv $encrypted_0a6446eb3ae3_iv -in super_secret.txt.enc -out super_secret.txt -d` line to `.travis.yml`

### Final Steps
1. Using the templates, create a `.travis.yml` and `deploy.sh` file for the repo in the root directory of the repo.
2. Make sure `.travis.yml`, `deploy.sh`, and `super_secret.txt.enc` are added to the `.gitignore`

### Example .travis.yml
```yml
language: ruby
cache: bundler
rvm: 2.3.1
branches:
  only:
  -  [BRANCH]
addons:
  ssh_known_hosts: git.wpengine.com
script:
- cd wp-content/themes/[THEME NAME]
- compass compile
before_install:
- openssl aes-256-cbc -K $encrypted_06196c1cb5d9_key -iv $encrypted_06196c1cb5d9_iv
  -in [NAME]_travis.enc -out /tmp/[NAME]_travis -d
- gem install compass
before_deploy:
- cd $TRAVIS_BUILD_DIR
deploy:
- provider: script
  skip_cleanup: true
  script: chmod +x deploy.sh && sh deploy.sh
  on:
    branch: [BRANCH]
```

### Example deploy.sh
```bash
chmod 600 /tmp/[NAME]_travis
eval "$(ssh-agent -s)" # Start the ssh agent
ssh-add /tmp/[NAME]_travis
git remote add [GIT REPO NAME] git@git.wpengine.com:staging/[WPEngine Install].git # add remote for staging site
git fetch --unshallow # fetch all repo history or wpengine complain
git status # check git status
git checkout [BRANCH] # switch to development branch
git add wp-content/themes/[THEME]/*.css -f # force all compiled CSS files to be added
git commit -m "compiled assets" # commit the compiled CSS files
git push -f [GIT REPO NAME] [BRANCH]:master #deploy to staging site from development to master
```