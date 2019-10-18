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

# MORE INFORMATION COMING SOON

Lets move on to doing someing with this data in [Part 3: Math]

[Square Sales Commissions Posting]: /jekyll/update/2018/03/05/Square-Sales-Commissions-Calculator.html
[Part 3: Math]: /jekyll/update/2018/03/08/Square-Commissions-Part-3-Math.html