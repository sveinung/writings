Page Objects in JavaScript
==========================

Some JavaScript views are complex; they have a lot of markup and they have a lot of logic, and therefore, they are often hard to test. If these views are very susceptible to change they will likely burden you with a lot of exstra work keeping the tests running green. Failures may be caused by changes to the markup, some selector, or maybe some ajax response. Other complicating factors might be the views' frequency of use and how hard they are to mock.

This blog post will deal with testing complex views in a maintainable way through the use of _Page Objects_ (though the examples will be simple for educational purposes).

The examples will use an app where the user can view a list of books

![The library app](/page_objects/img/1-library.png?raw=true)

after clicking the cross

![The library app](/page_objects/img/2-add-book-view.png?raw=true)

you can add more books

![The library app](/page_objects/img/3-adding-a-book.png?raw=true)

and see them appear in the list

![The library app](/page_objects/img/4-book-added.png?raw=true)

Hiding access to the markup
---------------------------

The first thing you can do is to start wrapping your selectors inside some reusable abstractions. Consider for example the following test, where we expect a view to be hidden once we push the 'cancel' button.

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

