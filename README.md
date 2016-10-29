Overview
--------

These are some notes I took while learning ruby. They are not a guide or a
tutorial.

Installation
------------

On Windows:

- Use [rubyinstaller](http://rubyinstaller.org/)
- I needed to [fix an ssl cert issue with `gem`](http://guides.rubygems.org/ssl-certificate-update/)


Managing Dependencies
---------------------

Things we can manage on a per-project basis:

- The version of ruby we are using
- The versions of ruby libs we have installed

Some of the available tools are:

- [gem](https://rubygems.org/): fetch/install versions of a library (similar to `pip` in python)
- [rvm](https://rvm.io/): manage per-project versions of both ruby and libraries (with "gemsets")
- [rbenv](https://github.com/rbenv/rbenv): manage per-project versions of ruby
- [bundler](http://bundler.io/) manage per-project versions of libraries

It looks like `gem` is the lower-level utility for installing a single library
from a gem repository. `gem` installs packages/artifacts called "Gems" which
contain the ruby library or program you wanted to fetch/install.

To isolate project dependencies and manage per-project versions of ruby itself,
it looks like I should use either:

- rbenv + bundler
- rvm

### Bundler

Bundler installs easily with `gem install bundler`. You add project
dependencies to a `Gemfile` at the root of your project directory. Then
`bundle install` will fetch/install those dependencies. Bundler creates a
`Gemfile.lock` to pin all of your dependencies (and their dependencies?)

### rvm and rbenv on Windows...

I wasn't able to get rbenv or rvm working on Windows 7 (with msys/mingw, but
maybe cygwin would work, or Windows 10 with the Ubuntu subsystem).

rvm uses a bash script to install and it had issues trying to untar an archive
containing symlinks. There is a way to enable (or simulate) symlinks in
msys/mingw, but tar still had issues when I did this.

rbenv requires compiling a C version of a `realpath` function (this is an
optimization). `rbenv` could run, but failed to successfully install any
versions.

Modules/Imports
---------------

### `require` vs `include`

`require` is what you should use to "import" classes or whatever from another
ruby file. If you require a file from the same directory, you can use
`require_relative`. Both of these do _not_ include the extension.

`include` has nothing to do with files. It is an OOP mechanism to support
mixins.

Classes
-------

Classes look like

```
class Thing
    def initialize
        @x = 1
    end

    def do_thing
        @x += 1
    end
end

thing = Thing.new
```

You use the `class` keyword to define the class, and use the `def` keyword to
define the methods.

The `initialize` method is used to setup a new instance. Use the `new` method
to create new instances. `Thing.new` creates a new instance of `Thing`. `new`
will invoke the `initialize` method to setup that instance.

Use `@` to define variables in `intialize`. For example, `@x = 1` creates an
instance variable (named `x`). Use `@x` to access that variable within any
method on that class.

### Method Visibility

(more at: https://en.wikibooks.org/wiki/Ruby_Programming/Syntax/Classes#Declaring_Visibility)

Ruby methods are public by default, but can be made protected or private.

A protected method is accessible by _any_ instance of the class (including
subclasses). You can access protected methods of a class as long as you are
within that class.

Private methods are private to an instance. No other instance can access the
private method of another instance. Private methods of a superclass _are_
accessible to instances of the subclass.

A good example of the protected/private difference is an equality method:

```
# If `get_x` is a private method this will not work! We'll fail to access
# `other.get_x`.
#
# If `get_x` is protected, then `other` must be an instance of `Foo`. Our
# instance can access protected methods on instances of the same class.
class Foo
    ...
    def equal_to?(other)
        @x == other.get_x
    end

    protected
    def get_x
        @x
    end
end
```

##### `public`, `protected`, `private` keywords

The scope of these keywords is a bit funky. The keyword applies to all methods
defined in the class after that keyword.

```
# Funky method visibility scope
class Foo
    # methods are public by default
    def public_method
    end

    private
    def private_method
    end

    # This is NOT public. It is private because it follows the private keyword.
    # You could explicitly mark the method as public or protected, or move it
    # before the `private` keyword.
    def another_private_method
    end
end
```

There is another way to mark methods private or protected that avoids this
scoping pitfall, which is to declare visibility after defining the method by
passing the methods as arguments to the `private` and `protected` keywords
(I've seen people say that `private` and `protected` are actually methods)

```
# Better method visibilty declarations
class Foo
    def public_method; end
    def protected_method; end
    def private_method; end
    def another_private_method; end

    # declare private/protected methods here. these must come after the method
    # is defined, or you will get a name error
    protected :protected_method
    private :private_method, :another_private_method

    # this method is public! when using protected/private keywords with method
    # argument
    def another_public_method; end
end
```

### Instance variable visibility

Instance variables are private, always (and remember that "private" things in
Ruby are accessible in the subclass).

How do you make a variable readable/writable? You define getters and setters
in a way that makes the methods act like an attribute.

```
class Foo
    def initialize(val)
        @val = val
    end

    # getter
    def val
        @val
    end

    # setter
    def val=(val)
        @val = val
    end
end

a = Foo.new(1)
puts a.val
a.val = 2
puts a.val
```

##### Accessors

"Accessors" are a shorter way to define getters and setters. There are three
keywords we can use:

- `attr_reader`: create a getter method
- `attr_writer`: create a setter method
- `attr_accessor`: create both a getter and a setter

The example above can therefore be shortened to the following.

```
class Foo
    def initialize(val)
        @val = val
    end

    attr_reader :val
    attr_writer :val
end
```

Since we want _both_ a reader and a writer, we can use `attr_accessor` instead.

```
class Foo
    def initialize(val)
        @val = val
    end

    attr_accessor :val
end
```

### Method naming conventions

Ruby allows `!`, `=`, and `?` characters in method names:

- A method ending in a question mark (?) should return a boolean
- A method ending in an exclamation/bang (!) is potentially dangerous
- A method ending in an equal sign (=) is a setter method. For example, if
a class wants to expose an attribute `value` as gettable and settable,
the class should define `def value` and `def value=(value)` as the getters and
setters respectively.

A good reference is
[The Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide) (which I
obviously haven't read because I have been using four spaces for indentation
instead of two).
