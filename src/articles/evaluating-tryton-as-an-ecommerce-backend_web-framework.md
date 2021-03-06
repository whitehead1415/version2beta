---
pagetitle: Tryton as an e-commerce back-end - Web Framework
longtitle: The crux of the matter - ERP needs a web framework so we don't do things twice.
tags: cto
author: Rob Martin
published: 2011-11-19 16:44
snippet: This is part three in a series exploring how Tryton might fare as the heavy-lifting portion of an e-commerce package. This third part explores what it means to do e-commerce.
---

This is part three in a series exploring how Tryton might fare as the heavy-lifting portion of an e-commerce package. This third part explores what it means to do e-commerce.

The whole series:

-   [Part 1: About Tryton][part1]
-   [Part 2: Stalking the Tryton community][part2]
-   Part 3: The crux of the matter

[part1]: /articles/evaluating-tryton-as-an-ecommerce-backend_about-tryton "Evaluation Tryton as an e-commerce back-end: Stalking the Tryton community"
[part2]: /articles/evaluating-tryton-as-an-ecommerce-backend_stalking-the-tryton-community "Evaluation Tryton as an e-commerce back-end: Stalking the Tryton community"

## First a review

If you read my last two posts on this topic, you'll recall I'm in the process of discovering the whys and hows of using Tryton as the back end of an e-commerce system.

To review, Tryton[^tryton] is an open source ERP[^erp] platform licensed under the GNU General Public License,[^gpl] written in Python[^python] and working with a PostgreSQL[^postgres] database server. Phew. In short, it's a cool way to set up One Software To Rule Them All.[^lotr]

In this post we get into the meat and potatoes: what is e-commerce anyway?

[^tryton]: [http://tryton.org/][Tryton] Tryton project home

[Tryton]: http://tryton.org/ "Tryton home page"

[^erp]: Don't know what [Enterprise Resource Planning][ERP] is? Wikipedia does.

[ERP]: http://en.wikipedia.org/wiki/Enterprise_resource_planning "Wikipedia article on enterprise resource planning."

[^gpl]: [The Gnu General Public License version 3.0][GPL-3]. Tryton is licensed under these terms.

[GPL-3]: http://www.gnu.org/licenses/gpl-3.0.html "The Gnu General Public License version 3.0"

[^python]: The [Python programming language official website][Python]. Guido van Rossum started development on Python in 1986, and named it in part after Monty Python's Flying Circus.

[Python]: http://www.python.org/ "The Python programming language official website. Hello, Guido!"

[^postgres]: Have I mentioned how much I like [PostgreSQL][] before? I really did discount and ignore it before I had to start using it on a project, and that's all it took. I prefer PostgreSQL now over every other SQL database I've used.

[PostgreSQL]: http://www.postgresql.org/ "Have I mentioned how much I like PostgreSQL before?"

[^lotr]: No ring required.

## What's e-commerce anyway?

There are many ways to define e-commerce. Perhaps most comprehensive would be a list of features that are implemented in a given e-commerce system. I considered creating such a list for this blog post. Ultimately, I decided to keep it simple, because the most simple view also exposes the biggest holes in almost all e-commerce systems.

Here's the simple definition: **e-commerce is the implementation of a business' customer interface on the internet.** If it involves a customer, a company, and a computer, it's probably e-commerce.

Obviously this covers all the typical interactions: Buying a shirt, choosing an XL size. Reviewing my order before I check out. Paying for my order, getting my order confirmation. Getting a note when my shirt ships. Returning it when I discover the hole in the underarm.  As you can see, this is all part of a process.

A lot of this typical process is well defined by best practices, and implemented in (almost) every sort of e-commerce platform. So why do we need to consider Tryton?

The problem is that my process, or your process, might actually be different. In fact, I can say my process is definitely different - I'm not selling shirts, or widgets, or anything concrete. I sell services as diverse as disinfecting a compromised web server to building an e-commerce web site. Many of my sales are projects, made up of milestones and tasks and billing stages. Sure, when I order a shirt online I might wish to see when it's going to ship, but when my customer orders a web site, she might wish to see where we're at in the process of producing it.

My work is not unique in its e-commerce needs. Heck, my field isn't even unique. Just about everyone who sells anything other than widgets online needs something that's either custom, or developed special for their vertical market. Lawyers, doctors, veternarians, carpet cleaners, housekeepers, daycare providers, doggie daycare providers, landscapers, painters, roofers, artists, appliance repair persons, gutter cleaners, chimney sweeps, HVAC cleaners, pest control applicators, plumbers, electricians, home theater installers, home automation designers, car mechanics, small engine mechanics, massage therapists, optomotrists, window cleaners, holiday window painters, office plant carepeople, office fish carepeople, snow removers, community theater, professional theater, cinema, parking space rental, storage space rental, apartment rental, house rental, hoteliers and moteliers, concierge and personal assistants - the list goes on.

Why can't I pay my rent online? Schedule my car service? Schedule a gutter cleaning, a chimney cleaning, and a tooth cleaning - all for the same day? After all, I can practically watch Domino's make my pizza with their new system.


## E-commerce is the ERP interface

ERP is a system that manages documents as they progress through a pre-defined process. The documents very often represent something else &mdash; a purchase order represents some product that needs to be received and paid for, a sales order represents a set of products that need to be shipped and paid for, a work order represents some raw product that will be transformed into something new. We call the process they go through "workflow".

The more accurately we model the real world with our ERP analogs, the more accurate the information we retrieve from the ERP will be. If we count incoming deliveries before marking an order received, we'll probably have better inventory numbers and fewer overcharges from vendors. If we eschew back door sales, chances are good our sales figures will be closer to reality and more profitable too.

Normally speaking, our customer doesn't have much more access to our ERP system than glancing at my screen as I fill out their order. That is, until we connect a website to it, and then much of the interaction they have is - or should be, as I propose - interfaced to the ERP.

Let's start at the beginning with a brand new customer, and fictitious business called Hearty Dog Treats that sells natural treats for dogs. Jane Doe goes shoppping for some new treats for her Newfoundland (his name is Jayne, but he's named for Jayne Cobb from Firefly and Serenity, not because it sounds just like Jane's name) and she finds Hearty Dog Treats through a series of fortuitous events. Here is her process, and how it could and should represent an interface tightly integrated with ERP:

1. Jane visits the home page of the website to see who the company is. A sidebar advertisement is promoting 12" bully sticks.[^bully] This advertisement might be promoted based on an overstock, or special pricing received, or other condition modeled within the ERP.
2. Jane clicks through to the product page for bully sticks. She reads the ingredients (wow - she didn't realize what they were actually made of!) She looks at the price and stock status. The three primary pieces of information Jane pursued all come from the ERP system. A food's ingredient list really must be kept up to date and managed in a system that allows for appropriate controls and distribution. Something like an ERP system would work. The stock status is certainly derived from the ERP's inventory of bully sticks. And Jane's price comes from the default web pricelist. Too bad she's not a repeat customer or a dealer - she could create an account on the website, and get access to better pricing. *Her* pricing.
3. Jane is actually looking for duck hearts. She returns to the "Treats" section of the catalog. The catalog as a whole, and the navigation for it, may very well come directly from the ERP system. Indeed, if Hearty Dog Treats maintains multiple sales channels (say, online sales, a print catalog, and flyers to hand out at dog shows) they might wish to maintain the official copy for marketing materials in the ERP system as well, associated directly with the product or product line. Besides, the high school kids with the after school jobs running a point of sale terminal at Hearty Dog Treats' retail location doesn't know much about the product, so putting good product information into the ERP system will help them make the sale too.
4. Jane clicks through to Freeze Dried Duck Hearts. (These are real, by the way.[^fddh] I often claim that either of my Basset hounds would balance a piano on their nose for a freeze-dried duck heart.) See step 2 above.
5. Jane decides to buy. She fills out her order and submits it. She's just created a new document in the ERP system and started the process of filling an order.
6. Jane gets a confirmation email from the ERP system. She clicks the link and sees that her order has been picked from inventory and moved to shipping. She thinks to herself how cool it is to be able to see the progress of her order.
7. When Jane's order is ready to ship, the ERP system captures her credit card funds. Sure, it used to be that e-commerce systems grabbed the money when the order came in, regardless of whether it could ever be filled, but today's system knows it's better all around to charge for what you're shipping, not what you hope to ship as soon as possible.
8. Jane receives an email that her order has shipped, containing a tracking number so that she can check the progress of her delivery. She does. The truck is out for delivery.
9. The day after delivery, she gets an email from Hearty Dog Treats[^hearty] saying she should have received her order, and asking if everything okay with it. It also suggests she might want to subscribe to their email newsletter, or follow them on Twitter, or fan them on Facebook. In fact, maybe Jane would like to click the link below and share Hearty Dog Treats with her friends on Twitter and Facebook.

Every step of Jane's order interfaced with the ERP system. Many, many websites work this way, but far from all of them. Most tier 3 and 4 businesses aren't running ERP at all. They're getting by on Quickbooks, Peachtree, or sometimes even the shoebox accounting method. Many e-commerce systems don't connect to any ERP system, or even to Quickbooks. Since that is the case, some companies actually use their e-commerce *as an ERP package* even though the design is weak at best.

That brings us to the next point: ERP is the company's interface to the e-commerce system.

[^bully]: Bullies are great dog treats, but their made from bull "pizzle" which is really just an old English word for penis. We buy [these bully sticks][bully].

[bully]: http://www.freshisbestinc.com/products/pet-treats-chews/bully-sticks-12-inch "Fresh Is Best 12" bully sticks."

[^fddh]: Freeze-dried duck hearts are also quite real, and I have never used a more effective treat with our girls. Check out [Fresh Is Best's freeze-dried duck hearts][fddh].

[fddh]: http://www.freshisbestinc.com/products/pet-treats-chews/freeze-dried-duck-heart-bites "Fresh is Best freeze dried duck hearts"

[^hearty]: Hearty Dog Treats is a business I want to build, but as yet it's just a dream waiting for a preponderance of time combined with a sufficiency of money. [Fresh is Best Inc.][fib] is not fictitious. They are one of our customers, but not currently using an ERP system.

[fib]: http://www.freshisbestinc.com/ "Fresh is Best Inc home page"


## ERP is the e-commerce interface

It stands to reason that, if ERP is the flip side of the e-commerce coin, the e-commerce is the flip side of the ERP coin too, right? (If not, that would make a cool magic trick...)

For each step Jane when through, there were one or more steps that Hearty Dog Treats went through as well. They need to order the product and have some in stock. They need to process her order and issue a pull slip or pick ticket or delivery order or whatever Hearty Dog Treats is using to represent a stock move from inventory to shipping to the customer. They need to order more stock, if their inventory has fallen below the trigger point. They need to reconcile her payment on the back account. All of these tasks are triggered by, and performed in their ERP system.

In fact, the reason Hearty Dog Treats has ERP is so that these processes can be automated as much as possible. Hearty Dog Treats wants things to run smoothly and profitably, even when the boss is away.


## Wow, so Tryton can do all this?

Nope.

Tryton can't do it. OpenERP can't do it. Magento can't do it. Quickbooks, Peachtree, Sage &mdash; I don't really think any ERP or accounting system, open or closed source, *designed for small business,* can do what I'm describing. (If you think I'm wrong about this, comment or tweet at me right away. I don't want to redesign the wheel. I do want a system that does all of this, scales down to a mom-and-pop shop and up to tens of thousands of products and customers.)

I can't let that sit. I feel like those statements need to be expanded upon. I hear someone saying, "There's someone wrong on the Internet! He uses the handle Version2beta!"

First, Tryton can do it, because Tryton is a framework for developing business applications. Given a framework and a developer, much can be done. It's my opinion after months of planning this project that Tryton can do it well. Nonetheless, Tryton cannot do it yet.

According to OpenERP SA, OpenERP will integrate with a number of popular open source e-commerce platforms, including OSCommerce, Magento, and Joomla.[^oerp1] And they do - I know some of the people who have worked on the Magento Connector.[^oerp2] In the very next paragraph of the same page, OpenERP says that "to benefit from all the power of OpenERP ... you can also use our own eCommerce system that automatically plugs in an existing website. Add a few lines of HTML in your website and OpenERP converts it to a full featured eCommerce platform."[^oerp3] The rest of the page lists cool things you can do, a broken image link, and some comments from visitors asking how. If you dig deeper into their site, you can find the e-commerce module, version 5.0.1.0 (which often indicates it is a module for OpenERP version 5, even though it's listed under OpenERP version 6). This module is not an official module, not quality certified, and reimplements Partner, Address, Payment, Shop, Category, Sale Order, and Order Line &mdash; all standard OpenERP objects &mdash; in order to achieve e-commerce. And so I hold that OpenERP cannot do this.

Magento can't do this.[^magento] Now that they've been bought by eBay, Magento is doing a lot of cool things, and it would not surprise me if they moved in this direction down the road. (I'm certain many companies will.) They're even providing a US$1 million stimulus to encourage small business e-commerce.

Quickbooks might be able to do it, if you go for the Enterprise version and add on third party software. It's my impression (earned in part by having used Quickbooks) that it's a great accounting package for small businesses, but the software's enterprise features are flawed with usability issues. Then there's the third party problem for the e-commerce software - how integrated is it with Quickbooks? Both Quickbooks and the e-commerce package are proprietary; how well are they kept in sync? Is the quality of the code there? I'm not trying to engage in FUD[^fud] &mdash; I'd be thrilled to hear about a solid Quickbooks e-commerce solution that goes beyond widgets. Let me know and I will list it right here.

Peachtree can't do it &mdash; or if it can, we're looking at the same situation as Quickbooks at best. But Peachtree's parent company, Sage, can do it. Once again, though, you need to use their proprietary module (eBusiness Web Services) combined with a third party e-commerce application.[^sage] Besides, Sage is pretty pricey for a small business. We had a customer gut OpenERP (and us, along with it) and put in Sage. Rumor has it the licenses alone were more than US$80,000, plus integration, plus e-commerce software and hosting.

Netsuite can probably do it. With Netsuite Ecommerce, a lot of set-up, and US$1,200 a month, a mom-and-pop shop can sell widgets online and get the benefit of Netsuite's accounting, finance, and supply chain management. At that price, it is probably affordable for a business doing a million a year in sales. (Mom and pop are good at this widget business.)

You know who can do it, with open source software even? Apache OFBiz.[^ofbiz] Honestly I know very little about OFBiz (which stands for "Open For Business"), but it clearly does a lot, and if you're looking for a community open source ERP project that does web e-commerce right now, this might be your choice. If you use it, let me know what you think?

[^oerp1]: [OpenERP SA's "feature" page on integrated e-commerce][oerp1].

[oerp1]: http://doc.openerp.com/v6.0/features/integrated_ecommerce.html "Feature page"

[^oerp2]: [OpenERP SA's e-commerce doc page from the OpenERP v6 technical guide][oerp2]. Although it is labeled version 6, this page seems to discuss a version 5 module.

[oerp2]: http://doc.openerp.com/v6.0/technical_guide/ecommerce.html "Technical guide"

[^oerp3]: OpenERP is growing and improving, or at least that's what I hear. Version 6.1 might have what version 6.0 lacked. Check it out for yourself. [Here's the page I visited][oerp3].

[oerp3]: http://doc.openerp.com/v6.0/features/integrated_ecommerce.html "OpenERP's integrated e-commerce capabilities?"

[^magento]: See [Magento's feature list][magento] for more information. Really though, I like the quote from [Sharoon Thomas discussing a Magento connector for Tryton][connector]: "An integration with Magento does not add 'The best e-commerce possibility for Tryton' but does make 'A great ERP backend for Magento'."

[magento]: http://go.magento.com/features/ "Magento's list of features"

[connector]: http://groups.google.com/group/tryton-dev/msg/403620501bffdb55 "Tryton-dev discussion regarding a Magento connector for Tryton"

[^stimulus]: Magento Go's [stimulus program][stimulus] comes in the form of a $15/mo credit for 12 months, if you qualify and apply. It's marketing, but it's kinda cool marketing. And, if you're looking for that kind of e-commerce service and hosting, the $15 credit means you can get 12 months free at the lowest cost plan, but read the fine print because you still have to buy other services with them.

[stimulus]: http://go.magento.com/1-million/ "Magento Go US$1 million stimulus program"

[^fud]: [Fear, uncertainty, and doubt][fud].

[fud]: http://en.wikipedia.org/wiki/Fear,_uncertainty_and_doubt "Wikipedia article on the sales and marketing tactic known as FUD."

[^sage]: I was surprised by this. I thought they had their own e-commerce module, but the closest I find for Sage MAS 90 and 200 is the [eBusiness Web Services module][sage].

[sage]: http://www.sagemas.com/Products/Sage-ERP-MAS-90-and-200/eBusiness/eBusiness-Web-Services "Sage eBusiness Web Services"

[^ofbiz]: I don't know what to make of this project, honestly. [Open For Business][ofbiz] has been around for ages, they're up to version 10, and they've got some high profile users, like British Telecom. It doesn't seem to be limited in any way to widgets. But, it's written in Java and I'm a programming snob, and it feels heavy to me. Really heavy. I'd really love to hear from someone who's worked with OFBiz, or even worked on the project. Tell me about it?

[ofbiz]: http://ofbiz.apache.org/ "Apache Open For Business project"


## What does Tryton need?

I think Tryton needs a web framework.

I'm somewhat at odds with Cédric Krier[^cedrick] on this, I know. We've chatted on IRC, and he assures me that Tryton doesn't need a full framework; JSON[^jsonrpc] is enough. And he's right - JSON is enough. Tryton itself is a framework, and with an easily implemented interface like JSON, one can do all I describe above. But unless there's something on the other end of the JSON, it's useless to my customers, and in turn useless to my customers' customers.

Cédric's "something on the other end of the JSON" approach puts Tryton as a rather smart back end to a web application (any application, really - I can totally see using it behind an Android or iPhone app), without building an actual framework into Tryton. But I'm not sure that a real web framework built into Tryton wouldn't be very, very useful. I have ideas on how to do this, involving some of my favorite tech tools - MongoDB[^mongo], Jinja2[^jinja2] or Restructured Text[^rst], maybe a little Flask[^flask] action. Some Redis[^redis] and Celery[^celery] to make sure we never ask Tryton to try too hard.

I'm certain this can be done. I'm pretty sure I can do it. I strongly suspect it's already been done by at least one company, for private use.

[^jsonrpc]: [JSON-RPC][] is like XML-RPC, but quieter.
[JSON-RPC]: http://en.wikipedia.org/wiki/JSON-RPC "JSON-RPC Wikipedia article."
[^cedrick]: [Cédric Krier][cedrick] is with [b2ck.com][] and is something of the [BFDL][] for Tryton. Every time I write that, I am uncomfortable. I've only seen one person definitively call him Benevolent Dictator for Life, but he so clearly fits the role, even without better confirmation I have to stop saying "...something of the BFDL...". He is. Complain at me if you think I'm wrong.
[cedrick]: http://twitter.com/#!/cedrickrier "Cédric Krier on Twitter."
[b2ck.com]: http://b2ck.com "B2CK Software Development."
[BFDL]: http://en.wikipedia.org/wiki/Benevolent_Dictator_For_Life "Wikipedia's article about open source software development leadership and the title 'Benevolent Dictator For Life'."
[^mongo]: [MongoDB][mongo] comes from "humongous" - a no-SQL, document-oriented database capable of some very cool feats.
[mongo]: http://www.mongodb.org/ "MongoDB, a document-oriented, open source, high performance datbase."
[^jinja2]: [Jinja2][] is a designer-friendly templating language, somewhat based on Django but extending Django templates in some important and well-considered ways. It's also [Armin Ronacher][mitsuhiko] project, and I seem to be a fanboi.
[jinja2]: http://jinja.pocoo.org/ "Jinja2 template engine."
[^rst]: Restructured text, more properly written reStructuredText, is another markup language. It happens to be the markup used by the [Sphinx][] documentation generator, so I use it all the time.
[rst]: http://docutils.sourceforge.net/rst.html "Markup language, part of Docutils."
[^sphinx]: Sphinx is a brilliant documentation generator. Apparently it works with source code quite nicely, but so far I've been taking it straight.
[sphinx]: http://sphinx.pocoo.org/ "Easy and beautiful documentation."
[^flask]: [Flask][] is a Python microframework, a set of tools that can connect a web interface to an object back-end with appropriate routing and logic. Another [Armin Ronacher][mitsuhiko] project.
[flask]: http://flask.pocoo.org/ "Flask micoframework in Python."
[mitsuhiko]: https://twitter.com/#!/mitsuhiko "Armin Ronacher on Twitter."
[^redis]: Redis is a key-value store, much like memcached, the entire DNS system, etc. It's a tremendously efficient way to store smaller bits of data, like information you might cache a while.
[redis]: http://redis.io/ "Key-value store."
[^celery]: Celery provides a distributed task queue, kind of a to-do list for other programs.
[celery]: http://celeryproject.org/ "A distributed taask queue."

## A Tryton web framework

When I first started this blog post almost two months ago, my work life was considerably different than it is right now. At that time, I thought I may be days from starting the web framework project for at least one, and maybe two different customers.

Hell, when I first started this series of blog posts on Tryton as an e-commerce back-end, I thought I'd have a Tryton-based e-commerce platform done by now. Funny, that. But then life knocked on my door - kicked it in, more like. Among other things, I worked with one of the best Tryton programmers in the world for nearly 40 hours straight, and at the end of it, I had lost my biggest customer. (Correlation, not causation - he didn't steal my customer or anything like that.)

I still have prospects for using Tryton as an e-commerce back end. If any of them come through I may continue the series. Until then I hope it's been a useful discussion.

### What else is there?

-   [Part 1: About Tryton][part1]. An overview of Tryton – architecture,
    business model, features, license.
-   [Part 2: Stalking the Tryton community][part2]. There are some
    impressive people working on this project. I’m pleased to get to
    know them.
-   Part 3: The crux of the matter. What
    does a Tryton-backed e-commerce system need to do?

*If I’ve inspired a response from you (probably the type of response that deserves an apology), mention [@version2beta][] on [Twitter][] and I’ll see it there. Or, if you can’t comment in less than 126 characters (my handle takes 14 characters with a space), blog about it and tweet that.*

[@version2beta]: http://twitter.com/version2beta "Me, twitterfied"
[Twitter]: http://twitter.com/ "All of twitter, including me."
