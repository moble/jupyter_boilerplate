# Jupyter notebook boilerplate

Adds a menu item to Jupyter notebooks (previously IPython notebooks) to insert
boilerplate, snippets, and examples of code.

This notebook extension adds a menu item (or multiple menu items) after the
`Help` menu in Jupyter notebooks.  This new menu contains little snippets of
code that we all forget from time to time but don't want to google, or are just
too lazy to type.  It can also be helpful for people just starting out with
python, who need some ideas for what to do next -- like defining a function or
a class.

The new menu comes with a default value relevant for python programming (though
this can be adjusted, as shown below).  It is called "Boilerplate", and
contains sub-menus with snippets for a few popular python packages, as well as
basic python and notebook markdown.  Here's a screenshot of the menu, opened on
the "Matplotlib" option:

![Opened boilerplate menu](boilerplate_screenshot.png)

So, for example, if you are editing a code cell and want to import matplotlib
for use in the notebook, you can just click the "Boilerplate" menu, then mouse
over "Matplotlib".  This will open up a new sub-menu, with an item "Set up for
notebook".  Clicking on that item will insert the code snippet at the point
where your cursor was just before you clicked on the menu.  In particular, for
this `matplotlib` example, the following code gets inserted:

```python
import matplotlib as mpl
import matplotlib.pyplot as plt
%matplotlib inline
```

The inserted text will be selected, so that you can delete it by pressing
backspace or delete, or you can just select another snippet to replace it.

Note that many of the snippets involve variable names prefixed with `bp_`.  For
example, a new numpy array is created as `bp_new_array`.  These are
intentionally dumb names that you really should replace.  Failing to do so
could lead to ugly bugs in your code.

Similarly, some strings are intended to be replaced, such as the axis labels in
plots.  These are there to show you what can be done, and to remind you to put
informative labels in your plots.  If you don't want, e.g., a title on your
plot, just remove that line.


# Installation

You can download the file for this extension with the following command run in
an ipython cell (or remove `%%bash` and run from the command line):

```bash
%%bash
curl -L https://rawgithub.com/moble/ipynb_boilerplate/master/boilerplate.js > $(ipython locate)/nbextensions/boilerplate.js
echo $(ipython profile locate)/static/custom
```

Now, that should output the name of a directory.  You'll need to edit the
`custom.js` file in that directory and add the following lines:

```javascript
$([IPython.events]).on('app_initialized.NotebookApp', function(){

    require(["nbextensions/boilerplate"], function (boilerplate_extension) {
        console.log('Loading `boilerplate` notebook extension');
        var menus = boilerplate_extension.boilerplate_menus;
        boilerplate_extension.load_ipython_extension(menus);
    });

})
```

If you start a new notebook (or refresh any open ones), you should now see the
"Boilerplate" menu.


# Customizing the menu(s)

The default menu might have irrelevant stuff for you, or may not have something
you would find useful.  You can easily customize it by adjusting the `menus`
variable defined in `custom.js` (as seen above).  The `menu` is a nested
[javascript "object"](http://api.jquery.com/Types/#Object) (which is just like
a python dictionary).  So to change the `menu`, you just have to change that
object.

## Add or update a simple menu item

Suppose you want to add a new command to the "Numpy" sub-menu.  You can do that
by inserting some lines into your `custom.js`, so that it looks like this:

```javascript
        var menus = boilerplate_extension.boilerplate_menus;
        $.extend(true, menus, {
            'Boilerplate' : {
                'Numpy' : {
                    'My new command': 'np.new_command_code()',
                },
            },
        });
        boilerplate_extension.load_ipython_extension(menus);
```

The first and last lines shown here were already placed in your `custom.js`
during installation; we've just added that `extend` stuff.  Now, if you refresh
your notebook, you'll see a new menu item named "My new command".  And if you
click it, it will insert `np.new_command_code()` into your notebook.

If you want to change/update the content of, say, the numpy "Import" command,
just add your version; the old one will be replaced.

You can also add or change more than one thing at a time.  Just add new things
after any of the commas in the menu above.


## Add a menu item with more complicated code

The example above inserted a simple one-line snippet of code, and didn't have
any quotation marks (single or double).  Unfortunately, javascript doesn't deal
well with strings.  (There are no raw triple-quoted strings, like in python.)
So if you want to insert code with multiple lines, or with quotation marks, you
need to be a little more careful.  Fortunately, this extension provides a handy
escaping function to help with this.

It's best described with another example.  Let's insert code like above, but
with some more lines and some quotes:

```javascript
        var menus = boilerplate_extension.boilerplate_menus;
        var escape = boilerplate_extension.escape_strings;
        $.extend(true, menus, {
            'Boilerplate' : {
                'Numpy' : {
                    'My new command': escape(['np.new_command_code()',
                                              'np.other_new_code("with strings!")',
                                              'np.string_craziness(\'escape single quotes once\')',]),
                },
            },
        });
        boilerplate_extension.load_ipython_extension(menus);
```

As you can see, each line of code is a separate string in an array, and that
array is passed to the `escape` function.  Note that the strings are in
single-quotes (`'`), which means that any single quotes you want to appear in
the notebook have to be escaped (`\'`), but double quotes (`"`) don't.  So
generally, I just stick to double quotes in my snippets.  But you can do
whatever you prefer.


## Delete a menu item

Suppose you don't care about scipy.  You can delete that whole submenu by
inserting a line in `custom.js` so that your file looks like this:

```javascript
        var menus = boilerplate_extension.boilerplate_menus;
        delete menus['Boilerplate']['Scipy'];
        boilerplate_extension.load_ipython_extension(menus);
```


## Rename a menu item

Suppose you want to change the label "Sympy" to read "Symbolic python".  You
have to copy the old "Sympy" menu into the new one, and then delete the old one:

```javascript
        var menus = boilerplate_extension.boilerplate_menus;
        menus['Boilerplate']['Symbolic python'] = menus['Boilerplate']['Sympy'];
        delete menus['Boilerplate']['Sympy'];
        boilerplate_extension.load_ipython_extension(menus);
```

Note, however, that this will place the renamed menu at the bottom of the list,
since I used objects (see [TODO list](#TODO)).  Hopefully I'll fix this in
future updates.

## More menu fun

Note that "Boilerplate" is inside the first level of the `menus` object.  If
you want to add an entirely new menu visible at the top of the notebook, you
could just `extend` the `menus` object with another group.  For example, if you
use "Pandas" a lot, you might want all of those commands readily available.

You might even think all my snippets are dumb, and you want to just start over.
That's no problem; just create your own `menus` variable.  But you might want
to use my original as a guide, so you can find it in the `boilerplate.js` file.
And in general, it's just a javascript object, so you'll find answers to all
your manipulation needs by googling.


# TODO

I'm new to javascript, so I based my menu structure on objects, not realizing
that they have no order.  So it's impossible to re-order the menus.  I
apparently should have used arrays.

Also, I hadn't seen jQuery before, so some of my code is quite ugly.  It still
works, but I could bring it more into line with other ipython programming.

There are still some "not yet implemented" functions, that are mostly just
placeholders to remind me to do something about them.  Scipy needs some real
functions, and the matplotlib examples can probably be easily copied from the
gallery.  Generally, more boilerplate can always be added.

It would be nice to add a couple improvements to the menu-item formatting.  For
example, it should be possible (and easy!) to take an arbitrary javascript
function as the command to be executed `onClick`.

Or maybe we want the top level of a sub-menu to be clickable -- for example,
where there's one basic option that might be needed frequently, but then a
sub-menu with some more advanced options.  In this case, the item's value would
probably be an array, with first element a string, and second element the
sub-menu.