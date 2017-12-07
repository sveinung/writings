Autonomous teams – Organizing for Speed and Agility
===================================================

Lead time is the time it takes you to complete a process, for instance from conceiving an idea to actually having something implemented in production. Shortening this lead time makes everything better. If you think it doesn't apply to you you're probably ripe for being naturally selected in really really bad way.

For innovation to work optimally learning has to be as time and cost efficient as possible. Increase lead time and you'll reduce your learning rate. Increase deployment costs and you'll reduce the number of hypotheses you can run for the same amount of money, thereby reducing the number of features you can try out before running out of money or having your steering committee lose their patience and kicking your butt out on the street.

This blogposts highlights some common organizational pitfalls that negatively impacts your ability to deliver value to your customers.

An organization can either optimize a value stream for _utilization_ or _throughput_. Not both. Since throughput makes more sense if you'd like to make it easier to be innovative I'm going to assume throughput is your cup of tea.


Getting rid of feature handovers
--------------------------------

Anyone who has ever worked on anything with other people have experienced that the one thing impacting lead time the most is handovers. Some handovers we can't easily be without if we need people to specialize as developers, designers, and so on. Some handovers, though, we can avoid by designing our organization the right way.

Different teams have different goals and backlogs – it's why we split our organizations in multiple teams in the first place, to be able to work concurrently with more than one thing. And because different teams have different goals and backlogs they'll also have different priorities. The conclusion is that if we design our organization so that every feature requires work done on more than one team we're going to have a bad time! That includes "thinking work" by business people and other people who like to "analyze".

<un-agile-org.png>

As long as you divide your people into teams you're going to have silos and suboptimization. Just by existing, teams act as barriers to communication and cooperation (this is due to them having different goals than other teams, as we stated above). [1] What you need to do is to _organize your teams in a way that causes suboptimization to be as optimal as possible_. If you're optimizing for speed and agility, you want to reduce the number of handovers needed to deliver customer value and to test hypotheses. If that means increasing the number of handovers needed to implement _shared services_, so be it.

What you should do is to organize your teams around business outcomes and not competencies or even applications. [3] If you organize around competencies or applications you're most likely going to have a significant amount of handovers per feature you want implemented. If you organize around business outcomes – one outcome, one team – you'll avoid feature handovers. You might still have to endure handovers due to _shared services_ (e.g. a common style guide or common SSO solution), but such work doesn't have to be a part of every feature you implement, like frontend or database work probably will.


Shared services
---------------

Sometimes we need to get something done with a shared service to deliver value. This can be giving our apps to operations for deployment into production, adding some configuration to a message broker or ordering more virtual machines.

The people maintaining these shared services doesn't just hang around and wait for your every command. They too have backlogs and priorities different from your own. Therefore if you need something from them and the way you get that is by getting them to do it – e.g. deploying your app – you're going to get idle time in your process. Simply because you're getting involved in another team's process.

Now, if you want teams who depend on these shared services to optimize for autonomy, these services should be self-serviced.

This way, you'll have turned a _process_ into a _resource_, and resources are much less intrusive and much easier to reuse across teams and projects. Take for instance an internal startup embedded in an established organization. This startup will require different processes to function. By turning the process into a resource you make it easier to reuse the capabilities you might have, and thereby possibly increasing your chances of being innovative as well. [4]

In the case of operations, this is where public cloud really shines and where on-premise operations platforms lag behind. A large reason why a public cloud provider is able to provide a self-serviced platform for many customers is because they have a lot of spare capacity. They're economically incentivized to do so. If it wasn't for spare capacity you'd get resource contentions between different clients and that would hurt their wallets. On-premise platform providers have economical incentives to _not_ have spare capacity, because spare capacity is wasted money. And that simply will not do if you're viewed as a cost center, as many on-premise operations platforms are. The unfortunate result is that the operations platform will become a scarce resource. Scarce resources are protected and protection will hurt autonomy.

Steering committees and the like
--------------------------------

Having a short lead time alone isn't enough if your backlog easily fills up with garbage.

People inside the team are exposed to most of the feedbackloops related to the development of the product or service. Seems natural doesn't it? Those who work with something on a daily basis know more about it than those who doesn't. The conclusion is that product owners, steering committee members and other people outside the team aren't really qualified in making good decisions concerning the product.


Sources:
- 1 The Everything Store: Jeff Bezos and the Age of Amazon
- 2 https://puppet.com/resources/whitepaper/state-of-devops-report
- 3 https://info.thoughtworks.com/download-agile-it-organization-design.html
- 4 The Innovators Dilemma.
