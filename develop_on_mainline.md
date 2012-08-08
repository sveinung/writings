How to develop on mainline
==========================

As we are working on software projects we often need to develop
multiple features concurrently, and if we are delivering these features
continuously they should be made available for our users as soon as they
have matured. Unfortunately for us, features are seldom finished at the
same time. Therefore, we need mechanisms for controlling when a given
feature is made available.

This blogpost will focus on techniques such as: feature branching, feature
toggles, and branch by abstraction. Especially the last two, which also
enable us to do development on _mainline_ (or trunk or master as some
people also call it).

Using feature branches
----------------------

The first technique for controlling the exposure of features is called
_feature branching_. Simply explained, this consists of creating a new
development branch every time you begin working on a new feature. After
the feature has reached sufficient maturity, it is merged back into the
main development branch.

Feature branching works best when each branch is relatively short-lived
and each feature does'nt overlap too much with other features during its
lifespan. Overlap leads to increased overhead when during merging as the
person doing the job has to decide which changes to keep.

One downside of feature branching is that they make it hard to share
code across branches. As we do not—and should not—plan everything ahead
of time, overlapping functionality between features are bound to happen.

Using feature toggles
---------------------


Branch by abstraction
---------------------


