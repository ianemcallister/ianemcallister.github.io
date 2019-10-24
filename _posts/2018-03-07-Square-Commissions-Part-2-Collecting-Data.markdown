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

Lets head back into our **getAllEmployees** function, here we're going to utilize **firebase-admin** to access all of our employee records from our database, we're going to remove our original resolve and the new implementation looks something like this:

{% highlight javascript %}
function getAllEmployees() {
	//	DEFINE LOCAL VARIABLES
	let ref = admin.database().ref('employees');

	//	RETURN ASYNC WORK
	return new Promise(function getAllEmployeesPromise(resolve, reject) {
		//	ACCESS DATABASE VALUES
		ref.once("value")
		.then(function(snapshot) {

			//	PASS THE VALUES BACK
			resolve(snapshot.val());

		});
	});
};
{% endhighlight %}

Lets talk about what is happening here.  At the top of our our `firebaseAPI.js` script we initilaized **firebase-admin**.  Now when we reference our database **admin.database()** we tell firebase the records we're looking for, **ref('employees')**.  Further down in the promise we're goint to use the *ref* variable we defined and collect the value at the **employees** path in our database.  Because fireabase had the capacity to setup 3-way binding we're specifiing here that we only want to collect the data **once**, and that we are looking for the **value** at that address.  This operation returns a snapshot and we pass the value of that snapshot back through our resolve.  Again, this isn't the focus of this tutorial so I'm not going to spend much more time here, but this will collect the data that we need.

# DOWNLOADING SQUARE TRANSACTION DATA
The crux of our example, lets head back into our `squareAPI.js` script.  We're going to start off in our module object and we're going to add a private function **_listTransactions** like this:

{% highlight javascript %}
//	DEFINE THE MODULE
let squareAPI = {
    _listTransactions: _listTransactions,
    getDailyTransactions: getDailyTransactions,
    test: function() { console.log('good square test'); }
};
{% endhighlight %}

We'll then head down and comment out a new section for `//  DEFINE PRIVATE FUNCTIONS` and add in our new function:

{% highlight javascript %}
//	DEFINE PRIVATE FUNCTIONS
/*
*	PRIVATE: LIST TRANSACTIONS
*
*	This function wraps up the square listTransactions function so we can handle recursive calls to handle batch size
*
*	@param {string} square location Id
*	@param {string} beginTime in RFC 3339 format
*	@param {string} endTime in RFC 3339 format
*	@param {string} cursor is a pagination cursor returned by a previous call to this endpoint. Provide this to retrieve the next set of results for your original query.
*	@return {array} an array of transactions collected from square
*/
function _listTransactions(locationId, beginTime, endTime, cursor) {
	//	DECLARE LOCAL VARIABLES
	//	RETURN ASYNC WORK
	return new Promise(function sqListTransactionsPromise(resolve, reject) {

	});
};
{% endhighlight %}

The reason we're adding this warpper is that square delivers transactions in batches of up to 100 transactions at a time. Anything beyond 100 transactions gets split into multiple batches or pages, we need to pass the cursor back to square to get the additional pages of results.  Because we don't know how many pages we're going to be dealing with we'll set this up as a function we can call recursivly until we get to the end of the list.

Lets add the local variables we'll need for this function to the top of the function. **transactionlist** will be the collection of all the transactions from all of the colleced batches, **apiInstance** will be used to call the square function, and **opts** defines the start and end time that transactions should fall between, cursor will be a value returned from our first call:

{% highlight javascript %}
function _listTransactions(locationId, beginTime, endTime, cursor) {
	//	DECLARE LOCAL VARIABLES
	let transactionlist = [];
	let apiInstance = new SquareConnect.PaymentsApi();
	let opts = { 
		'beginTime': beginTime,
		'endTime': endTime,
		'cursor': cursor,
		'locationId': locationId
	};
{% endhighlight %}

Next, inside the promise we're going to call the function that does all the work, **apiInstance.listPayments** with our specific paramters:

{% highlight javascript %}
//	RETURN ASYNC WORK
return new Promise(function sqListTransactionsPromise(resolve, reject) {
    //	CALL 
    apiInstance.listPayments(opts)
    .then(function success(origTxsBatch) {

    }).catch(function error(e) {
        reject(e);
    });

});
{% endhighlight %}

We're passing it our **opts** variables above as a parameter.  In case comething goes wrong we've got our **error** function, if everything goes as planned we'll need to fill out the **success** function.  We'll start by checking for pagination:

{% highlight javascript %}
.then(function success(origTxsBatch) {
    //	CHECK FOR PAGINATION
    if(origTxsBatch.cursor != undefined) {
        
    } else {

    }
{% endhighlight %}

If a cursor was found that means that our transactions have been seperated into seperate pages, if that is the case we need to follow the cursor by recursivly calling the function again, so that's what we're doing here:

{% highlight javascript %}
.then(function success(origTxsBatch) {
    
    //	CHECK FOR PAGINATION
    if(origTxsBatch.cursor != undefined) {
        
        //	IF A CURSOR WAS FOUND
        _listTransactions(locationId, beginTime, endTime, origTxsBatch.cursor)
        .then(function cursorSuccess(nextTxsBatch) {

        }).catch(function cursorError(e) {
            reject(e);
        });

    } else {
        //	IF NO CURSOR WAS FOUND
    }
{% endhighlight %}

Then, in the cursor found scenario, when we do successfully find the last list of transactions, we're going to iterate over the list of original transactions, append them to the **nextTxsBatch** we received in the final call of **_listTransactions**, then pass the list back up the chain using resolve:

{% highlight javascript %}
if(origTxsBatch.cursor != undefined) {
    
    //	IF A CURSOR WAS FOUND
    _listTransactions(locationId, beginTime, endTime, origTxsBatch.cursor)
    .then(function cursorSuccess(nextTxsBatch) {

        //	ITERATE OVER ORIGTXSBATCH
        for(let i = origTxsBatch.payments.length -1; i >= 0; i--) {
            nextTxsBatch.push(origTxsBatch.payments[i]);
        };

        //	PASS LIST BACK UP THE CHAIN
        resolve(nextTxsBatch);

    }).catch(function cursorError(e) {
        reject(e);
    });

} else {

    //	IF NO CURSOR WAS FOUND

}
{% endhighlight %}

That should cover the cursor scenario, when no cursor is found we still want to iterate over the list of transactions, add them to our **transactionlist** and resolve the transactions list

{% highlight javascript %}
} else {

    //	IF NO CURSOR WAS FOUND

    //	ITERATE OVER ORIGTXSBATCH
    for(var i = origTxsBatch.payments.length -1; i >= 0; i--) {
        transactionlist.push(origTxsBatch.payments[i]);
    }; 

    //	PASS LIST BACK UP THE CHAIN
    resolve(transactionlist);

}
{% endhighlight %}

Great, that should properly collect our transacrtions, now lets wire it up to our **getDailyTransactions** function.  Returning to the function instead of resolving a string we're going to call **_listTransactions** with resolve the results instead:

{% highlight javascript %}
function getDailyTransactions(date) {
	//	DEFINE LOCAL VARIABLES
	let locationId = '';
	let beginTime = '';
	let endTime = '';
	let cursor = undefined;
	
	//	RETURN ASYNC WORK
	return new Promise(function getDailyTrainsactionsPromise(resolve, reject) {
		
		//	RUN LIST TRANSACTIONS FUNCTION
		_listTransactions(locationId, beginTime, endTime, cursor)
		.then(function txsSuccess(allTxs) {

			//	RESOLVE RESULTS
			resolve(allTxs);

		}).catch(function txsError(e) {
			reject(e);
		});

	});
};
{% endhighlight %}

If you're astute you'll notice that none of our paramaters we're passing in have meaningful values, so before this will work we need to change that.  Your **locationId** will be specific to sqaure, you should be able to track it down through your developer dashboard. As for the **beginTime** and **endTime** lets put some strings in there for now with the understanding that we'll fill them out programatically later.

{% highlight javascript %}
function getDailyTransactions(date) {
//	DEFINE LOCAL VARIABLES
	let locationId = '[YOUR-LOCATION-ID-GOES-HERE]';
	let beginTime = '2017-08-12T00:00:00-07:00';
	let endTime = '2017-08-12T23:23:59-07:00';
	let cursor = undefined;

{% endhighlight %}

# BUILDING START AND STOP TIMES
You may have noticed that righ now we're using beginTime and endTime strings above, this defeats the purpsoe of passing the date as a paramater in the CLI.  Lets pull that string apart and generate it dynamically.

If this were a larger scale project I would probably create a whole new module for data processing and formatting, but for the sake of efficiency we're just going to create these new functions in our `squareAPI.js` script.  So lets decalare two functions in the same block that we were just dealing with:

{% highlight javascript %}
function getDailyTransactions(date) {
//	DEFINE LOCAL VARIABLES
	let locationId = '[YOUR-LOCATION-ID-GOES-HERE]';
	let beginTime = _buildTimeStamp(date, 'beginTime');
	let endTime = _buildTimeStamp(date, 'endTime');
	let cursor = undefined;

{% endhighlight %}

We'll declare a new private function called **_buildTimeStamp()** that takes two paramters, our date variables and a string declaring which part of the date stamp we're generating.  Now lets go build that function.  We'll start by adding it to our module:

{% highlight javascript %}
//	DEFINE THE MODULE
let squareAPI = {
	_buildTimeStamp: _buildTimeStamp,
	_listTransactions: _listTransactions,
	getDailyTransactions: getDailyTransactions,
    test: function() { console.log('good square test'); }
};
{% endhighlight %}

Then under the `//	DEFINE PRIVATE FUNCTIONS` section we're going to add a new function:

{% highlight javascript %}
/*
*	PRIVATE: BUILD TIME STAMP
*
*	This function wraps up the square listTransactions function so we can handle recursive calls to handle batch size
*
*	@param {string} date is a date in YYYY-MM-DD format
*	@param {string} side is a string listed as either 'beginTime' or 'endTime'
*	@return {string} a string formatted as YYYY-MM-DDTHH:MM:SS-07:00 (assuming transactions happen in the pacific time zone, otherwise change this to the proper GMT offest)
*/
function _buildTimeStamp(date, side) {
	//	DEFINE LOCAL VARIABLES
	//	RETURN VALUE
};
{% endhighlight %}

*(Note here that I'm assuming the transactions are taking place in the pacific time zone, if that is not the case for your project make sure to change the '-07:00' GMT offest to refeclt the timezone that your transactions are takeing place in)*

In our **_buildTimeStamp** funtion lets add some logic.  We'll start by defining our return value and returning it:

{% highlight javascript %}
function _buildTimeStamp(date, side) {
	//	DEFINE LOCAL VARIABLES
	let returnValue = "";

	//	RETURN VALUE
	return returnValue;
};
{% endhighlight %}

Next I'll add a gate for begin time and end time, with some error handling in case we don't get the correct value:

{% highlight javascript %}
//	IDENTIFY RETURN VALUE FORAMT
if(side == "beginTime") {

} else if(side == "endTime") {

} else {
    throw "invalid side provided"
}
{% endhighlight %}

Then we'll generate strings based on the time we're looking for:

{% highlight javascript %}
//	IDENTIFY RETURN VALUE FORAMT
if(side == "beginTime") {
    returnValue = date + 'T00:00:00-07:00';
} else if(side == "endTime") {
    returnValue = date + 'T23:23:59-07:00';
} else {
    throw "invalid side provided"
}
{% endhighlight %}
 
# WRAPPING UP DATA COLLECTION

AWESOME WORK!  That should do it for our data collection.  If we head back to our command line and type `node cli 2018-03-07` (or a date that you actually have square transaction data on) we should see the transactions data and the employee list data read out in your console log.  If you didn't please feel free to check my project on github, [Square Commissions Calculator], for any ways that it might be differing from yours.  It will be most helpful to look at the commit we're on, so lets commit that now:

    $ git add .
    $ git commit -m "Add: square-connect logic"
    $ git push origin master

Then lets move on to doing someing with this data in [Part 3: Math]

[Square Sales Commissions Posting]: /jekyll/update/2018/03/05/Square-Sales-Commissions-Calculator.html
[Part 3: Math]: /jekyll/update/2018/03/08/Square-Commissions-Part-3-Math.html
[Promises]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
[Promise.all()]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all
[Square Commissions Calculator]: https://github.com/ianemcallister/sqrComCalc