---
layout: post
title:  "Square Commissions: Part 1 (Defining Modules And Dependencies)"
date:   2018-03-06 21:04:19 +0900
categories: jekyll update
---

*(This posting is part of a sequence, to see the project overveiw please check out the [Square Sales Commissions Posting]), you'll need to complete the setup setup there before starting this section.*

# OVERVIEW
Picking up where the last posting left off, we're ready to start incorporating the Square API and your database of choice in this posting to pull in the information we'll need to calculate our commisions.

# BUILDING THE DRIVING MODULE
Ideally we won't always be accessing this information through a CLI, it would be nice to incorproate this tool into some sort of web server at some point. In order to do this we're going to extrapolate the logic into it's own module that can be run to produce the results we want.  Lets call this new module `comCalculator.js`.  So in your text editor go ahead a create a new file in the root directory and call it `comCalculator.js`.

Lets start by adding some documentatin to the top of the file, and our base level commenting, for good measure.

{% highlight javascript %}
/*
*	COMMISSION CALCULATOR
*
*	This is the commission calculator module
*/

//  DECLARE DEPENDENCIES

//  DEFINE THE MODULE

//  DEFINE LOCAL VARIABLES

//  NOTIFY PROGRESS

//  EXPORT THE MODULE
{% endhighlight %}

Next, our module is little more than a javscript object, so lets create it, calling it `commissionCalculator` and adding a simple test function that we can run from outside the module to make sure its working.  The code should look something like this:

{% highlight javascript %}
//  DEFINE THE MODULE
let commissionCalculator = {
    test: function() { console.log('good comCalc test'); }
};
{% endhighlight %}

Then, to export the module for use in other scripts we use `module.exports` that is built into node and reference the object we wish to export:

{% highlight javascript %}
//  EXPORT THE MODULE
module.exports = commissionCalculator;
{% endhighlight %}

# ACCESSING THE COMMISSIONS CALCULATOR MODULE
Now we can go back to our `cli.js` script and add the `comCalculator.js` as one of our dependencies by requiring it at the top of our file:

{% highlight javascript %}
//  DECLARE DEPENDENCIES
let comCalculator 	= require('./comCalculator.js');
{% endhighlight %}

You can see here we're requiring the script from our root folder, now the contents of that module can be accessed in the `cli.js` script through the `comCalculator` object.  Lets make sure it works by adding a quick notification to our `cli.js` file:

{% highlight javascript %}
//   NOTIFY PROGRESS
console.log(dateparam);
comCalculator.test();
{% endhighlight %}

Save both files.  If we go back to the command line now and type

`$ node cli 2018-03-06`

We shoudl get the following response:

**2018-03-06**

**good test**

If thats what you got, congrats, we're ready to start adding our module that will communicate with the square API.  Before we do that though, make sure to commit your changes and push your code to gitHub.

`$ git add .`

`$ git commit -m "Add: comCalculator Module"`

`$ git push origin master`

# ADDING A SQUARE MODULE
Similarly to the way we added our comCalculator module, lets create a module that will be dedicated to everything we need to do square API wise.  So in our text editor, in our root directory lets create a new script and call it `squareAPI.js`.  Like we did last time lets start with some basic structure in this file:

{% highlight javascript %}
/*
*	SQUARE API
*
*	This is module handles all of our square API needs
*/

//   DECLARE DEPENDENCIES

//   DEFINE THE MODULE
let squareAPI = {
    test: function() { console.log('good square test'); }
};

//   DEFINE LOCAL VARIABLES

//   NOTIFY PROGRESS

//  EXPORT THE MODULE
module.exports = squareAPI;
{% endhighlight %}

We haven't done anything new here, just some different values from our **comCalculator** module.

Alright, this is where the fun starts. There are a couple of ways that we could access the square API, the easist by far is to use [Square Connect Node SDK], it can be found at npmjs.com.  Which brings up another consideration, in order to use this package we'll need to setup node package managment on this project.

To do this head to the command line and type:

`$ npm init`

This will kick off the creation of a `package.json` file for our project.  We're just going to follow the onscreen instructions, but I can also give you my answers if that is helpful:

`package name: (sqrcomcalc)` just hit enter to keep sqrcomcalc as the package name

`version: (1.0.0)` don't need a version name unless you want one, just hit enter

`description:` I'm going to say `square commissions calcualtor` then hit enter

`entry point: (cli.js)` for now we don't have to do anything here, but if we no longer wanted to access this through teh CLI we would change this.  Just hit enter for now

`test command:` nothing to do here right now, just hit enter

`git repository: (https://github.com/ianemcallister/sqrComCalc.git)` yours will list your repository, just hit enter

`keywords:` don't have to put anything here, but you can if you want, just hit enter

`author:` I will enter my name, `Ian McAllister` and hit enter

`license: (ISC)` the generic one is fine for now, just hit enter

Then it will show you the object it created:

{% highlight javascript %}
{
  "name": "sqrcomcalc",
  "version": "1.0.0",
  "description": "square commissions calculator",
  "main": "cli.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/ianemcallister/sqrComCalc.git"
  },
  "author": "Ian McAllister",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/ianemcallister/sqrComCalc/issues"
  },
  "homepage": "https://github.com/ianemcallister/sqrComCalc#readme"
}
{% endhighlight %}

`Is this ok? (yes)`, if it all looks good then hit enter.  

Congrats, you've setup npm with this project!

# USING .GITIGNORE
This is a great opportunity to add a .gitIgnore file.  This is an important file because it identifies files and directories that will remain in your local repository but not be pushed to your public repository.  

**Why do we do this?**

There are a couple of reasons to use .gitIgnore.  First there are some files that don't need to be moved up to the public repository, like all of your npm package files.  Anyone who is going to download your repository is going to have to install all of the packages on their machine anyway and installing them from NPM rather than from your repository is the best way to make sure that they're as up to date as they can be. 

Also there are security considerations, such as config files that you don't want thrown up on a public repository where anyone can see them, or worse, scrape them.  

So, we're going to build a .gitIgnore file to identify the files we don't want include in our git commits.

From your text editor create a new file and call it `.gitignore`, make sure us use that period before the characters, it's impotant.

For the sake of this example we're going to keep the file very simple, just add the following lines:

{% highlight javascript %}
# Dependency directories
node_modules/
{% endhighlight %}

This will ensure that we're not commiting our node_modules directory (that we're going to create in a moment) to our git repository.

# INSTALLING NPM PACKAGES
Alright, now we're ready to a couple of npm packages, so head to your command line and type the following:

`$ npm install square-connect --save`

This will install the **square-connect** package from npm and add it to our package.json file.

While we're here I'm also going to add **firebase-admin** because that's what I'll be using for my database.  We're not going to go much into [Firebase] in this tutorial but it's awesome and you should check it out.  We'll install the package using the following command line statement:

`$ npm install --save firebase-admin`

Now your `package.json` file should look something like this:

{% highlight javascript %}
{
  "name": "sqrcomcalc",
  "version": "1.0.0",
  "description": "square commissions calculator",
  "main": "cli.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/ianemcallister/sqrComCalc.git"
  },
  "author": "Ian McAllister",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/ianemcallister/sqrComCalc/issues"
  },
  "homepage": "https://github.com/ianemcallister/sqrComCalc#readme",
  "dependencies": {
    "firebase-admin": "^8.6.1",
    "square-connect": "^2.20190925.0"
  }
}
{% endhighlight %}

If this is what yours looks like, congratulations, you're doing great work!

# ADDING SQUARE MODULE
All that's left to do for this step is to add our square and firebase modules.  Because we already started creating this one lets finish it up.  Under the `//  DECLARE DEPENDENCIES` section of your `squareAPI.js` file in your text editor we need to require our **square-connect** package, we'll do so by adding the following:

{% highlight javascript %}
//  DECLARE DEPENDENCIES
let SquareConnect 	= require('square-connect');
let defaultClient 	= SquareConnect.ApiClient.instance;
{% endhighlight %}

We're then going to add a few lines below that:

{% highlight javascript %}
// Configure OAuth2 access token for authorization: oauth2
let _oauth2           = defaultClient.authentications['oauth2'];
_oauth2.accessToken   = process.env.SQUARE_APP_TOKEN;
{% endhighlight %}

**What's Happening Here?**

So in order to connect with square's server and access our transaction information we'll need to register a [Square Developer Account] with them.  After you create a login you'll be take to a page that looks like this:

![Square_Developer_Dash](/assets/squareDeveloperDash.jpg)

We're going to click **New Application** and we'll be taken to this screen:

![Square_App_Application](/assets/newApplicationSquare.jpg)

I called my project **CommissionsCalculator**.  Finally we'll be taken to the confirmation screen where we can look up our credentials.  I'm not going to show you my credentials (for obvious reasons)

![Confirmed_Square_App](/assets/confirmedAppSquare.jpg) 

Down in the lower left hand corner of the page you'll see a toggle switch that says **Sandbox Settings**.  Toggle that switch so it says **Production Settings**, for the sake of this example, we're not trying to charge any cards or save anything to the server, but we do want to get acutaly transaction data from the server, so we want the production access token, rather than the sandbox.

Under the **Credentials** tab in the **Production Settings** mode find the **Personal Access Token** section, click **show** to see the access token.  Select the full access token and copy and past it to your clipboard.

Next we're going to go into the shell and add this to our environment variables. 

# ADDING SQUARE APP TOKEN TO ENVIRONMENT VARIABLES
Sometimes it makes sense to use a config file, I like to use environment variables to keep secret tokens like this secret, then theres no chance that you accidently save them to git hub.  Also, if you end up running this application on a server somewhere services like [Heroku] make it easy to add these environment variables.  So in your terminal, open up a new tab because we're going to edit your environment file.  I'm on a mac so I'm going to show you that way, write the following command:

`$ nano .bash_profile`

this should open up your `.bash_profile`, we're going to get in and out quickly, so what you want to add is the following:

`export SQUARE_APP_TOKEN='[YOUR-ACCESS-TOKEN-GOES-HERE]'`

Then we're going to `cntl-X` to exit, it will ask us if we want to save, say yes. You're done with that terminal window, you can close it.

# CONFIRMING SQUARE API MODULE
At this point your `squareAPI.js` file should look something like this:

{% highlight javascript %}
/*
*	SQUARE API
*
*	This is module handles all of our square API needs
*/

//  DECLARE DEPENDENCIES
let SquareConnect 	= require('square-connect');
let defaultClient 	= SquareConnect.ApiClient.instance;

//   Configure OAuth2 access token for authorization: oauth2
let _oauth2 		= defaultClient.authentications['oauth2'];
_oauth2.accessToken = process.env.SQUARE_APP_TOKEN;

//   DEFINE THE MODULE
let squareAPI = {
    test: function() { console.log('good square test'); }
};

//   DEFINE LOCAL VARIABLES

//   NOTIFY PROGRESS

//   EXPORT THE MODULE
module.exports = squareAPI;
{% endhighlight %}

If your looks basically like that, great work!

# ADDING FIREBASE MODULE
We'll go through this part quickly because it's not the focus of our project (this could be a whole posting on its own, I LOVE firebase).  We're going to head back to your text editor, create a new file, and name it `firebaseAPI.js`.  Then the file should be setup to look like this:

{% highlight javascript %}
/*
*	FIREBASE API
*
*	This is module handles all of our firebase API needs
*/

//   DECLARE DEPENDENCIES
let admin = require("firebase-admin");

//   DEFINE GLOBAL VARIABLES
let serviceAccount = {
	"type":           process.env.FB_TYPE,
	"project_id":     process.env.FB_PROJECT_ID,
	"private_key_id": process.env.FB_PRIVATE_KEY_ID,
	"private_key":    process.env.FB_PRIVATE_KEY,
	"client_email":   process.env.FB_CLIENT_EMAIL,
	"client_id":      process.env.FB_CLIENT_ID,
	"auth_uri":       process.env.FB_AUTH_URI,
	"token_uri":      process.env.FB_TOKEN_URI
};

//   Initialize the app with a service account, granting admin privileges
admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  databaseURL: "https://[YOUR-FIREBASE-DB-NAME-HERE].firebaseio.com"
});

//	DEFINE THE MODULE
let firebaseAPI = {
    test: function() { console.log('good firebase test'); }
};

//	DEFINE LOCAL VARIABLES

//	NOTIFY PROGRESS

//  EXPORT THE MODULE
module.exports = firebaseAPI;
{% endhighlight %}

Same as with the square module we need to add all of the firebase secrets as environment variables.

# ADDING SQUARE AND FIREBASE TO OUR COMMISSIONS CALCULATOR
Finally, the moment we've been working towards, now we can add our square and firebase modules to our commissions calculator module.  So in our text editor open `comCalculator.js` and under the `//  DECLARE DEPENDENCIES` comment add the following:

{% highlight javascript %}
//  DECLARE DEPENDENCIES
let square    = require('./squareAPI.js');
let firebase  = require('./firebaseAPI.js');
{% endhighlight %}

That's it, now they're all wired together. Lets commit all of our updates and move on to the next section, [Part 2: Collecting Data].

    $ git add .
    $ git commit -m "Add: SquareAPI and FirebaseAPI"
    $ git push origin master

[Square Sales Commissions Posting]: /jekyll/update/2018/03/05/Square-Sales-Commissions-Calculator.html
[Square Connect Node SDK]: https://www.npmjs.com/package/square-connect
[Firebase]: https://firebase.google.com
[Square Developer Account]: https://firebase.google.com
[Heroku]: https://www.heroku.com
[Part 2: Collecting Data]: /jekyll/update/2018/03/07/Square-Commissions-Part-2-Collecting-Data.html