[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bhttps%3A%2F%2Fgithub.com%2Fsymphonyoss%2Fbot-butler.svg?size=large)](https://app.fossa.io/projects/git%2Bhttps%3A%2F%2Fgithub.com%2Fsymphonyoss%2Fbot-butler?ref=badge_large)

[![Symphony Software Foundation - Active](https://cdn.rawgit.com/symphonyoss/contrib-toolbox/master/images/ssf-badge-incubating.svg)](https://symphonyoss.atlassian.net/wiki/display/FM/Incubating) [![Build Status](https://travis-ci.org/symphonyoss/bot-butler.svg)](https://travis-ci.org/symphonyoss/bot-butler) [![Dependencies](https://www.versioneye.com/user/projects/58ac50944ca76f0047de1847/badge.svg?style=flat-square)](https://www.versioneye.com/user/projects/58ac50944ca76f0047de1847?child=summary)

# Bot Butler
The Bot Butler is a collection of scripts for [Hubot](https://hubot.github.com/) that have been built and tested against [Symphony](http://www.symphony.com), using the [Hubot Symphony adapter](https://github.com/symphonyoss/hubot-symphony).

Scripts can be tested locally or deployed as a Docker container; additional scripts can be easily defined and distributed.

## Local run
1. `git clone https://github.com/symphonyoss/bot-butler.git ; cd bot-butler` - Checkout the bot-butler project
2. `cp env.sh.sample env.sh` - Configure environment variables (more on configuration below)
3. [Install NodeJS](https://nodejs.org/en/download/) 4.0 or higher
4. `npm install` - Install all project dependencies (in `./node_modules` folder)
5. `npm run generate-hubot` - Generate the hubot butler bot in `./butler-build` folder; if the `./butler-build` already exists, it won't be overridden
6. `npm run start-bot-butler` - Run it

To clean the generated bot simply type `npm run clean`; to know more about `npm run` scripts, checkout [package.json](package.json).

Please note that, between steps 5 and 6, you can edit the ./butler-build contents, change dependency versions, add more hubot scripts dependencies and more; alternatively, you can edit `bootstrap/build.sh` to add any configuration step after the `yo` command.

## Docker run
1. Checkout project and set environment variables (as above, steps 1 and 2)
2. Create the Docker image - `docker build -t butler:v0.9.0 .`
3. Run the Docker image - `docker run butler:v0.9.0`

The root folder of the project will be mounted on `/home/butler`; as such, the following folder will be inherited by default:
- `src/scripts` into `/home/butler/butler-build/scripts`
- `./env.sh` into `/home/butler/butler-build/env.sh`
- `./certs` into `/home/butler/butler-build/certs`

To override default values, use the following syntax:
```
docker run -v /myscripts:/home/butler/scripts -v /mycerts:/home/butler/certs butler:v0.9.0
```

To check the Docker image contents, without launching the bot, run:
```
docker run -it --entrypoint /bin/bash butler:v0.9.0
```

## Configuration
Bot configurations are defined by environment variables in `env.sh` file, located in the root folder of the project; since the file may contain sensitive information, it is [ignored by github](.gitignore); a `env.sh.sample` file is provided to copy from.

Below are the configuration items:

- Symphony Pod coordinates: the Symphony API endpoint coordinates; make sure they point to the Symphony Pod you want to use and that you have access to it; default values are pointing to the [Foundation Open Developer Platform](https://symphonyoss.atlassian.net/wiki/display/FM/Open+Developer+Platform):
```
export HUBOT_SYMPHONY_HOST=foundation-dev.symphony.com
export HUBOT_SYMPHONY_KM_HOST=foundation-dev-api.symphony.com
export HUBOT_SYMPHONY_SESSIONAUTH_HOST=foundation-dev-api.symphony.com
export HUBOT_SYMPHONY_AGENT_HOST=foundation-dev-api.symphony.com
```
Read more on [hubot-symphony](https://github.com/symphonyoss/hubot-symphony).

- Bot Certificates: Ensure that you load your Bot user certificate and PrivateKey into the `./certs` folder
```
export HUBOT_SYMPHONY_PUBLIC_KEY=./certs/bot-PublicCert.pem
export HUBOT_SYMPHONY_PRIVATE_KEY=./certs/bot-PrivateKey.pem
export HUBOT_SYMPHONY_PASSPHRASE=changeit
```

If you want to know more about how to generate and register a certificate for your Symphony pod, checkout the [Foundation certificate-toolbox](http://github.com/symphonyoss/certificate-toolbox); bear in mind that:
1. `HUBOT_SYMPHONY_PUBLIC_KEY` must point to `user/<bot-name>-cert.pem`
2. `HUBOT_SYMPHONY_PRIVATE_KEY` must point to `user/<bot-name>-key.pem`
3. `user/<bot-name>-key.pem` have no password; set it using `openssl rsa-in ./user/<bot-name>-key.pem -out ./user/<bot-name>.key.pem -des3`

Each script defined in [src/scripts](src/scripts)) may need some environment variables to be configured, for example:
```
export HUBOT_JIRA_URL=https://symphonyoss.atlassian.net
export HUBOT_JIRA_USERNAME=username
export HUBOT_JIRA_PASSWORD=password
```

### Proxy configuration
If your Hubot instance will be behind a Proxy, you will need to follow these instructions prior to a local or Docker run.

1. Edit `bootstrap/start.sh` and add - at the beginning of the file - the following statements, which configure NodeJS to use a Proxy server for outbound connections:
```
export http_proxy=http://proxy.company.com:8080
export https_proxy=http://proxy.company.com:8080
```
2. Create `src/scripts/_proxy.coffee` (underscore is intentional in the naming so the script is loading first!) to forward all HTTP requests:
```
proxy = require 'proxy-agent'
module.exports = (robot) ->
 robot.globalHttpOptions.httpAgent =
proxy('http://proxy.company.com:8080', true)
 robot.globalHttpOptions.httpsAgent =
proxy('http://proxy.company.com:8080', true)
```

## Customise scripts

### Writing a custom script
Want to write your own Hubot script? The best way is to take a look at [an existing script](src/scripts) and follow the [hubot-scripts documentation](https://www.npmjs.com/package/hubot-scripts).

All custom Hubot scripts in [src/scripts](src/scripts) are included in the bot; to define a sub-set of them, you can define a `hubot-scripts.json` file, which is created by `npm run generate-hubot` command and is empty by default.

Any third-party dependencies for scripts need the addition of your package.json otherwise a lot of errors will be thrown during the start up of your hubot. You can find a list of dependencies for a script in the documentation header at the top of the script.

### Distributing a custom script
Hubot scripts can be automatically referenced if:
- listed in the [hubot-scripts organization](https://github.com/hubot-scripts) page
- [tagged on npmjs as *hubot-scripts*](https://www.npmjs.org/browse/keyword/hubot-scripts)

The easiest way is to [publish the npm package](https://docs.npmjs.com/getting-started/publishing-npm-packages) using *hubot-scripts* as tag; read more on https://www.npmjs.org/browse/keyword/hubot-scripts

## Project dependencies
Bot Butler scripts depend on the following components:
- [Hubot](https://hubot.github.com/)
- [Hubot Symphony adapter](https://github.com/symphonyoss/hubot-symphony)
- [Yeoman](https://www.npmjs.com/package/yo)
- [Generator Hubot](https://www.npmjs.com/package/generator-hubot)

The [build.sh](bootstrap/build.sh) script invokes Yeoman, which generates a node project from scratch, using the generator-hubot; as such, the `package.json` `dependencies` definition doesn't impact the final bot runtime and it's only used to report on dependency metadata, such as security vulnerabilities and license compliance.

The [Versioneye dashboard](https://www.versioneye.com/user/projects/58ac50944ca76f0047de1847?child=summary) reads the `yarn.lock` file committed in the root project folder; to update it:
1. [Install Yarn](https://yarnpkg.com/lang/en/docs/install/#mac-tab)
2. run `yarn install --force`
3. Push any change to `yarn.lock`
