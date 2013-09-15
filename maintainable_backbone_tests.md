Writing maintainable Backbone view tests
========================================

Testing complex Backbone.js views can be painful. These views often
contain quite a bit of markup and logic, resulting in tests that are
both cluttered and difficult to understand. Additionally, we often end
up duplicating DOM selectors and Ajax responses.

In this blog post we'll show one way of writing clean and maintainable
tests for Backbone views.

The example
-----------

Throughout the blog post we'll use a simple library application as an
example. The user can view a list of books:

![The library app](pageobjects/img/1-library.png?raw=true)

And click "add book" to get a new view for adding a book:

![The library app](pageobjects/img/2-add-book-view.png?raw=true)

Then the user can fill in some info about the book:

![The library app](pageobjects/img/3-adding-a-book.png?raw=true)

And then the book appears in the list:

![The library app](pageobjects/img/4-book-added.png?raw=true)

In this blog post we try to write tests for `AddBookView`, which is the
view that is opened when clicking "Add book" and closed when saving the
new book. Basically, we want to test that the adding of new books works
as intended. In detail, we want to:

1. Insert the name of the author
2. Insert the title of the book
3. Choose a genre from the drop-down
4. Press "Confirm"

A typical test for this Backbone view using Jasmine might look something
like this: (we've added some comments to clarify the intention)

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
    addBookView.$(".author-input").val("Miguel de Cervantes Saavedra");

    //  Set the title field
    addBookView.$(".title-input").val("Don Quixote");

    //  Choose a genre for the book
    var dropdown = addBookView.$(".genres-dropdown");
    dropdown.find(".dropdown-trigger").click();
    dropdown.find("a[data-value='Picaresco']").click();

    // Fake Ajax responses, so that we can see that the correct request
    // is sent
    var server = sinon.fakeServer.create();

    //  Save the book
    this.addBookView.$(".submit-button").click();

    //  Find the body of the last Ajax request
    var requestBody = server.queue[0].requestBody;

    // Respond to the Ajax request and restore XMLHttpRequest
    server.respond();
    server.restore();

    //  Check that we actually saved what we expect to have saved
    expect(JSON.parse(requestBody)).toEqual({
        author: "Miguel de Cervantes Saavedra",
        title: "Don Quixote",
        genre: "Picaresco"
    });
});
```

This a _lot_ of code, especially considering that the functionality
being tested is quite simple. Also, it's not easy to see precisely what
is being tested without reading all the code. Actually, in this test
about a fourth of the lines are dedicated to setting up the view. A lot
of that setup isn't even relevant for the functionality we are testing.

Clean up view creation
----------------------

The first step towards a cleaner test is to move the view creation into
a helper:

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
```

The result:

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

Now our setup is a bit cleaner and can easily be reused across tests.
When we first started testing our code, we often used `beforeEach` for
this type of setup, but there are significant problems with this
approach. First of all, when we have a lot of tests in a single file
it's often quite hard to find a setup that works for _all tests_. Also,
we often needed to jump to the `beforeEach` to see the current state of
the code. Additionally, it's also possible to have several `describe`s
inside each other, and therefore several `beforeEach`s who all work on
setting up the state. We prefer to use creation methods instead.

Hiding access to the markup
---------------------------

Writing a lot of jQuery selectors and triggering events on DOM objects
in our code can easily make our tests brittle in the long run. If we
removed the `author-input` field and added an `author-firstname-input`
field and an `author-lastname-input` field instead, we would have to
change quite a lot of all tests hitting this view. To alleviate this
problem and make our tests more robust we can wrap access to low-level
jQuery code.

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

     //  Fake ajax responses
     var server = sinon.fakeServer.create();

     //  Save the book
     this.addBookView.$(".submit-button").click();

     //  Responding with what was sent in
     var requestBody = server.queue[0].requestBody;
     server.respond();
     server.restore();

     //  Check that we really save what we expect to have saved
     expect(JSON.parse(requestBody)).toEqual({
         author: "Miguel de Cervantes Saavedra",
         title: "Don Quixote",
         genre: "Picaresco"
     });
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

The remaining clutter is related to faking ajax with sinon and asserting that the saved book is what we inserted. Let's start with cleaning up the ajax parts.

Hiding backend communication
----------------------------

Sinon in our tests gives us a lot of the same issues as with jQuery: A lot of low-level logic that isn't really relevant to the test. So, the first thing we can do is to write a `save(book)` method, where the provided `book` parameter is the the JavaScript object we expect to have saved.

```diff
 it('saves the book', function() {
     var addBookView = createAddBookView({ genres: ["Picaresco"] });
     addBookView.render();

     addBookViewPageObject(addBookView.$el).
         author("Miguel de Cervantes Saavedra").
         title("Don Quixote").
         genre("Picaresco").
+        save({
+            author: "Miguel de Cervantes Saavedra",
+            title: "Don Quixote",
+            genre: "Picaresco"
+        });

-    //  Fake ajax responses
-    var server = sinon.fakeServer.create();
-
-    //  Save the book
-    this.addBookView.$(".submit-button").click();
-
-    //  Responding with what was sent in
-    var requestBody = server.queue[0].requestBody;
-    server.respond();
-    server.restore();
-
-    //  Check that we really save what we expect to have saved
-    expect(JSON.parse(requestBody)).toEqual({
-        author: "Miguel de Cervantes Saavedra",
-        title: "Don Quixote",
-        genre: "Picaresco"
-    });
 });
```
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
             dropDownViewPageObject($el.find(".genres-dropdown")).
                 openMenu().
                 chooseOption(genre);
             return this;
         },
+        save: function(book) {
+            var server = sinon.fakeServer.create();
+
+            $el.find(".submit-button").click();
+
+            var requestBody = server.queue[0].requestBody;
+
+            server.respond();
+            server.restore();
+
+            expect(JSON.parse(requestBody)).toEqual(book);
+        }
     };
 };
```

However, there are some problems with having the `save` method doing the assertion. E.g. we might in some tests want to assert that _no_ book is saved if we add client-side validation. One solution is to have the `save` method return a new object containing all the relevant expect methods.

```diff
 it('saves the book', function() {
     var addBookView = createAddBookView({ genres: ["Picaresco"] });
     addBookView.render();

     addBookViewPageObject(addBookView.$el).
         author("Miguel de Cervantes Saavedra").
         title("Don Quixote").
         genre("Picaresco").
-        save({
-            author: "Miguel de Cervantes Saavedra",
-            title: "Don Quixote",
-            genre: "Picaresco"
-        });
+        save().
+        expectToHaveSaved({
+            author: "Miguel de Cervantes Saavedra",
+            title: "Don Quixote",
+            genre: "Picaresco"
+        });
 });
```
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
             dropDownViewPageObject($el.find(".genres-dropdown")).
                 openMenu().
                 chooseOption(genre);
             return this;
         },
-        save: function(book) {
+        save: function() {
             var server = sinon.fakeServer.create();
 
             $el.find(".submit-button").click();
 
             var requestBody = server.queue[0].requestBody;
 
             server.respond();
             server.restore();
 
-            expect(JSON.parse(requestBody)).toEqual(book);
+            return {
+                expectToHaveSaved: function(attributes) {
+                    expect(JSON.parse(requestBody)).toEqual(attributes);
+                }
+            };
         }
     };
 };
```

In the end the test looks like this:

```javascript
 it('saves the book', function() {
     var addBookView = createAddBookView({ genres: ["Picaresco"] });
     addBookView.render();

     addBookViewPageObject(addBookView.$el).
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

It has gotten quite slimmer since we started. There's almost no terms in the test that isn't relevant to the functionality being tested.

Wrapup
------

To make it even easier to create Page Objects use can use [po.js](https://github.com/kjbekkelund/po.js)

`addBookViewPageObject` might then look like the following:

```javascript
 var addBookViewPageObject = po.create({
     author: po.input(".author-input"),
     title: po.input(".title-input"),
     cancel: po.button(".cancel-button"),
     submit: po.button(".submit-button"),
     genre: function(genre) {
         dropDownViewPageObject(this.$el.find(".genres-dropdown")).
             openMenu().
             chooseOption(genre);
         return this;
     },
     save: function() {
         return this.mockRequest(function() {
             this.submit();
         }, this);
     },
     mockRequest: function(callback, context) {
         var server = sinon.fakeServer.create();

         callback.call(context || this);

         this.lastRequestBody = server.queue[0].requestBody;

         server.respond();
         server.restore();

         return this;
     },
     expectToHaveSaved: function(attributes) {
         var requestBody = JSON.parse(this.lastRequestBody);
         var obj = _.pick(requestBody, _.keys(attributes));

         expect(obj).toEqual(attributes);

         return this;
     }
 });
```

The code examples can be found at [https://github.com/sveinung/pageobject-example](https://github.com/sveinung/pageobject-example)
