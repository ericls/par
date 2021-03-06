A parser for structured text
=============================

Sometimes I need a parser whick can parse structured text. I know I can
find such module for each type structured text format, such as:

* reStructuredText - [Docutils](http://docutils.sourceforge.net/)
* markdown - [pymarkdown](http://draketo.de/proj/pymarkdown_minisite/readme.html), 
    [python-markdown](http://packages.python.org/Markdown/)
    
But I still want to develop such thing myself, because I need to know the details
of how to develop such tool, and when I need I can modified it easily to suit
for my needs. So that's the reason why par developed. And I don't want to replace
any of above tools, but used for myself at first. And I also want to use
the same code base to implement different structured text format.

And I choose [pyPEG](http://fdik.org/pyPEG/) as the parser, I also test several modules, but I think this
one is simple enough. And I also modifed the pyPEG source so that it can suit for
me. What I do is adding some functions in Symbol class, and I also changed the Symbol
a bit. Such as:

    class Symbol(list):
        def __init__(self, name, what):
            self.__name__ = name
    #        self.append(name)          #removed by me
            self.what = what
    #        self.append((name, what))   this is old
            self.extend(what)           #this is new
    
        def render(self, index=0):
            """
            disply something like a tree, let me easy
            to find something:
            
                list_item:
                  number_list_item:
                            :'3.'
                    space:' '
                    list_content:
                      list_content_lines:
                        list_rest_of_line:
                                        :'Item 3'
                          eol:'\n'
                        list_content_line:'   * Item 3a'
                      blankline:
                        eol:'\n'
                
            """
            ...
        def find(self, name):
            """
            find a child node according the name, it'll
            only return the first matched child
            """
            ...
            
        def find_all(self, name):
            """
            find all child nodes one by one, it'll
            return a generator
            """
            
        @property
        def text(self):
            """
            Return the current node text, if there is child notes,
            it'll concat them all together.
            """
            
And I also write a simple Visitor class, so that the other visisor can 
derive from it. It has a `visit` method, just like:

    def visit(self, nodes, root=False):

And when it visiting the node, it'll search the whole nodes tree, and
it'll also invoke some dynamic functions just like:    

    if root:
        __begin__()       #if root is not Flase
    before_visit(node)    #invoke before visiting a node
    visit_<node.__name__>_begin(node)  #according current node name
    visit_<node.__name__>              #according current node name
    visit_<node.__name__>_end(node)    #according current node name
    after_visit(node)     #invoke after visiting a node
    if root:
        __end__()         #if root is not False
    
In Par there is two parsers for google code wiki format, and markdown format.
There are examples in examples folder.

markdown2html
--------------

This is an example of how to use par to parse markdown format text. It can be
used in command, for example:

    python markdown2html.py readme.md > readme.html

Paring Markdown Programmly
-----------------------------

Do like this:

    from par.md import parseHtml

    template = '''<!doctype html>
    <html>
    <head>
    <title>%(title)s</title>
    </head>
    <body>
    %(body)s
    </body>
    </html>'''

    tag_class = {'table':'table'}

    text = """
    # Test Markdown

    This is a pragraph
    """

    print parseHtml(text, template, tag_class)

Markdown Syntax Extend
-------------------------

### Table Support

do like this:

    || a || b || c ||
    || c || d || e ||

will get this:

    <table>
    <tr><td> a </td><td> b </td><td> c </td>
    </tr>
    <tr><td> b </td><td> c </td><td> d </td>
    </tr>
    </table>

### Definition List support

do like this:

    a --
        abc
    b --
        cde

will get this:

    <dl>
    <dt>a</dt>
    <dd><p>abc</p>
    </dd>
    <dt>b</dt>
    <dd><p>cde</p>
    </dd>
    </dl>

### Directly links

You can use:

    http://google.com

in text.

### Other text decorators

Par.md also adds some new text decorators, such as:

    ^text^          <sup>text<sup>
    ,,text,,        <sub>text<sub>
    ~~text~~        <span style="text-decoration: line-through">text</span>
    ***text***      <strong><em>text</em></strong>
    ___text___      <strong><em>text</em></strong>

### Bootstrap Tab Pages

Support tab pages, the format is:

    [[tabs(id=hello)]]:
        ```
        This is a test
        ```
    [[tabs(id=world)]]:
        * this is a list
        * test

### Bootstrap Alert

    [[alert(class=success)]]:
        Success
    [[alert(class=error)]]:
        Error
    [[alert(class=info)]]:
        Info
    [[alert]]:
        Message
    [[alert]]:
        #### Alert Title
        This is alert message
    [[alert(class=info,close)]]:
        This is an alert. It'll has a close button.

For bootstrap usage, you should invoke the code like this:

    from par.md import parseHtml
    from par.bootstrap_ext import blocks
    
    print parseHtml(text, template, block_callback=blocks)