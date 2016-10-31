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

To isolate project dependencies and manage per-project versions of ruby itself,
it looks like I should use either:

- rbenv + bundler
- rvm

### `gem`

`gem` is the lower-level utility for installing a single library from a gem
repository. `gem` installs packages/artifacts called "Gems" which contain the
ruby library or program you wanted to fetch/install.

One nice thing about `gem` is that it supports installing multiple versions of
the same library. Then your project must select which versions of those
globally-installed libraries your project needs (only one version of a library
can be _loaded_ at a time).

### Bundler

Bundler installs easily with `gem install bundler`. You add project
dependencies to a `Gemfile` at the root of your project directory. Then
`bundle install` will fetch/install those dependencies. Bundler creates a
`Gemfile.lock` to pin all of your dependencies (and their dependencies).

You need to use `bundler exec <command>` to ensure the command gets run with
the correct, bundler-installed versions of libraries. This is because bundler
installs everything "globally". Bundler does not manage a per-project directory
where my dependencies are stored.

For example, you can see how bundler mainpulates the load path by comparing
the following:

```
# show the plain ruby load path
# in bash, be sure to use single quotes (or escape the dollar sign)
ruby -e 'puts $LOAD_PATH'
```

Compare that with the bundler-modified load path.

```
# show the load path, as modified by bundler
bundler exec ruby -e 'puts $LOAD_PATH'
```

If you have libraries installed with bundler, the bundler-modified load path
should list a bunch of paths to gem directories _first_ and then all the
default paths.

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

### Why is `require 'rubygems'` at the top of everyone's code?

tl;dr: Never do this!

This [overrides the built-in `require`](http://stackoverflow.com/a/2711857) to
additionally search the your installed gems.

Historically (before Ruby 1.9) you needed to require rubygems yourself to use
installed gems. However, ["Ruby 1.9 and newer ships with RubyGems built-in"](
http://guides.rubygems.org/rubygems-basics/). Ruby 1.9 was released in 2007, so
new code that uses rubygems should *never* `require 'rubygems'`.

On old versions of Ruby (<= 1.8) you should [still never `require 'rubygems'](
http://2ndscale.com/rtomayko/2009/require-rubygems-antipattern) in your code.
Instead, configure the environment to acheive the same thing. You can `ruby
-rubygems main.rb` or `export RUBYOPT="rubygems"`. And for executables
installed by `gem`, rubygems installs wrappers that require rubygems for you.

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

Arrays
------

Arrays in Ruby are 0-indexed and look like:

```
x = [1, 2, 3]

# indexing
puts x[0]

# get the length - these are all the same
puts x.length, x.count, x.size
```

### Iterating arrays

The [Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide#syntax) says
to use iterators instead of `for` loops. Why?

- `for` loops use iterators under the cover anyway
- `for` loops don't have their own scopes! Variables defined in a for loop
are accessible after the for loop completes

Use the `each` method to iterate over an array. `each` accepts a "block" which
can be written in two different ways:

```
x.each do |val|
    puts val
end
```

A shorter way to write this is:

```
x.each { |val| puts val }
```

Blocks and Yield
----------------

### Blocks

A block in Ruby is like a closure or anonymous function. They accept arguments,
run code, and have a return value. They introduce a new, isolated scope so that
variables defined within the block don't leak outside of the block.

They are commonly used with iterators like:

```
# a block defined with do..end syntax
x.each do |val|
    puts val
end

# the same block defined with curly braces
x.each { |val| puts val }
```

### Yield

If a method uses `yield`, then the method expects to be used with a block.
When `yield` gets executed, the current method's execution is suspended and a
different block of code is run. After that block of code has run, execution
resumes in our function.

For example,

```
def do_thing
    puts 'before yield'
    yield 1, 'a', 2
    puts 'after yield'
end

# The following prints
#   before yield
#   1
#   a
#   2
#   after yield
do_thing do |a, b, c|
    puts a, b, c
end
```

This shows that we can run a method, yield to a block (with arguments!), and
the resume the original method.

*Aside* A block is a way to define a
[`Proc`](http://ruby-doc.org/core-2.3.1/Proc.html) object. `Proc` objects
can be defined with either `do..end` or curly brace syntax.

Symbols (what is that colon doing there?)
-----------------------------------------

*Guideline* - NEVER mix strings and symbols. They do not compare with each
other like you want and they hash differently.
*Guideline* - NEVER use dynamically-generated symbols. Avoid writing code
that requires you (or anyone) to convert between strings and symbols.

Lots of ruby code uses things like `:name` instead of a string like `"name"`.
Well `:name` is a symbol (_not_ a string). Symbols are kind of like strings,
except that:

- all occurrences of `:name` have the same reference (i.e. same object id)
- symbols are immutable! (strings are mutable)
- symbols are compared using their object ids (rather than char by char, as
with strings).
- symbols are hashed using their object id

If you are familiar with [string interning](
https://en.wikipedia.org/wiki/String_interning), symbols are like interned
strings (but be careful, symbols and strings aren't interchangeable everywhere)

This is a [good article](
http://www.randomhacks.net/2007/01/20/13-ways-of-looking-at-a-ruby-symbol/)
with examples of these differences and more.

Here is an example that shows two matching symbols are the same object, while
two matching strings may not be.

```
puts :hello.object_id   # 899868
puts :hello.object_id   # 899868
puts "hello".object_id  # 70183008528740
puts "hello".object_id  # 70183008528680
```

### Symbol Naming Rules

- symbols cannot start with a number!
- symbols can have spaces!

```
# a symbol with spaces
puts :'a b c'

### Symbols vs strings

Strings and symbols are NOT equal.

```
puts "hello" == "hello"  # true
puts :hello == :hello    # true
puts :hello == "hello"   # false
puts "hello" == :hello   # false
```

Symbols are indexable but are immutable.

```
y = :hello
puts y[0]    # prints 'h'
y[0] = "j"   # <-- ERROR - "undefined method `[]=' for :hello:Symbol (NoMethodError)"
```

You can get the symbol from a name using `intern`.

```
x = "hello"
x == :hello   # false

y = x.intern
y == :hello   # true
```

I've seen people say symbols are good for hashing, but DON'T mix symbols and
strings in a hash. Symbols hash on their object id, while strings hash on
their contents, effectively.

```
map = {}
map[:hello] = :hello
map["hello"] = "hello"
puts map  # prints {:hello=>:hello, "hello"=>"hello"}
```

Hashes/Maps
-----------

In Ruby, a hash is a mapping of key-value pairs (or "dictionary" or
"associative array"). They are defined with curly braces. Values are set
or fetched using square brackets.

Hashes can be initialized at definition-time:

```
map1 = {}                                # an empty map
map2 = {"a" => 1, "b" => 2}    # a map initialized with two key-value pairs
```

Then you can get and set things from the map

```
puts map1["a"]
map1["a"] = 5
```

### Hashes and symbols

The [Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide) says the
following about symbols and maps:

> Prefer symbols instead of strings as hash keys.
> Avoid the use of mutable objects as hash keys.

(Keep in mind, Ruby strings are mutable)

There are two ways to use symbols as keys in a map. This is the "rocket" way
(the fat arrows are sometimes called "hash rockets")

```
# :hello is a symbol that maps to a number
map = {:hello => 1}
puts map                # prints {:hello=>1}
```

This is the "hash literal" way, in which you put the colon _after_ the symbol
name. This is the _same_ map as above.

```
# :hello
map = {hello: 1}
puts map                # prints {:hello=>1}
```

The [The Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide)
recommends this style when your hash keys are symbols:

> Use the Ruby 1.9 hash literal syntax when your hash keys are symbols.
> Don't mix the Ruby 1.9 hash syntax with hash rockets in the same hash
> literal. When you've got keys that are not symbols stick to the hash rockets
> syntax.

I'm in favor of using only hash rockets everywhere. In particular, since symbol
names cannot start with a number you cannot use the hash literal syntax with
"integer symbols" like you might expect:

```
# symbols cannot start with a number
map = {:1 => 2}  # <-- SYNTAX ERROR - `:1` is an invalid symbol
map = {1: 2}     # <-- SYNTAX ERROR - `:1` is an invalid symbol
```
