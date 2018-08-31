---
title: Accounts
description: 'Quickly deploy your app'
disqusPage: 'Starterpack:Deployment'
---

# Deployment

## PM2 Meteor

You can setup your server on AWS or DigitalOcean. We recommend you use `Ubuntu 16.04` to quickly deploy without hassles.

We have the ability to quickly deploy our apps. One of the easiest way to do it, is to use `pm2-meteor`:

```
npm i -g pm2-meteor
```

Now go to your project root:

```
mkdir -p .deploy/qa
cd .deploy/qa
pm2-meteor init
```

Answer all the questions asked by pm2-meteor.

### Setting up the server

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
```

Run a new shell so it can recognise it, sometimes it doesn't:

```
bash
```

Install the node version

```
nvm i 8.11.4
npm i -g pm2
```

Setup links because `pm2-meteor` might complain:

```
sudo ln -s `which node` /usr/bin/node
sudo ln -s `which npm` /usr/bin/npm
sudo ln -s `which pm2` /usr/bin/pm2
```

Install MongoDB:
https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/

Quicker:

```
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo service mongod start
```

Deploy:

```
cd .deploy/qa
pm2-meteor deploy
```

If you have a multi-core server with 4 processors you can quickly scale like this:

```
pm2-meteor scale 4
```

Setup deployment script:

```
"deploy-qa": "cd .deploy/qa && pm2-meteor deploy"
```

```
npm run deploy-qa
```

## Alternatives

Mup, a docker containerised deployment tool: https://github.com/zodern/meteor-up

Or if you want to use AWS Beanstalkd you can use a `mup` extension: https://github.com/zodern/mup-aws-beanstalk

You can also use `now` if you prefer: https://github.com/jkrup/meteor-now

## GraphQL Playground

If you are looking to run the playground in your production app, you have to explicitly say so, because it only runs in development mode.

```js
import { initialize } from 'meteor/cultofcoders:apollo';

initialize(
  {
    introspection: true,
  },
  {
    gui: true,
  }
);
```

Enjoy!
