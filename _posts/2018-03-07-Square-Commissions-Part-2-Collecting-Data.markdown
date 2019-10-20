---
layout: post
title:  "Square Commissions: Part 2 (Collecting Data)"
date:   2018-03-07 21:04:19 +0900
categories: jekyll update
---
*(This posting is part of a sequence, to see the project overveiw please check out the [Square Sales Commissions Posting])*

# LAUNCHING FROM OUR CLI
In order for this program to work how we want it to right now we want to be able to call the program from the command line with a statment like this:

`$ node cli 2018-03-07`

and have it return all of the employees and their commission earnings for **2018-03-07**.  In order to set this up lets go back to our `cli.js` file in our text editor, we're done with the `//NOTIFY PROGRESS` section right now so we can get rid of those console logs.  Then let add comments for a new section called `//RUN`.

In this run section we will only run the program if we have a valid **dateparam**.  We could do much more thorough validation of this variable, but for time being lets just check to make sure it's present, and if it's not lets throw an error.  So it will look something like this:

{% highlight javascript %}
//	NOTIFY PROGRESS

//	RUN
if(dateparam != undefined) {

} else {
	//	NOTIFY ERROR - if no valid dateparam is present
	console.error('INVALID DATA PARAMETER, PLEASE ENTER VALID DATE PARAMETER "YYYY-MM-DD"')
}
{% endhighlight %}

Within that succesful if block we're going to run our commissions calculator.  So we're going to call a function from our `comCalculator` module called `dailyCommissions`.  Don't worry, we haven't written this function yet, but we will shortly.  We're going to pass that function our `dateparam` variable, now here's the important part, all of this work needs to happen asyncronsly while we wait for our program to reach out to square's server, collect the data, process it, and return a report.  Javascript has a great tool to handle this sort of operation, **promises**.  The promise will wait until either a response returns or an error returns and will run a block of code accordingly.  For our purposes if it's a success lets print the report, if it wasn't successfull we want to see the error.  In that case our code should now look like this:

{% highlight javascript %}
//	RUN
if(dateparam != undefined) {

	//	RUN ASYNC WORK - only if a valid dateparam is present
	comCalculator.dailyCommissions(dateparam)
	.then(function success(commissionsReport) {

		//	NOTIFY RESULTS
		console.log(commissionsReport);

	}).catch(function error(e) {
	    console.error('There was an error:', e);
	});
	
} else {
	//	NOTIFY ERROR - if no valid dateparam is present
	console.error('INVALID DATA PARAMETER, PLEASE ENTER VALID DATE PARAMETER "YYYY-MM-DD"')
}
{% endhighlight %}

We're either going to return a successful report or we're going to return an error.

# ADDING DAILY COMMISSIONS FUNCTION
Lets head over to our `comCalculator.js` file in our text editor. In the **commissionCalculator** object, we're going to add our **dailyCommissions** function by writing `dailyCommissions: dailyCommissions,`.  Then we'll comment out a new section for `//  PUBLIC FUNCTIONS` and add a new function **dailyCommissions(date)**.  It should look something like this:

{% highlight javascript %}
//   DEFINE THE MODULE
let commissionCalculator = {
	dailyCommissions: dailyCommissions, 
    test: function() { console.log('good comCalc test'); }
};

//   DEFINE LOCAL VARIABLES

//   NOTIFY PROGRESS

//   PUBLIC FUNCTIONS
function dailyCommissions(date) {

};
{% endhighlight %}

It is good practice to document what a function is doing so you (or someone else) can figure it out later, so lets add that here.

{% highlight javascript %}
/*
*	DAILY COMMISIONS
*
*	This function 
*	@param {string} date to be searched YYYY-MM-DD
*	@return {Object} collection of employees and their commissions
*/
function dailyCommissions(date) {

};
{% endhighlight %}

Moving into this **Daily Commissions** function lets comment out some of the sections we're going to need

{% highlight javascript %}
function dailyCommissions(date) {
    //    DEFINE LOCAL VARIABLES
    //    RETURN ASYNC WORK
};
{% endhighlight %}

I always make space to add local variables, even before I know that I'll need them.  As for the async work, we're going to return the promise that we referenced form our `CLI.js` file.  It will look something like this:

{% highlight javascript %}
//    RETURN ASYNC WORK
return new Promise(function dailyCommissionsPromise(resolve, reject) {
    //    DEFINE LOCAL VARIABLES
});
{% endhighlight %}

[Promises] are great, and an important tool for web development that must handle many situations were it takes data a moment to travel back and forth between servers and/or clients.

Inside this promise we're going to need to pull together all of our information from our various sources, square, and our database.  Each of these will be their own promises, so we're goint to utilize the [Promise.all()] method in Javascript that will allow us to submit multiple promises and will wait for all of them to resolve before proceeding.

So inside our **dailyCommissionsPromise** function lets define our array of promises as the following:

{% highlight javascript %}
//    DEFINE LOCAL VARIABLES
let allData = [
    square.getDailyTransactions(date),
    firebase.getAllEmployees()
];

//    RETURN ASYNC WORK
return new Promise(function dailyCommissionsPromise(resolve, reject) {
    //    DEFINE LOCAL VARIABLES
    
});
{% endhighlight %}

So what we've done here is identify two functions (that we haven't written yet), that will each return their promises, that we want to resolve before we finish up processing our data.  In order to wait for them both to resolve we need to add the Promise.all() method:

{% highlight javascript %}
//    DEFINE LOCAL VARIABLES
let allData = [
    square.getDailyTransactions(date),
    firebase.getAllEmployees()
];

//    RETURN ASYNC WORK
return new Promise(function dailyCommissionsPromise(resolve, reject) {
    //    DEFINE LOCAL VARIABLES
    //    WAIT FOR ALL PROMISES TO RESOLVE BEFORE PROCESSSING DATA
    Promise.all(allDataPromises)
    .then(function success(allDataArray) {
        resolve(allDataArray)
    }).catch(function error(e) {
    	reject(e);
    });
});
{% endhighlight %}

If all the promises resolve successfully we'll receive the **allDataArray** back, if an error occured somewhere that error will be pass back up the chain to where the original promise was first called so it can be handled there.  Eventually we'll want to add more logic when the data returns successfully, but for the moment we're going to resolve the data back up the chain.  

This is also a good opportunity to commit our code before moving on to the next step.

    $ git add .
    $ git commit -m "Add: Daily Commissions Function"
    $ git push origin master

# GETTING OUR PROMISES WORKING
If we run our code right now we're going to recieve an error because we haven't written our two square and firebase promise functions yet.  In the interest of having a working program as quickly as possible lets rough out those functions so we can make sure that our data is flowing properly. 

First lets go into `squareAPI.js`.  In our module lets define our new function that we reference in the previous step, **getDailyTransactions(date)**:

{% highlight javascript %}
//	DEFINE THE MODULE
let squareAPI = {
    getDailyTransactions: getDailyTransactions,
    test: function() { console.log('good square test'); }
};
{% endhighlight %}

Next we'll add this function below under a section commented out as `// DEFINE PUBLIC FUNCITONS//` along with our commented explaination about what the function does, and our base structure of async work being returned.  We'll also resolve a simple messages so we can confirm everything is working properly:

{% highlight javascript %}
//	DEFINE PUBLIC FUNCTIONS
/*
*	GET DAILY TRANSACTIONS
*
*	This function accepts a date in YYYY-MM-DD format and returns all the square transactions for all locations on that date.
*
*	@param {string} date in YYYY-MM-DD format
*	@return {object} collection of transactions returned from square 
*/
function getDailyTransactions(date) {
	//	DEFINE LOCAL VARIABLES
	//	RETURN ASYNC WORK
	return new Promise(function getDailyTrainsactionsPromise(resolve, reject) {
            resolve('good transactions test');
	});
};
{% endhighlight %}

But before we run our program we need to jump over to our `firebaseAPI.js` script and add our **getAllEmployees()** function.  So same as before, lets pop into our module first and add our function:

{% highlight javascript %}
//	DEFINE THE MODULE
let firebaseAPI = {
    getAllEmployees: getAllEmployees,
    test: function() { console.log('good firebase test'); }
};
{% endhighlight %}

Then lets define the function below:

{% highlight javascript %}
//	DEFINE PUBLIC FUNCTIONS
/*
*	GET ALL EMPLOYEES
*
*	This function will reach out to our databse (firebase in this case) and return a list of employees with all their prertinant information.
*
*	@return {object} collection of employee data
*/
function getAllEmployees() {
	//	DEFINE LOCAL VARIABLES
	//	RETURN ASYNC WORK
	return new Promise(function getAllEmployeesPromise(resolve, reject) {
		resolve('good database test');
	});
};
{% endhighlight %}

Great!  That should be enough to ensure that our scripts are properly wired up.  Lets head back to our command line and make sure it is.

# TESTING FOR SUCCESS AND COMMITTING
From the terminal lets go a head and run:

    $ node cli 2018-03-07

If everything is working properly we should get back:

    [ 'good transactions test', 'good database test' ]

I'm going to take this opportunity to commit again, just for good measure, as we have a working program now:

    $ git add .
    $ git commit -m "Add: getDailyTransactions and getAllEmployees functions"
    $ git push origin master

# FINALIZING OUR DATABASE MODULE
The database is not the focus of this example so I'm not going to spend much time on it here, but for the sake of this tutorial we need the data from the database so lets fill it out.

Lets head back into our **getAllEmployees** function, here we're going to utilize **firebase-admin** to access all of our employee records from our database, the implementation looks something like this:



# MORE INFORMATION COMING SOON

Lets move on to doing someing with this data in [Part 3: Math]

[Square Sales Commissions Posting]: /jekyll/update/2018/03/05/Square-Sales-Commissions-Calculator.html
[Part 3: Math]: /jekyll/update/2018/03/08/Square-Commissions-Part-3-Math.html
[Promises]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
[Promise.all()]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all