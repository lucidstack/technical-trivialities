---
layout: post
title:  "Elixir/Phoenix setup on Ubuntu with edeliver"
date:   2017-03-15 13:22:00 +0000
categories: elixir
---

In code blocks, `#` means root, while `$` means non-root user.

# Let's hop onto the server

1. Configure your locales, if they are not set; you want everything to be `en_US.UTF-8`
2. Follow instructions on [elixir-lang.org](http://elixir-lang.org/install.html#unix-and-unix-like) to install erlang + elixir. *NB:* Don't use asdf, it relies on bash aliases that don't work with deployment scripts. I tried and failed *miserably*
3. Install nodejs and npm.

        # apt-get install nodejs
        # apt-get install npm
        # ln -s `which nodejs` /usr/bin/node # some npm modules expect the 'node' command to be present

4. Install `brunch` with npm

        # npm install -g brunch

5. Locally, open the repository, and check out the `.deliver/config` file. It contains variables for the users you're gonna need on the server

        # where to build the release
        BUILD_HOST="xxx.xxx.xxx.xxx"
        BUILD_USER="elixir_build"
        BUILD_AT="/home/$BUILD_USER/$APP/builds"

        STAGING_HOSTS="xxx.xxx.xxx.xxx"
        STAGING_USER="elixir_staging"
        TEST_AT="/home/$STAGING_USER/$APP_staging"

        PRODUCTION_HOSTS="xxx.xxx.xxx.xxx"
        PRODUCTION_USER="elixir_deploy"
        DELIVER_TO="/home/$PRODUCTION_USER/$APP_live"

6. Obviously, feel free to edit the users, and use the set values in the next steps
7. Back on the server, create the user for building the app

        # adduser elixir_build

8. Create users for deploying the app _(if you are planning to build and run the app on the same server)_

        # adduser elixir_staging
        # adduser elixir_deploy

9. Authorize your ssh keys for all three users (add your ssh public key to `~/.ssh/authorized_keys`). If you had your key already authorised for the `root` user, use this:

        # mkdir ~elixir_build/.ssh
        # mkdir ~elixir_staging/.ssh
        # mkdir ~elixir_deploy/.ssh
        # cp ~/.ssh/authorized_keys ~elixir_build/.ssh
        # cp ~/.ssh/authorized_keys ~elixir_staging/.ssh
        # cp ~/.ssh/authorized_keys ~elixir_deploy/.ssh
        # chown -R elixir_build ~/.ssh
        # chown -R elixir_staging ~/.ssh
        # chown -R elixir_deploy ~/.ssh


# Let's deploy

That is the basic configuration for an Elixir+Phoenix server! Now, back on your local machine:

        $ mix edeliver build release
        $ mix edeliver deploy release to production

and start the server

        $ mix edeliver start production

Alternatively, you can deploy and start the release in one command

        $ mix edeliver build release
        $ mix edeliver deploy release to production --start-deploy


This is what's happening behind the curtains:
* `mix edeliver build release`
  * Push your commits to the `$BUILD_HOST` server
  * Copy your secrets file onto the server
  * Build a `.tar.gz` file with all the needed files (including things like compiled assets and secrets)
  * Copy the `.tar.gz` back in your repo, in gitignored folder
* `mix edeliver deploy release to production`
  * Copy the `.tar.gz` file containing your release onto the `$PRODUCTION_HOSTS` server
  * Unpack this file
* `mix edeliver start production`
  * Start the release through a single binary, containing all the needed code

After this, you should be able to access `http://$PRODUCTION_HOSTS:4000`.
As a last step, you might want to install `nginx` to expose the web server on port 80 and proxy your server through HTTPS.
