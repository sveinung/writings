Autonomous teams – Organizing for Speed and Agility
===================================================

Lead time is the time it takes you to complete a process, for instance from conceiving an idea to actually having something implemented in production. Shortening this lead time makes everything better. If you think it doesn't apply to you you're probably ripe for being naturally selected in really really bad way.

For innovation to work optimally learning has to be as time and cost efficient as possible. Increase lead time and you'll reduce your learning rate. Increase deployment costs and you'll reduce the number of hypotheses you can run for the same amount of money, thereby reducing the number of features you can try out before running out of money or having your steering committee lose their patience and kicking your butt out on the street.

This blogposts highlights some common organizational pitfalls that negatively impacts your ability to deliver value to your customers.

An organization can either optimize a value stream for _utilization_ or _throughput_. Not both. Since throughput makes more sense if you'd like to make it easier to be innovative I'm going to assume throughput is your cup of tea.


Getting rid of feature handovers
--------------------------------

Anyone who has ever worked on anything with other people have experienced that the one thing impacting lead time most else is handovers. Some handovers we can't easily be without if we need people to specialize as developers, designers, and so on. Some handovers, though, we can avoid by designing our organization the right way.

Different teams have different goals and backlogs – it's why we split our organizations in multiple teams in the first place, to be able to work concurrently with more than one thing. And because different teams have different goals and backlogs they'll also have different priorities. The conclusion is that if we design our organization so that every feature requires work done on more than one team we're going to have a bad time! That includes "thinking work" by business people and other people who like to "analyze".

<un-agile-org.png>

As long as you divide your people into teams you're going to have silos and suboptimization. Just by existing, teams act as barriers to communication and cooperation (this is due to them having different goals than other teams, as we stated above). What you need to do is to _organize your teams in a way that causes suboptimization to be as optimal as possible_. If you're optimizing for speed and agility, you want to reduce the number of handovers needed to deliver customer value and to test hypotheses. If that means increasing the number of handovers needed to implement _shared services_, so be it.

What you should do is to organize your teams around business outcomes and not competencies or even applications. If you organize around competencies or applications you're most likely going to have a significant amount of handovers per feature you want implemented. If you organize around business outcomes–one outcome, one team–you'll avoid feature handovers. You might still have to endure handovers due to _shared services_ (e.g. a common style guide or common SSO solution), but such work doesn't have to be a part of every feature you implement, like frontend or database work probably will.


Platform handovers
------------------

The classical example of a handover is the one to operations. The best way to do away with this type of handovers is to provide your operations platform self-serviced.

- can be standardized
- the platform should be self-serviced
- on-premise cloud _will_ impact autonomy negatively


Steering committees and the like
--------------------------------


Sources:
- The Everything Store: Jeff Bezos and the Age of Amazon
- https://puppet.com/resources/whitepaper/state-of-devops-report
- https://info.thoughtworks.com/download-agile-it-organization-design.html
