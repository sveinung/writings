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

Let's start with a simple view, for example the dropdown.

The first thing you can do is to start wrapping your selectors inside some reusable abstractions. Consider for example the following test, where we expect a view to be hidden once we push the 'cancel' button.

```javascript
 it('opens the dropdown', function() {
     var view = new DropDownView({
         defaultOption: "Choose!",
         options: [{
             value: "Picaresco"
         }, {
             value: "Satire"
         }]
     });
     view.render();

     expect(view.$(".dropdown-menu")).toHaveClass("hide");

     view.$(".dropdown-trigger").click();

     expect(view.$(".dropdown-menu")).not.toHaveClass("hide");
 });
```

The test accesses the DOM directly, which want to avoid, so we create a page object and inject the view.

```diff
+var DropDownViewPageObject = function(dropDownView) {
+    this.view = dropDownView;
+};

 it('opens the dropdown', function() {
     var view = new DropDownView({
         defaultOption: "Choose!",
         options: [{
             value: "Picaresco"
         }, {
             value: "Satire"
         }]
     });
     view.render();
+    var pageObject = new DropDownViewPageObject(view);

     expect(view.$(".dropdown-menu")).toHaveClass("hide");

     view.$(".dropdown-trigger").click();

     expect(view.$(".dropdown-menu")).not.toHaveClass("hide");
 });
```

Hiding flow between views
-------------------------


Hiding backend communication
----------------------------

