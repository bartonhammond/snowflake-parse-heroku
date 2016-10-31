# parse-server for Snowflake

Example project using:
- The [parse-server](https://github.com/ParsePlatform/parse-server) module on Express
- Can run parse-server locally or deploy it remotely on Heroku
- Uses Mailgun for email confirmation and password resets

Additional reading:
- Full Parse Server guide here: https://github.com/ParsePlatform/parse-server/wiki/Parse-Server-Guide
- Other deployment options beyond Heroku can be found here: https://github.com/ParsePlatform/parse-server-example

#### Test with your snowflake app
* For your convenience, an instance of parse-server is running remotely on Heroku
* Follow instructions to install snowflake app at `https://github.com/bartonhammond/snowflake`
* Copy src/lib/config.example.js to src/lib/config.js 
* Set `parseRemote: true`
* Set `hapiRemote: false`, `hapiLocal: false`, `parseLocal: false`
* Set Parse fields:
	- Set `appId: snowflake`
	- Set `masterKey: myMasterKey`
	- Set remote `url: http://snowflake-parse.herokuapp.com/parse`
* Run your snowflake app and confirm that you can Register, Update Log out, Reset password, etc.

### Steps to setup your own parse-server for Snowflake

#### Obtain a domain name
* Mailgun requires you to have a custom domain from which to send emails
* If you don't have a custom domain name already, obtain one now (Namecheap and GoDaddy are popular)
* Namecheap will be used as the domain name provider in this example (`https://www.namecheap.com`)

#### Signup for a free Mailgun account
* Mailgun will be used to send email confirmation and password reset emails
* Register at `https://mailgun.com/signup`
* Click on `Domains` in the upper menu bar
* Click on `Add New Domain` and enter your custom domain name
* Click on `Domain Verification & DNS`
* Important Note: You will find information here on configuring `TXT` and `MX` records.
* However, you will need to make slight modifications to this information in the next steps

#### Configure your domain name registrar to use Mailgun for email
* Log back into your domain registrar if necessary (assumes `Namecheap` in this example)
* Select `Domain List` from the left menu bar
* Select your custom domain
* Click on `Manage`
* Click on `Advanced DNS`
* The following configuration steps are specific to using Namecheap
* They require you to make slight but important modifications to the configuration information provided by Mailgun.
* If you are using a different registrar, then you can obtain additional information here `https://help.mailgun.com/hc/en-us/articles/202052074-How-do-I-verify-my-domain-`
* Add first `TXT` record from Mailgun, the one with value that starts with `v=spf1`, however use @ as the host name instead of using your actual custom domain
* Add second `TXT` record from Mailgun, the one with value that starts with `k=rsa;`, however, use `smtp._domainkey.` (i.e. don't append your custom domain).  Don't forget to include the trailing `.`
* Add both `MX` records, but use `@` as the hostname in both cases
* We skip adding the `CNAME` record in this example, as this is used for email tracking purposes only
* These changes will take up to 30 minutes to propagate before they become available to Mailgun

#### Confirm Mailgun is ready to use email
* Click on `Domains` in the upper menu bar
* Select your domain
* Click on `Domain Verification & DNS`
* Click on `Check DNS Records Now`
* Confirm that the `Current Value` field matches the `Enter This Value` field for both `TXT` and both `MX` records
* Additional information is available here `https://help.mailgun.com/hc/en-us/articles/202052074-How-do-I-verify-my-domain-`
* If necessary, obtain support from Mailgun and/or your domain registrar

#### Obtain Mailgun API Keys
* In Mailgun, click on your username in top right corner
* Select `Security` from the dropdown
* Select the `API Keys` tab
* Copy the `Active API Key`

Here's sample curl code for sending an email using your custom domain:

```
curl -s --user '<Active API Key>' \ 
https://api.mailgun.net/v3/samples.mailgun.org/messages \
 -F from='Snowflake Server <info@<your-domain-name>' \
 -F to='<enter the receiving email address>' \
 -F subject='Hello' \
 -F text='Testing some Mailgun awesomeness!'
```

#### Setup Heroku
* Setup parse-server on Heroku first since it will also setup MongoDB
	- Then you can run parse-server either locally or remotely at Heroku
	- In both cases, you'll be running MongoDB remotely
* Click the Heroku Button

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy)

* Complete the Heroku deployment form: 
	- Enter your Heroku `App_Name` -- this must be unique across all heroku apps (or let Heroku pick name for you)
	- Select the region to deploy your server (i.e. United States or Europe)
	- mLab MongoDB is automatically selected as an add-on
*  Next, you will be asked to configure the following environmental variables:
```
  PARSE_MOUNT:          Accept the default (`parse`)
  APP_ID:               Must match <appId> from the snowflake react-native src/lib/config.js file
  MASTER_KEY:           Must match <masterKey> from the snowflake react-native src/lib/config.js file
  SERVER_URL:           Replace 'yourappname' to match your Heroku 'App_Name' (http://<yourappname>.herokuapp.com/parse)
```

* Add the following additional environment variables which will be used to configure `parse-server`:
	- Click on `Settings` in the upper menu bar
	- Click on `Reveal Config Vars`
	- Set the following `Key` and `Value` pairs:
```	
  MONGODB_URI:          This variable is created automatically for you -- don't modify its value
  APP_NAME:             Choose the app name you want to appear in email confirmation & password reset and emails					
  MAILGUN_API_KEY:      Must match Mailgun `Active API Key`
  MAILGUN_DOMAIN:       Must match your <custom domain name>
  MAILGUN_FROM_ADDRESS: Use <anything>@<custom domain name>
  VERBOSE:              Set to true so Heroku keeps detailed logs
```
	
#### Deploy parse-server locally
* Configure the same set of environment variables in the terminal
* Copy the `./setenv-copy` to `./setenv`
* Edit the environment varaiables in the `./setenv` file
* Then source it: `source ./setenv`.

```
  export APP_ID= "<same as above>"
  export APP_NAME="<same as above>"
  export MAILGUN_API_KEY="<same as above>"
  export MAILGUN_DOMAIN="<same as above>"
  export MAILGUN_FROM_ADDRESS="<same as above>"
  export MASTER_KEY="<same as above>"
  export MONGODB_URI="<same as above>"
  export PARSE_MOUNT="<same as above>"
  export SERVER_URL="http://localhost:1337/parse"
```
	
* You'll need to relaunch MacOS terminal to set the new environment variables
* Type `npm install` from your `parse-server-example` project directory
* Type `npm start` to start `parse-server`

Example request to a server running locally:

```
curl -X POST \
  -H "X-Parse-Application-Id: <APP_ID>" \
  -H "Content-Type: application/json" \
  -d '{"score":1337,"playerName":"Sean Plott","cheatMode":false}' \
  http://localhost:1337/parse/classes/GameScore
  
curl -X POST \
  -H "X-Parse-Application-Id: myAppId" \
  -H "Content-Type: application/json" \
  -d '{}' \
  http://localhost:1337/parse/functions/hello
```

#### Test local parse-server with your snowflake app
* Follow instructions to install snowflake app at `https://github.com/bartonhammond/snowflake`
* Copy src/lib/config.example.js to src/lib/config.js 
* Change config.js as
```
  module.exports = {
    SESSION_TOKEN_KEY: 'SESSION_TOKEN_KEY',
  backend: {
    hapiRemote: false,
    hapiLocal: false,
    parseRemote: false,
    parseLocal:  true
  },
  PARSE: {
    appId: <APP_ID>,              // must match `APP_ID` environmental variable
    masterKey: '<MASTER_KEY>',    // must match `MASTER_KEY` environmental variable
    local: {
      url: '<LOCAL_SERVER_URL>',  // must match *local computer's* `SERVER_URL` environmental variable
    },
    remote: {
      url: '<REMOTE_SERVER_URL>'  // must match *heroku's* SERVER_URL environmental variable
    } 		
  }
}
```

#### Deploy parse-server on Heroku
* Clone this parse-server for Snowflake repo and change directory to it
* Make sure you have at least Node 4.3. `node --version`
* Download and install Heroku CLI from `https://devcenter.heroku.com/articles/heroku-command-line#download-and-install`
* Login into Heroku:
	- Type `heroku login`
	- Provide your Heroku credentials
* Setup a git `remote` to Heroku
	- Type `heroku git:remote -a <Heroku_App_Name>`
* Push your repo to Heroku:
	- Type `git push heroku master`

#### Test remote parse-server with your snowflake app
* Modify src/lib/config.js 
```
  backend: {
    hapiRemote: false,
    hapiLocal: false,
    parseRemote: true,
    parseLocal:  false
  },
```

#### Deploy parser-server-dashboard
* Parse server dashboard allows you to inspect the contents of the parse database
* It's to confirm the proper operation of the parse-server when its running either locally and remotely
* Clone the parse-dashboard from `https://github.com/ParsePlatform/parse-dashboard#getting-started`
* Install the dashboard:
```
  npm install
```
* modify the `package.json` file by adding the following two lines in the existing `scripts` section:
```
  "scripts": {
    "local": "node ./Parse-Dashboard/index.js --appId snowflake --masterKey myMasterKey --serverURL http://localhost:1337/parse",
    "remote": "node ./Parse-Dashboard/index.js --appId snowflake --masterKey myMasterKey --serverURL https://snowflake-parse.herokuapp.com/parse"
  }
```
* To run parse-dashboard against the local version of parse-server, type:
```
  npm run local
```

* To run parse-dashboard against the local version of parse-server, type:
```
  npm run remote
```
