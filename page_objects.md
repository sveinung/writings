Page Objects in JavaScript
==========================

Some JavaScript views are complex; they have a lot of markup and they have a lot of logic, and therefore, they are often hard to test. If these views are very susceptible to change they will likely burden you with a lot of exstra work keeping the tests running green. Failures may be caused by changes to the markup, some selector, or maybe some ajax response. Other complicating factors might be how much the views are used, and how hard they are to mock.

This blog post will deal with testing complex views in a maintainable way through the use og _Page Objects_ (though the examples will be simple for educational purposes).

Hiding access to the markup
---------------------------


Hiding flow between views
-------------------------


Hiding backend communication
----------------------------

