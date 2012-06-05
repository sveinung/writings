Why should the business guys care about Continuous Delivery?
============================================================

Continuous Delivery (CD) does not neccessarily mean that we push new features
into production often, it means that we are able to push new features
into production as soon as they are ready. If you push out new features
every day, but those features were "finished" two months ago, you're not
doing CD.

This also applies to the other stages of an ideas lifecycle: we want to
decrease the time an idea spends as work-in-progress. In the lean world 
this is known as _cycle time_. Having a low cycle time gives us many of the
benefits explained below.

Lower deltas means lower risk
-----------------------------

If we deploy each feature to production individually, there are only so
many things that can go wrong at a time, and it is much easier to pinpoint
the code that caused the failure than if we deploy a years worth of work at
once. This applies both at the code-level and the feature-level.

At the code-level this is quite obvious -- it's why we do TDD. Debugging
one new line of code is easier than debugging a hundred.

The same goes for features deployed into production -- releasing ten
features separately makes it easier to pinpoint those the customers respond
well to than releasing all at once.

Working with small batches also enables shorter feedback loops.

Shorter feedback loops
----------------------

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
------------------------------------------

With CD your you get more responsive: Toward your customers, and
toward your competitors. You should be capable of deploying a given
feature to production when the feature is ready with as little overhead
as possible -- not as little part of a huge batch half a year later. If
the time between a customer request and the delivery of the resulting
feature is short, the customer gets the feeling that he/she is listened
to (and is respected).
