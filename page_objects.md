Page Objects in JavaScript
==========================

Some JavaScript views are complex; they have a lot of markup and they have a lot of logic, and therefore, they are often hard to test. If these views are very susceptible to change they will likely burden you with a lot of exstra work keeping the tests running green. Failures may be caused by changes to the markup, some selector, or maybe some ajax response. Other complicating factors might be the views' frequency of use and how hard they are to mock.

This blog post will deal with testing complex views in a maintainable way through the use of _Page Objects_ (though the examples will be simple for educational purposes).

The examples will use an app where the user can add books to a library like below.

![The library app](/page_objects/img/1-library.png?raw=true)

Hiding access to the markup
---------------------------

The first thing you can do is to start wrapping your selectors inside some reusable abstractions. Consider for example the following test, where we expect a view to be hidden once we push the 'cancel' button.

TODO: A picture of the view

```javascript
it('hides the view when cancelling', function() {
    var view = new AddBookView();
    view.render();

    expect(view.$el).not.toHaveClass('hide');

    view.$(".cancel-button").click();

    expect(view.$el).toHaveClass('hide');
});
```

Hiding flow between views
-------------------------


Hiding backend communication
----------------------------

