Public Cloud will always be better
==================================

Alternativ tittel: Don't use on-premise operations platforms

Spare capacity
--------------

Self-serviced platform have a lot of spare capacity, which they're economically incentivized to have. Without spare capacity you'd get resource contentions between different clients and that would reduce their income. If you use an on-premise operations platform instead, something else happens. With on-prem you'll have economic incentives to _not_ have spare capacity, because spare capacity is wasted money. And that simply will not do if the on-premise operations platform is viewed as a cost center. The unfortunate result is that the operations platform will become a scarce resource. Scarce resources are protected and this protection will reduce autonomy.

E.g., imagine having an on-premise database as the main storage of your organization. Each team that needs storage gets their own separate spaces (or schemas) in this database. Now, an internal startup wants to test something radically new to try to beat your main competitor. As this organization has standardized on this database the internal startup has no choice but to use the same storage as everybody else. The difference between the internal startup and the other teams is that the startup has no idea how their new product is going to be used. If their product turns out to be a real hit they could very well end up spending a lot more of the databases resources than the organizations established products. In the worst case scenario new products ends up impacting the ability of established products to create customer value. The database administrators might well be aware of this and as a counter-measure institute control mechanisms to avoid anyone spending more resources than planned (for example with up-front performance testing). These control mechanisms are added development costs for the internal startup and therefore a competative disadvantage compared to their competitors.


Upfront investments
-------------------

Cloud providers can invest lots of money upfront in hiding their operations processes behind a self-serviced operations platform. They simply have to because of the number of clients they're aiming for.


Economic incentives
-------------------

https://blogg.bekk.no/organizing-for-speed-17462894baf4

Public cloud providers organize their services around business outcomes. In-house operations departments organize around outputs as operations in itself isn't what they sell to their customers, unless they're the operations department of Amazon or Google.


Outcome oriented teams' budgets are directly influenced by how well their products and services perform. If they sell more their budgets (should atleast) increase too.

Operations teams' budgets are seldom a function of how their clients (the development teams) use them. Often their budgets are either fixed or tied to how many people are on the operations team.

If you find you cannot use public cloud, you should at the very least find a way to tie your operations teams budget directly changes in the market affecting the development teams. If not, you may very well find that you miss market opportunities just because operations doesn't have the resources and incentives to scale.

Fixed bare-minimum budgets < Fixed ample budgets < Budgets tied to demand
