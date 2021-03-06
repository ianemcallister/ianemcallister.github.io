---
layout: post
title:  "Square Sales Commissions Calculator"
date:   2018-03-05 21:04:19 +0900
categories: jekyll update
---
![Square_Logo](/assets/squareLogo.png)

# BACKGROUND
[Square] is a game changer for small businesses.  If you've visited a farmers market, local street fair, or perhaps your local coffee shop recently, you've probably come across the Square point-of-sale system.  They have made credit card processing much, MUCH easier for smaller vendors. Now there are a plethora of merchant service companies compeating in the space, but what makes Square great is that they're a technology company first.  They have a fantastic API and well documented [developer] tools that make extending the functionality of their services increadibly easy for developers.

I started playing around with their api to get a better handle on our sales data.  What I discovered is that I could automate some of the process I was spending time on each week without a great deal of effort.

# PROBLEM TO SOLVE
Square offers no out-of-the-box solution for calculating the commisions and gross pay of employees login into thier system.

# QUALIFYING THE PROBLEM
Square is great at collecting credit card funds and tracking sales data.  In my business I pay people commission to encourage sales engagment.  Unfortunatley, out-of-the-box, Square does not have a solution for calculating commisions.  For this reason I have built tools to take some of these calculations off my plate.  This series of posts explores my solution and how I have implamented it in my business.

# SOLUTION
To solve this problem we'll build a command line interface tool that we can run with a date value.  It will report all the guaranteed wages, commissions, and tips generated by each employee on that given day.  Extra points if we email that list to ourselves, or even better the employees.

# PROCESS STEPS
1. Accept a date to report on
2. Download transactions data for that date
3. Download employee information (id, email, guaranteed rate & commission rate)
4. Filter & sum transaction data by employee
5. Calculate commission earnings
6. Calculate difference of commissions and guaranteed rate
7. Notify the results

# SEQUENCE
I've broken these steps down into the folloiwng postings:
* Project Setup is covered in this posting
* [Part 1: Defining Modules & Dependencies] - Requirnments
* [Part 2: Collecting Data] - Covers step 2, 3 & 4
* [Part 3: Math] - Covers steps 5 & 6
* [Part 4: Notify Results] - Covers step 7

# REQUIRNMENTS
In order to follow along with this sequence you'll need the following:
1. [Node.js]
2. your own square developer [credentials], which can be obtained here
3. your own database, I'm a big fan of [firebase], an account can be created here

# PROJECT SETUP
Lets start in your favorite sandbox directory and create a new directory for our project, I'll call mine **sqrComCalc**

`$ mkdir sqrComCalc`

lets navigate into the directory

`$ cd sqrComCalc`

lets create a readme file just to have something in that folder

`$ echo "# sqrComCalc" >> README.md`

then lets initialize our git repository

`$ git init`

lets add the contents of this directory to our first commit

`$ git add .`

then we'll identify this commit 

`$ git commit -m "first commit"`

make sure we link it to our remote repository on github (you'll have to set this up on your own, if you need help check [here])

`$ git remote add origin git@github.com:[YOUR-GITHUB-USERNAME]/[WHATEVER-YOU-ALL-THE-PROJECT-I-SAID-SQRCOMCALC].git`

finally lets push our local repository

`$ git push -u origin master`

now we're ready to jam. If you want to reference my repository at anypoint it can be found here on github [ianemcallister/sqrComCalc]

# BUILD CLI
*(Before you start moving along this step make sure you have NodeJs installed)*

Now that we have the place to put our files lets build our command line file.

In your favorite editor (I was a [sublime] fan for a long time, but recently have come to love [Visual Studio Code] so that's what I'll be using), open our project directory and lets create a new file called

`cli.js`

lets open the file, then it doesn't hurt to add a little documatation at the top to help others (or your future self) remember what this file is about later, so I'm just going to comment out the top few lines and title the file

```
/*
*	CLI
*
*	This is the command line interface script
*/
```

Next lets make some space for the dependent files we'll need later, the local variables we'll be defining, and any notifycations that we want to pass along to the terminal

    //  DECLARE DEPENDENCIES

    //  DEFINE LOCAL VARIABLES

    //  NOTIFY PROGRESS

Because this program relies on a date received from the command line when the script is launched lets start by handling that argument.  Node has a built-in tool to handle these arguments (```process.argv```).  Lets make sure it's working by reading the parameter to the screen at launch.  So under our local variables section lets add:

    let dateparam = process.argv[2]


then under the notify section we'll add

    console.log(dateparam)

```process.argv``` returns an array of arguments, the first two don't concern us right now, so we're looking at the 3rd element in the array (but because it's zero-indexed we're looking at element 2 here).  If you run this script from the command line now with some sort of value as a paramter you should see that value reported back to you in the console.

`$ node cli 2018-03-05`

This should report back `2018-03-05` in the command line. If so, **congratulations!** 

At this point your `cli.js` file should look something like this:
{% highlight javascript %}
/*
*	CLI
*
*	This is the command line interface script
*/

//  DECLARE DEPENDENCIES

//   DEFINE LOCAL VARIABLES
let dateparam = process.argv[2]

//   NOTIFY PROGRESS
console.log(dateparam);
{% endhighlight %}

If it does, lets push those changes to our github:

    $ git add .
    $ git commit -m "Add: CLI"
    $ git push origin master

and move on to [Part 1: Defining Modules & Dependencies]

[Square]: https://squareup.com/us/en
[developer]: https://squareup.com/us/en/developers
[Part 1: Defining Modules & Dependencies]: /jekyll/update/2018/03/06/Square-Commissions-Part-1-Defining-Modules-and-Dependencies.html
[Part 2: Collecting Data]: /jekyll/update/2018/03/07/Square-Commissions-Part-2-Collecting-Data.html
[Part 3: Math]: /jekyll/update/2018/03/08/Square-Commissions-Part-3-Math.html
[Part 4: Notify Results]: /jekyll/update/2018/03/09/Square-Commissions-Part-4-Notify-results.html
[Node.js]: https://nodejs.org/en/
[credentials]: https://squareup.com/us/en/developers
[firebase]: https://firebase.google.com
[here]: https://github.com/new
[sublime]: https://www.sublimetext.com
[Visual Studio Code]: https://code.visualstudio.com
[ianemcallister/sqrComCalc]: https://github.com/ianemcallister/sqrComCalc