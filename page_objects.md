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
 it('chooses an option', function() {
     var view = new DropDownView({
         defaultOption: "Choose!",
         options: [{
             value: "Picaresco"
         }, {
             value: "Satire"
         }]
     });
     view.render();

     expect(view.$(".dropdown-trigger .chosen-value")).toHaveText("Choose!");

     view.$(".dropdown-trigger").click();
     view.$(".dropdown-menu a[data-value='Satire']").click();

     expect(view.$(".dropdown-trigger .chosen-value")).toHaveText("Satire");
 });
```

The test accesses the DOM directly, which want to avoid, so we create a page object and inject the view.

```diff
+var DropDownViewPageObject = function(dropDownView) {
+    this.view = dropDownView;
+};

 it('chooses an option', function() {
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

     expect(view.$(".dropdown-trigger .chosen-value")).toHaveText("Choose!");

     view.$(".dropdown-trigger").click();
     view.$(".dropdown-menu a[data-value='Satire']").click();

     expect(view.$(".dropdown-trigger .chosen-value")).toHaveText("Satire");
 });
```

Then we can move the interraction with the view into the page object. In this case opening the dropdown and choosing the option "Satire".

```diff
 var DropDownViewPageObject = function(dropDownView) {
     this.view = dropDownView;
 };

+_.extend(DropDownViewPageObject.prototype, {
+    openMenu: function() {
+        this.view.$(".dropdown-trigger").click();
+        return this;
+    },
+    chooseOption: function(option) {
+        this.view.$(".dropdown-menu a[data-value='" + option + "']").click();
+        return this;
+    }
+});

 it('chooses an option', function() {
     var view = new DropDownView({
         defaultOption: "Choose!",
         options: [{
             value: "Picaresco"
         }, {
             value: "Satire"
         }]
     });
     view.render();
     var pageObject = new DropDownViewPageObject(view);

     expect(view.$(".dropdown-trigger .chosen-value")).toHaveText("Choose!");

-    view.$(".dropdown-trigger").click();
-    view.$(".dropdown-menu a[data-value='Satire']").click();
+    pageObject.
+        openMenu().
+        chooseOption("Satire");

     expect(view.$(".dropdown-trigger .chosen-value")).toHaveText("Satire");
 });
```

Personally, I also like to move the assertions into the page objects if it _improves readability_, or if the same assertions are _used frequently_.

```diff
 var DropDownViewPageObject = function(dropDownView) {
     this.view = dropDownView;
 };

 _.extend(DropDownViewPageObject.prototype, {
     openMenu: function() {
         this.view.$(".dropdown-trigger").click();
         return this;
     },
     chooseOption: function(option) {
         this.view.$(".dropdown-menu a[data-value='" + option + "']").click();
         return this;
+    },
+    expectToHaveChosen: function(option) {
+        expect(this.view.$(".dropdown-trigger .chosen-value")).toHaveText(option);
+        return this;
+    },
+    expectToBeHidden: function() {
+        expect(this.view.$(".dropdown-menu")).toHaveClass("hide");
+        return this;
     }
 });

 it('chooses an option', function() {
     var view = new DropDownView({
         defaultOption: "Choose!",
         options: [{
             value: "Picaresco"
         }, {
             value: "Satire"
         }]
     });
     view.render();
     var pageObject = new DropDownViewPageObject(view);

-    expect(view.$(".dropdown-trigger .chosen-value")).toHaveText("Choose!");

     pageObject.
-        openMenu().
-        chooseOption("Satire");
+        expectToHaveChosen("Choose!").
+        openMenu().
+        chooseOption("Satire").
+        expectToHaveChosen("Satire");

-    expect(view.$(".dropdown-trigger .chosen-value")).toHaveText("Satire");
 });
```

In the end we end up with the code in the block below. Notice how much more readable the test is!

```javascript
 var DropDownViewPageObject = function(dropDownView) {
     this.view = dropDownView;
 };

 _.extend(DropDownViewPageObject.prototype, {
     openMenu: function() {
         this.view.$(".dropdown-trigger").click();
         return this;
     },
     chooseOption: function(option) {
         this.view.$(".dropdown-menu a[data-value='" + option + "']").click();
         return this;
     },
     expectToHaveChosen: function(option) {
         expect(this.view.$(".dropdown-trigger .chosen-value")).toHaveText(option);
         return this;
     },
     expectToBeHidden: function() {
         expect(this.view.$(".dropdown-menu")).toHaveClass("hide");
         return this;
     }
 });

 it('chooses an option', function() {
     var view = new DropDownView({
         defaultOption: "Choose!",
         options: [{
             value: "Picaresco"
         }, {
             value: "Satire"
         }]
     });
     view.render();
     var pageObject = new DropDownViewPageObject(view);

     pageObject.
         expectToHaveChosen("Choose!").
         openMenu().
         chooseOption("Satire").
         expectToHaveChosen("Satire");
 });
```

Hiding flow between views
-------------------------


Hiding backend communication
----------------------------

