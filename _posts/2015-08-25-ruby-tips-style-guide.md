---
layout: post
title: "Ruby Tips: Style Guide"
tags:
feature_image:
---

{% highlight ruby %}
"Programs are meant to be read by humans and only incidentally for computers to execute." - H. Abelson and G. Sussman
{% endhighlight %}

When it comes to Ruby, and many other programming languages, the computer does not care if your code looks pretty while it executes your program. However, programs are created and maintained by humans, so coding style is super important.<!--more-->
One day, someone other than you is going to have to read your code. Even when you <em>are</em> working alone. Imagine working on a project, writing some sloppy code, and coming back to it a few months later. I guarantee you will waste hours trying to decipher your smelly, cryptic code.

By writing code in a consistent manner and using some common best practices, you can make the world a better place for yourself and your fellow developers.

This post is a sampling of best practices based on the following ruby community resources:

* <a href="https://github.com/bbatsov/ruby-style-guide">ruby style guide</a>
* <a href="https://github.com/bbatsov/rails-style-guide">rails style guide</a>

Above all else, be consistent, make your code readable, and if you are working on a project with a team, whatever style guidelines are approved should be followed.

### Tab Spacing
Use soft tabs (two spaces) instead of hard tabs (four spaces) when indenting a line of code.

{% highlight ruby %}
# GOOD
def method
  "indented two spaces."
end

# BAD
def method
    "indented too many spaces."
end

{% endhighlight %}

### Naming
Classes should be CamelCase, methods, variables, and constants (all caps) should be snake_case.
{% highlight ruby %}
SOME_CONSTANT = []

class SomeClass
  # class content
end

def some_method
  # method content
end

some_local_variable = nil
{% endhighlight %}

### Strings
Prefer string interpolation and string formatting instead of string concatenation:
{% highlight ruby %}
# GOOD
email_with_name = "#{user.name} <#{user.email}>"

# BAD
email_with_name = user.name + ' <' + user.email + '>'
{% endhighlight %}

Adopt a consistent string literal quoting style.
{% highlight ruby %}
# Be consistent with whichever you choose
name = "Tyler"
name = 'Tyler'
{% endhighlight %}

### Class Organization
When it comes to organizing your methods, constants, attr_accessors, and so on within a class, there is some debate as to whether class or instance methods should come first. Generally speaking, it doesn't matter too much but things should be organized in a logical manner and you should be consistent. I recommend following the <a href="https://github.com/bbatsov/ruby-style-guide">style guide</a> which uses the following order:

1. extend/include modules
2. CONSTANTS
3. attr_accessors
4. public class methods
5. initialize method
6. public instance methods
7. private/protected methods

{% highlight ruby %}
class Person
  # extend and include go first
  extend SomeModule
  include AnotherModule

  # constants are next
  SOME_CONSTANT = 20

  # afterwards we have attribute macros
  attr_reader :name

  # followed by other macros (if any)
  validates :name

  # public class methods are next in line
  def self.some_method
  end

  # initialization goes between class methods and other instance methods
  def initialize
  end

  # followed by other public instance methods
  def some_method
  end

  # protected and private methods are grouped near the end
  protected

  def some_protected_method
  end

  private

  def some_private_method
  end
end
{% endhighlight %}

### Comments
{% highlight ruby %}
"Good code is its own best documentation. As you're about to add a comment, ask yourself, How can I improve the code so that this comment isn't needed? Improve the code and then document it to make it even clearer." - Steve McConnell
{% endhighlight %}
Use comments sparingly. If you find yourself commenting a ton, you should think about writing better code and it will speak for itself.
