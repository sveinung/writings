Writing maintainable Backbone view tests
========================================

Testing complex JavaScript views is a pain. These views often contain lots of markup and logic, resulting in tests that are both cluttered and difficult to understand. Additionally, there is often duplicated code, such as DOM selectors and Ajax responses. In my experience, writing maintainable tests for these views can be greatly improved by using _Page Objects_.

// what is page object (2 setninger)
// kanskje noe sånt som "facade for viewene dine", eller noe i den retningen

The example
-----------

The examples will use an app where the user can view a list of books

![The library app](pageobjects/img/1-library.png?raw=true)

after clicking the cross

![The library app](pageobjects/img/2-add-book-view.png?raw=true)

you can add more books

![The library app](pageobjects/img/3-adding-a-book.png?raw=true)

and see them appear in the list

![The library app](pageobjects/img/4-book-added.png?raw=true)

We want to test that the adding of new books works as intended. In detail, we want to:
1. Insert the name of the author
2. Insert the title of the book
3. Choose a genre from the drop-down

A typical test of a Backbone view looks like this:

```javascript
 it('saves the book', function() {
     //  Create some genres for the drop-down
     var genres = new Genres([
         { name: "Crime novel" },
         { name: "Picaresco" }
     ]);

     //  The book we are going to save
     var book = new Book();

     //  The view we are testing
     var addBookView = new AddBookView({
         genres: genres,
         book: book
     });
     addBookView.render();

     //  Set the author field
     addBookView.$(".author-input").
         val("Miguel de Cervantes Saavedra").
         change();

     //  Set the title field
     addBookView.$(".title-input").
         val("Don Quixote").
         change();

     //  Choose a genre for the book
     var dropdown = addBookView.$(".genres-dropdown");
     dropdown.find(".dropdown-trigger").click();
     dropdown.find("a[data-value='Picaresco']").click();

     //  Listen to the 'sync' event so that we can check what we saved later
     var callback = sinon.spy();
     this.addBookView.book.on('sync', callback);

     //  Fake ajax responses
     var server = sinon.fakeServer.create();

     //  Save the book
     this.addBookView.$(".submit-button").click();

     //  Responding with what was sent in
     var response = server.queue[0].requestBody;
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

This a _lot_ of code, especially considering that the functionality being tested is quite simple.

In this test about a fourth of the lines are dedicated to setting up the view. A lot of that setup isn't even relevant for the functionality we are testing.

Clean up view creation
----------------------

The first thing we can do is to move view creation into a helper function.

```diff
 it('saves the book', function() {
-    //  Create some genres for the drop-down
-    var genres = new Genres([
-        { name: "Crime novel" },
-        { name: "Picaresco" }
-    ]);
-
-    //  The book we are going to save
-    var book = new Book();
-
-    //  The view we are testing
-    var addBookView = new AddBookView({
-        genres: genres,
-        book: book
-    });

+    var addBookView = createAddBookView({ genres: ["Picaresco"] });
     addBookView.render();

     //  The rest of the test
 });

+function createAddBookView(options) {
+    options = options || {};
+
+    var genres = [];
+    if (options.genres) {
+        genres = _.map(options.genres, function(genre) {
+            return { name: genre }
+        });
+    }
+
+    return new AddBookView({
+        genres: new Genres(genres),
+        book: new Book()
+    });
+}
```

```javascript
 it('saves the book', function() {
     var addBookView = createAddBookView({ genres: ["Picaresco"] });
     addBookView.render();

     //  The rest of the test
 });

 function createAddBookView(options) {
     options = options || {};
 
     var genres = [];
     if (options.genres) {
         genres = _.map(options.genres, function(genre) {
             return { name: genre }
         });
     }
 
     return new AddBookView({
         genres: new Genres(genres),
         book: new Book()
     });
 }
```

This cleaned up the test quite a bit, but there's still a lot to do, like removing markup-specific code.

Hiding access to the markup
---------------------------

```javascript
 it('saves the book', function() {
     var addBookView = createAddBookView({ genres: ["Picaresco"] });
     addBookView.render();

     //  Set the author field
     addBookView.$(".author-input").
         val("Miguel de Cervantes Saavedra").
         change();

     //  Set the title field
     addBookView.$(".title-input").
         val("Don Quixote").
         change();

     //  Choose a genre for the book
     var dropdown = addBookView.$(".genres-dropdown");
     dropdown.find(".dropdown-trigger").click();
     dropdown.find("a[data-value='Picaresco']").click();

     //  The rest of the test
 });
```

Writing a lot of jQuery selectors and triggering events on DOM objects in our code can easily make our tests brittle in the long run. If we removed the `author-input` field and added an `author-firstname-input` field and an `author-lastname-input` field instead, we would have to change quite a lot of all tests hitting this view. To alleviate this problem and make our tests more robust we can wrap access to low-level jQuery code.

```diff
 it('saves the book', function() {
     var addBookView = createAddBookView({ genres: ["Picaresco"] });
     addBookView.render();

     //  Set the author field
     addBookView.$(".author-input").
         val("Miguel de Cervantes Saavedra").
         change();

     //  Set the title field
     addBookView.$(".title-input").
         val("Don Quixote").
         change();

     //  Choose a genre for the book
     var dropdown = addBookView.$(".genres-dropdown");
     dropdown.find(".dropdown-trigger").click();
     dropdown.find("a[data-value='Picaresco']").click();

     //  The rest of the test
 });
```

So, instead of calling e.g. `$(".author-input")` in our tests, we want to do something like the following:

```diff
 it('saves the book', function() {
     var addBookView = createAddBookView({ genres: ["Picaresco"] });
     addBookView.render();

-     //  Set the author field
-     addBookView.$(".author-input").
-         val("Miguel de Cervantes Saavedra").
-         change();
-
-     //  Set the title field
-     addBookView.$(".title-input").
-         val("Don Quixote").
-         change();

-    //  Choose a genre for the book
-    var dropdown = addBookView.$(".genres-dropdown");
-    dropdown.find(".dropdown-trigger").click();
-    dropdown.find("a[data-value='Picaresco']").click();

+    addBookViewPageObject(addBookView.$el).
+        author("Miguel de Cervantes Saavedra").
+        title("Don Quixote").
+        genre("Picaresco");

     //  The rest of the test
 });
```

To achieve this, we can wrap the jQuery code in an object.

```diff
+function addBookViewPageObject($el) {
+    return {
+        author: function(author) {
+            $el.find(".author-input").
+                val(author).
+                change();
+            return this;
+        },
+        title: function(title) {
+            $el.find(".title-input").
+                val(title).
+                change();
+            return this;
+        },
+        genre: function(genre) {
+            var dropdown = $el.find(".genres-dropdown");
+            dropdown.find(".dropdown-trigger").click();
+            dropdown.find("a[data-value='" + genre + "']").click();
+            return this;
+        }
+    };
+};
```

An object such as this is called a _Page Object_, as it's an abstraction used for interacting with the DOM.

```javascript
 it('saves the book', function() {
     var addBookView = createAddBookView({ genres: ["Picaresco"] });
     addBookView.render();

     addBookViewPageObject(addBookView.$el).
         author("Miguel de Cervantes Saavedra").
         title("Don Quixote").
         genre("Picaresco");

     //  The rest of the test
 });
```

```javascript
 function addBookViewPageObject($el) {
     return {
         author: function(author) {
             $el.find(".author-input").
                 val(author).
                 change();
             return this;
         },
         title: function(title) {
             $el.find(".title-input").
                 val(title).
                 change();
             return this;
         },
         genre: function(genre) {
             var dropdown = $el.find(".genres-dropdown");
             dropdown.find(".dropdown-trigger").click();
             dropdown.find("a[data-value='" + genre + "']").click();
             return this;
         }
     };
 };
```

That jQuery code dealing with the drop-down doesn't really belong in a Page Object for AddBookView, so we should move it into a separate Page Object. This new Page Object should deal with opening the drop-down and choosing an option from the list.

```diff
 function addBookViewPageObject($el) {
     return {
         author: function(author) {
             $el.find(".author-input").
                 val(author).
                 change();
             return this;
         },
         title: function(title) {
             $el.find(".title-input").
                 val(title).
                 change();
             return this;
         },
         genre: function(genre) {
-            var dropdown = $el.find(".genres-dropdown");
-            dropdown.find(".dropdown-trigger").click();
-            dropdown.find("a[data-value='" + genre + "']").click();

+            dropDownViewPageObject($el.find(".genres-dropdown")).
+                openMenu().
+                chooseOption(genre);
             return this;
         }
     };
 };
```
```diff
+function dropDownViewPageObject($view) {
+    return {
+        openMenu: function() {
+            $view.find(".dropdown-trigger").click();
+            return this;
+        },
+        chooseOption: function(option) {
+            $view.find(".dropdown-menu a[data-value='" + option + "']").click();
+            return this;
+        }
+    }
+}
```

The result is as following:

```javascript
 it('saves the book', function() {
     var addBookView = createAddBookView({ genres: ["Picaresco"] });
     addBookView.render();

     addBookViewPageObject(addBookView.$el).
         author("Miguel de Cervantes Saavedra").
         title("Don Quixote").
         genre("Picaresco");

     //  The rest of the test
 });
```
```javascript
 function addBookViewPageObject($el) {
     return {
         author: function(author) {
             $el.find(".author-input").
                 val(author).
                 change();
             return this;
         },
         title: function(title) {
             $el.find(".title-input").
                 val(title).
                 change();
             return this;
         },
         genre: function(genre) {
             dropDownViewPageObject($el.find(".genres-dropdown")).
                 openMenu().
                 chooseOption(genre);
             return this;
         }
     };
 };
```
```javascript
 function dropDownViewPageObject($view) {
     return {
         openMenu: function() {
             $view.find(".dropdown-trigger").click();
             return this;
         },
         chooseOption: function(option) {
             $view.find(".dropdown-menu a[data-value='" + option + "']").click();
             return this;
         }
     }
 }
```

Hiding backend communication
----------------------------

The next thing we will look at is hiding ajax calls and responses. In this example we will look at test that asserts that the AddBookView saves a new book.

// Forresten, jeg ville injecta denne: https://github.com/sveinung/pageobject-example/blob/caf904d3d6f1009e5c9d4ec217fcb51b62600c6b/src/main/webapp/modules/library/books/addBookView.js#L20
// det gjøre testen bedre også. Liker ikke at det plutselig kommer en `view.book` der.

```javascript
 it('saves the book', function() {
     var genres = new Genres([
         {"name":"Crime novel"},
         {"name":"Picaresco"}
     ]);
     var book = new Book();
     var view = new AddBookView({
         genres: genres,
         book: book
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

+    var addBookViewPageObject = new AddBookViewPageObject(view.$el);

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

+    addBookViewPageObject.
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

     var addBookViewPageObject = new AddBookViewPageObject(view.$el);

     var callback = sinon.spy();
     view.book.on('sync', callback);

     addBookViewPageObject.
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

     var addBookViewPageObject = new AddBookViewPageObject(view.$el);

     var callback = sinon.spy();
     view.book.on('sync', callback);

     addBookViewPageObject.
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
     var book = new Book();
     var view = new AddBookView({
         genres: genres,
         book: book
     });
     view.render();

     var addBookViewPageObject = new AddBookViewPageObject(view);

     addBookViewPageObject.
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

