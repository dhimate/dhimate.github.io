---
title: "MuleSoft Capacity Explanation for Noobs and CXOs "
date: 2020-12-21
layout: post
tags:
 - MuleSoft
categories:
 - Programming
toc: true
---
I was talking to a C-level executive the other day and he asked me to explain how does MuleSoft's capacity/sizing across various deployment models work. Given I always wear my technical hat I jumped right into the concept of cores and virtual cores and containers (and Kubernetes!) and did a pretty bad job. Fortunately, that executive was _very_ accomodating and _somewhat_ understood what I was talking about. Nevertheless, I should've done better.

I was thinking about this conversation over the next couple of days and decided to explain it to my wife - who is very non-technical.

Think of MuleSoft as a specialty cookie store. Or even better, think of Salesforce as a big giant grocery store, with a bakery section called 'MuleSoft'.
MuleSoft specializes in cookies. They offer 3 types of options to buy their fantastic product. 

- **Cloudhub:** Pre-baked cookies, these come in boxes. Very light and crispy. Rightfully branded as 'Cloudhub'. You can buy as many boxes as you want. Each box contains 10 cookies. This means with one box you can feed 10 guests. If you have more guests coming over, or some guests are unusually hungry you can buy more boxes. 
There is also an allergen/gluten free version of pre-baked cookies for government guests. 
- **Hybrid:** A cookie dough to bake your cookies in your oven. The cookie dough is branded as 'On-premise / Hybrid'. You can decide how big (or small) each cookie can be. You can make as many cookies as you want with the dough. (No more 10 cookies per box restriction!). Of course, some guests will find your ridiculously small sized cookies unappetizing (or funny).
- **Runtime Fabric:** After some guests complained that pre-baked cookies were too big, and 'make your own cookies' unappetizing. MuleSoft started offering 'Runtime Fabric' branded cookie dough. You still have to bake your cookies, but the sizes of cookies are standardized. They won't look funny.


 

