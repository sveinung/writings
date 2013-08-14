Testing complex JavaScript views using Page Objects
===================================================

Testing complex JavaScript views is a pain. These views often contain lots of markup and logic, resulting in tests that are both cluttered and difficult to understand. Additionally, there is often duplicated code, such as DOM selectors and Ajax responses. In my experience, writing maintainable tests for these views can be greatly improved by using _Page Objects_.

// what is page object (2 setninger)
// kanskje noe sånt som "facade for viewene dine", eller noe i den retningen

The example
-----------

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
+var DropDownViewPageObject = function($dropDownView) {
+    this.$view = $dropDownView;
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
+    var pageObject = new DropDownViewPageObject(view.$el);

     expect(view.$(".dropdown-trigger .chosen-value")).toHaveText("Choose!");

     view.$(".dropdown-trigger").click();
     view.$(".dropdown-menu a[data-value='Satire']").click();

     expect(view.$(".dropdown-trigger .chosen-value")).toHaveText("Satire");
 });
```

Then we can move the interraction with the view into the page object. In this case opening the dropdown and choosing the option "Satire".

```diff
 var DropDownViewPageObject = function($dropDownView) {
     this.$view = $dropDownView;
 };

+_.extend(DropDownViewPageObject.prototype, {
+    openMenu: function() {
+        this.$view.find(".dropdown-trigger").click();
+        return this;
+    },
+    chooseOption: function(option) {
+        this.$view.find(".dropdown-menu a[data-value='" + option + "']").click();
+        return this;
+    }
+});

 it('chooses an option', function() {
     var view = new DropDownView(…);
     view.render();
     var pageObject = new DropDownViewPageObject(view.$el);

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
 var DropDownViewPageObject = function($dropDownView) {
     this.$view = $dropDownView;
 };

 _.extend(DropDownViewPageObject.prototype, {
     openMenu: function() {
         this.$view.find(".dropdown-trigger").click();
         return this;
     },
     chooseOption: function(option) {
         this.$view.find(".dropdown-menu a[data-value='" + option + "']").click();
         return this;
+    },
+    expectToHaveChosen: function(option) {
+        expect(this.$view.find(".dropdown-trigger .chosen-value")).toHaveText(option);
+        return this;
+    },
+    expectToBeHidden: function() {
+        expect(this.$view.find(".dropdown-menu")).toHaveClass("hide");
+        return this;
     }
 });

 it('chooses an option', function() {
     var view = new DropDownView(…);
     view.render();
     var pageObject = new DropDownViewPageObject(view.$el);

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
 var DropDownViewPageObject = function($dropDownView) {
     this.$view = $dropDownView;
 };

 _.extend(DropDownViewPageObject.prototype, {
     openMenu: function() {
         this.$view.find(".dropdown-trigger").click();
         return this;
     },
     chooseOption: function(option) {
         this.$view.find(".dropdown-menu a[data-value='" + option + "']").click();
         return this;
     },
     expectToHaveChosen: function(option) {
         expect(this.$view.find(".dropdown-trigger .chosen-value")).toHaveText(option);
         return this;
     },
     expectToBeHidden: function() {
         expect(this.$view.find(".dropdown-menu")).toHaveClass("hide");
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
     var pageObject = new DropDownViewPageObject(view.$el);

     pageObject.
         expectToHaveChosen("Choose!").
         openMenu().
         chooseOption("Satire").
         expectToHaveChosen("Satire");
 });
```

Hiding backend communication
----------------------------

The next thing we will look at is hiding ajax calls and responses. In this example we will look at test that asserts that the AddBookView saves a new book.

```javascript
 it('saves the book', function() {
     var genres = new Genres([
         {"name":"Crime novel"},
         {"name":"Picaresco"}
     ]);
     var view = new AddBookView({
         genres: genres
     });
     view.render();

     var callback = sinon.spy();
     view.book.on('sync', callback);

     view.$(".author-input").
         val("Miguel de Cervantes Saavedra").
         change();

     view.$(".title-input").
         val("Don Quixote").
         change();

     var dropdown = view.$(".genres-dropdown");
     dropdown.find(".dropdown-trigger").click();
     dropdown.find("a[data-value='Picaresco']").click();

     var server = sinon.fakeServer.create();

     view.$(".submit-button").click();

     // Responding with what was sent in
     var response = server.queue[0].requestBody;
     server.respondWith([200, { "Content-Type": "application/json" }, response]);
     server.respond();
     server.restore();

     expect(callback).toHaveBeenCalledWith(sinon.match({
         attributes: {
             author: "Miguel de Cervantes Saavedra",
             title: "Don Quixote",
             genre: "Picaresco"
         }
     }));
 });
```

We quickly do the same thing we did in the previous example and hide interraction in a page object. This time, we are reusing the DropDownViewPageObject.

```diff
+var AddBookViewPageObject = function($addBookView) {
+    this.$view = $addBookView;
+    this.genreDropDown = new DropDownViewPageObject(this.$view.find(".genres-dropdown"));
+};
+_.extend(AddBookViewPageObject.prototype, {
+    author: function(author) {
+        this.$view.find(".author-input").
+            val(author).
+            change();
+        return this;
+    },
+    title: function(title) {
+        this.$view.find(".title-input").
+            val(title).
+            change();
+        return this;
+    },
+    genre: function(genre) {
+        this.genreDropDown.
+            openMenu().
+            chooseOption(genre);
+        return this;
+    }
+});

 it('saves the book', function() {
     …
     var view = new AddBookView({ genres: genres });
     view.render();

+    var pageObject = new AddBookViewPageObject(view.$el);

     var callback = sinon.spy();
     view.book.on('sync', callback);

-    view.$(".author-input").
-        val("Miguel de Cervantes Saavedra").
-        change();
-
-    view.$(".title-input").
-        val("Don Quixote").
-        change();
-
-    var dropdown = view.$(".genres-dropdown");
-    dropdown.find(".dropdown-trigger").click();
-    dropdown.find("a[data-value='Picaresco']").click();

+    pageObject.
+        author("Miguel de Cervantes Saavedra").
+        title("Don Quixote").
+        genre("Picaresco");

     var server = sinon.fakeServer.create();

     view.$(".submit-button").click();

     // Responding with what was sent in
     var response = server.queue[0].requestBody;
     server.respondWith([200, { "Content-Type": "application/json" }, response]);
     server.respond();
     server.restore();

     expect(callback).toHaveBeenCalledWith(sinon.match({
         attributes: {
             author: "Miguel de Cervantes Saavedra",
             title: "Don Quixote",
             genre: "Picaresco"
         }
     }));
 });
```

Afterwards we move the Sinon XHR stubbing into the page object.

```diff
 var AddBookViewPageObject = function($addBookView) {
     this.$view = $addBookView;
     this.genreDropDown = new DropDownViewPageObject(this.$view.find(".genres-dropdown"));
 };
 _.extend(AddBookViewPageObject.prototype, {
     author: function(author) { … },
     title: function(title) { … },
     genre: function(genre) { … },
+    save: function() {
+        var server = sinon.fakeServer.create();
+
+        this.$view.find(".submit-button").click();
+
+        // Responding with what was sent in
+        var response = server.queue[0].requestBody;
+        server.respondWith([200, { "Content-Type": "application/json" }, response]);
+        server.respond();
+        server.restore();
+    }
 });

 it('saves the book', function() {
     …
     var view = new AddBookView({ genres: genres });
     view.render();

     var pageObject = new AddBookViewPageObject(view.$el);

     var callback = sinon.spy();
     view.book.on('sync', callback);

     pageObject.
         author("Miguel de Cervantes Saavedra").
         title("Don Quixote").
         genre("Picaresco").
+        save();

-    var server = sinon.fakeServer.create();
-
-    view.$(".submit-button").click();
-
-    // Responding with what was sent in
-    var response = server.queue[0].requestBody;
-    server.respondWith([200, { "Content-Type": "application/json" }, response]);
-    server.respond();
-    server.restore();

     expect(callback).toHaveBeenCalledWith(sinon.match({
         attributes: {
             author: "Miguel de Cervantes Saavedra",
             title: "Don Quixote",
             genre: "Picaresco"
         }
     }));
 });
```

We have one potential little problem here. Each test that makes assertions about saving books will have intimate knowledge about the AddBookView. That is, that the view has an instance of a Book. If there's only a few tests, who cares (!), but if there are many we should probably clean it up. In this case we could inject the view itself instead of the jQuery object.

Notice that in order to avoid unnecessary state in the page object the `save` function returns an object with the XHR assertion function with the `saveCallback` bound in a closure.

```diff
 var AddBookViewPageObject = function($addBookView) {
     this.$view = $addBookView;
     this.genreDropDown = new DropDownViewPageObject(this.$view.find(".genres-dropdown"));
 };
 _.extend(AddBookViewPageObject.prototype, {
     author: function(author) { … },
     title: function(title) { … },
     genre: function(genre) { … },
     save: function() {
+        var saveCallback = sinon.spy();
+        this.view.book.on('sync', saveCallback);

         var server = sinon.fakeServer.create();
 
         this.$view.find(".submit-button").click();
 
         // Responding with what was sent in
         var response = server.queue[0].requestBody;
         server.respondWith([200, { "Content-Type": "application/json" }, response]);
         server.respond();
         server.restore();

+        return {
+            expectToHaveSaved: function(book) {
+                expect(saveCallback).toHaveBeenCalledWith(sinon.match({
+                    attributes: book
+                }));
+            }
+        };
     }
 });

 it('saves the book', function() {
     …
     var view = new AddBookView({ genres: genres });
     view.render();

     var pageObject = new AddBookViewPageObject(view.$el);

     var callback = sinon.spy();
     view.book.on('sync', callback);

     pageObject.
         author("Miguel de Cervantes Saavedra").
         title("Don Quixote").
         genre("Picaresco").
         save().
+        expectToHaveSaved({
+            author: "Miguel de Cervantes Saavedra",
+            title: "Don Quixote",
+            genre: "Picaresco"
+        });

-    expect(callback).toHaveBeenCalledWith(sinon.match({
-        attributes: {
-            author: "Miguel de Cervantes Saavedra",
-            title: "Don Quixote",
-            genre: "Picaresco"
-        }
-    }));
 });
```

The end result is then:

```javascript
 var AddBookViewPageObject = function(addBookView) {
     this.view = addBookView;
     this.genreDropDown = new DropDownViewPageObject(this.view.$(".genres-dropdown"));
 };
 _.extend(AddBookViewPageObject.prototype, {
     author: function(author) {
         this.view.$(".author-input").
             val(author).
             change();
         return this;
     },
     title: function(title) {
         this.view.$(".title-input").
             val(title).
             change();
         return this;
     },
     genre: function(genre) {
         this.genreDropDown.
             openMenu().
             chooseOption(genre);
         return this;
     },
     save: function() {
         // Use this when asserting the XHR response
         var saveCallback = sinon.spy();
         this.view.book.on('sync', saveCallback);

         var server = sinon.fakeServer.create();

         this.view.$(".submit-button").click();

         // Responding with what was sent in
         var response = server.queue[0].requestBody;
         server.respondWith([200, { "Content-Type": "application/json" }, response]);
         server.respond();
         server.restore();

         return {
             expectToHaveSaved: function(book) {
                 expect(saveCallback).toHaveBeenCalledWith(sinon.match({
                     attributes: book
                 }));
             }
         };
     }
 });

 it('saves the book', function() {
     var genres = new Genres([
         {"name":"Crime novel"},
         {"name":"Picaresco"}
     ]);

     var view = new AddBookView({
         genres: genres
     });
     view.render();

     var pageObject = new AddBookViewPageObject(view);

     pageObject.
         author("Miguel de Cervantes Saavedra").
         title("Don Quixote").
         genre("Picaresco").
         save().
         expectToHaveSaved({
             author: "Miguel de Cervantes Saavedra",
             title: "Don Quixote",
             genre: "Picaresco"
         });
 });
```

Hiding flow between views
-------------------------

```javascript
 var LibraryViewPageObject = function(libraryView) {
     this.view = libraryView;
 };
 _.extend(LibraryViewPageObject.prototype, {
     clickAddBook: function() {
         var self = this;
         var genresResponse = [{"name":"Crime novel"},{"name":"Picaresco"}];
         responseFaker.fakeResponse(genresResponse, {}, function() {
             self.view.$(".add-book").click();
         });
     }
 });
 it('shows the AddBookView', function() {
     var library = new Library([
         {"title":"Of Mice and Men","uri":"/book/1"},
         {"title":"Sult","uri":"/book/2"}
     ]);
     var libraryView = new LibraryView({
         library: library
     });
     libraryView.render();

     var pageObject = new LibraryViewPageObject(libraryView);

     pageObject.
         clickAddBook();

     expect(libraryView.$(".add-book-view")).not.toHaveClass("hide");
 });
```

