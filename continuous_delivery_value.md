Why should the business guys care about Continuous Delivery?
============================================================

At its core, Continuous Delivery is about delivering features to
customers as soon as they are ready. Doing this has several benefits
for both business and development. First and foremost it lets us adapt
continuously and rapidly to customer feedback and changes in the market,
which enables us gain insights from real usage. As a result, we have a
better basis for knowing if we are building the right product.

A lot have been said and written about the technical aspects of
continuous delivery -- why it reduces overhead for development by
automating everything that can be automated. This blog post will
therefore focus on three code _business_ aspects of delivering
continuously.

Smaller deltas of change means lower risk
-----------------------------------------

With relation to the amount of changes done between one deployment to
production and the next we sometimes talk of the _delta of change_.
Reducing the delta of change means that less can go wrong at a given
deploy. Why? Both from a technical and business perspective it is far
easier to pinpoint the problem when there are less changes in the product -
when doing fewer changes at a time we can be more certain that we are
actually delivering the _right thing_.

When releasing fewer changes it is also easier for business to
understand the impact of each additional change. This enable us to learn
more about customers and not waste money on building features that
does not add value.

Also, fewer changes means that we can deploy more often. And deploying
more often means shorter feedback loops. And shorter feedback loops
means we will learn more about customers.

Shorter feedback loops
----------------------

("Being able to make the correct decisions faster is one of the main
advantages of shorter feedback loops. Basically, faster feedback loops
enable us to learn more about our customers faster.")

("The developers of a product often have several layers between them and
the customer. Shorter feedback loops means fewer and more rapid code
changes, which will increase the developers understanding of what the
customer wants. This improvement can lead to improved decisions and
technical innovation that are more aligned with what the user wants.
The more you know your customers, the easier it is to build the right
thing.")

We already know the benefits of a shortened feedback loop when it comes
to testing: Doing testing continuously (as a part of development) instead
afterwords enables us to catch programmatic errors earlier, while they
are still cheap to correct.

The same thing goes for releasing - we can not be sure that a given
feature gives the customer value unless it has been released. Therefore,
for all we know, we might be creating the wrong thing. We want to _fail
fast_.

Being able to sooner make the correct decisions are one of the main
advantages of shorter feedback loops, but to be able to decide anything
we need data.

Increased responsiveness
------------------------

("By continuously delivering small changes rapidly you can be more
reponsive, both with regards to customers and with regards to the market
and your competitors. Increased responsiveness means that you are able
to faster adapt to customer needs and market changes. For example, the
faster you are able to deliver on a customer request, the happier and
more satisfied they will be.")

("Why keep a feature that is already developed and ready for production
just lay around waiting to be deployed in a year? Or for that matter, in
a month or a week?")

With Continuous Delivery you get more responsive: Toward your customers, and
toward your competitors. You should be capable of deploying a given
feature to production when the feature is ready with as little overhead
as possible -- not as little part of a huge batch half a year later. If
the time between a customer request and the delivery of the resulting
feature is short, the customer gets the feeling that he/she is listened
to (and is respected).
