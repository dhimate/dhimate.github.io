---
title: "Google Spreadsheets and ArrayFormula for XIRR Calculation"
date: 2017-10-24
tags:
 - Google
toc: true
---
I was looking for a simple way to calculate performance of my investments over time. Popular portfolio trackers like Moneycontrol and Value Research did not meet my needs. So I decided to try out Google Spreadsheets. 
Nice thing with Google Spreadsheets is its support for import and queries. 
e.g. AMFI India publishes daily NAV’s on their website http://portal.amfiindia.com/spages/NAV1.txt
You can import the data programmatically and run a query on it to find desired NAV.
I can use following simple query to check the latest NAV of Birla Sunlife Frontline Equity fund, whose symbol is INF209K01YY7

```
=QUERY(SPLIT(QUERY(importDATA("http://portal.amfiindia.com/spages/NAV1.txt"),
"SELECT * Where Col1 like '%INF209K01YY7%'"),";"), "Select Col5")
```
Google Spreadsheets obviously supports data from Google Finance with its built in function GOOGLEFINANCE
```
=GOOGLEFINANCE("SBIN","price")
```
This will return latest price for State Bank of India. 
You can also use Stock Exchange assigned ID’s to query the data from Google Finance 

```
=GOOGLEFINANCE("500510","price")
```

This will return latest price for Larsen and Tubro. 
However the most killer feature of Google Spreadsheets is its support for ArrayFormula in conjunction with XIRR. 
Let me explain. 
Consider following cash flow. 

A	|B	|C	|D
--------|-------|-------|--------
1	|Transaction|Symbol|Date
2	|Sell|Havells|31/12/2017
3	|Buy|Havells|01/01/2017
4	|Buy|Havells|06/01/2017

We have to update the XIRR formula to include the new transaction we just added in our cash flow

```
=XIRR(D2:D4,C2:C4)*100
```
Symbol	|XIRR	
--------|-------
Havells	|4.493

This is very cumbersome to update XIRR formula, if you have a diversified portfolio with 10+ symbols and investing on a regular basis. 

A	|B	|C	|D
--------|-------|-------|--------
1	|Transaction|Symbol|Date
2	|Sell|Havells|31/12/2017
3	|Buy|Havells|01/01/2017
4	|Buy|Havells|06/01/2017
2	|Sell|SBI|31/12/2017
3	|Buy|SBI|01/01/2017
4	|Buy|SBI|06/01/2017
<br/> 

```
Havells =XIRR(D2:D4,C2:C4)*100
SBI =XIRR(D5:D7,C5:C7)*100
```
You can see for two symbols we have to keep close track of cell numbers etc to accurately calculate XIRR. 
ArrayFormula provides nice workaround to simplify and automate this process of using XIRR formulas for a growing number of transactions

> https://support.google.com/docs/answer/3093275?hl=en

> Enables the display of values returned from an array formula into multiple rows and/or columns and the use of non-array functions with arrays.

XIRR can be easily combined with ArrayFormula like below

```
=ArrayFormula(XIRR((B2:B7="HAVELLS")*D2:D7,C2:C7))*100
=ArrayFormula(XIRR((B2:B7="SBI")*D2:D7,C2:C7))*100
```

As you can see that except the symbol name both the formulas are identical. We can remove the hard coded symbol name and refer to the cell which has symbol name.
Using ArrayFormula in the portfolio tracker has simplified maintenance of the spreadsheet and saved quite a few hours for me.

