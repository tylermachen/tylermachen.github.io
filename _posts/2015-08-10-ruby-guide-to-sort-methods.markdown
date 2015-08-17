---
layout: post
title: "Ruby Guide: Sort Methods"
date: 2015-08-10 05:14:28 -0400
feature-img:
---
## The Basics
---

### .sort

By default, Ruby's `.sort` method will sort any array of numbers or strings in ascending (low to high) order. When sorting numbers, this is self explanatory. When sorting strings, Ruby considers letters that come earlier in the alphabet to be less than letters that appear later.

<!--more-->

### Numbers
Given the following array of numbers:

{% highlight ruby %}
[1,5,9,2,4]
{% endhighlight %}

Ruby will sort as follows:

{% highlight ruby linenos %}
array = [1,5,9,2,4]
array.sort
# => [1,2,4,5,9]
{% endhighlight %}

### Strings
Given the following array of strings:

{% highlight ruby %}
['dog', 'rabbit', 'aardvark', 'turtle', 'cat']
{% endhighlight %}

Ruby will sort as follows:

{% highlight ruby linenos %}
array = ['dog', 'rabbit', 'aardvark', 'turtle', 'cat']
array.sort
# => ['aardvark', 'cat', 'dog', 'rabbit', 'turtle']
{% endhighlight %}


## How does it work?
---

In contrast to Ruby's `.each` method that passes a *single* array item to a block per iteration, Ruby's `.sort` method passes a **pair** of array items and compares them to each other **one iteration at a time**.

Upon comparing two array values, Ruby will determine which value, if any, should come before the other based on a value (0, 1, or -1) that it receives from a block, and make swaps as necessary until all pairs have been iterated
over.

* see below for more about how `.sort` uses the values 0, 1, and -1 to determine the sort order of items in an array.

### Numbers
{% highlight ruby linenos %}
array = [1,5,9,2,4]

array.sort do |a, b|
  if a == b
    0
  elsif a > b
    1
  elsif a < b
    -1
  end
end
# => [1,2,4,5,9]
{% endhighlight %}

* The first iteration will send our first pair of array values `1` and `5` to our
  block as the values for our block variables `a` and `b` respectively. Now our `.sort` method is ready to compare our values...

  * If `a` and `b` are equal, the block will return `0`, and `.sort` will leave them in their current places within our array.

  * If `a` is less than `b` and belongs before it, the block will return `-1` and `.sort`
  will, once again, leave them in their current locations in our array because `a` is already before `b`.

  * If `a` is greater than `b` and belongs after it, the block will return `1` and `.sort`
  will swap the locations of `a` and `b`.

* In our case: `1` is less than `5` and already comes before it in our array so `.sort` is going to return a `0` from our block and move on to the next iteration/pair of array values.

* On our next iteration `.sort` will shift over one index in our array and compare the next set of values. For us, this means we will be comparing values `5` and `9`. You may notice that `5` was the second value in our first pair. This is how `.sort` moves through the array. If we had swapped `1` and `5` after our first iteration, then our new comparison pair would have been `1` and `9`.

### Strings
{% highlight ruby linenos %}
array = ['dog', 'rabbit', 'aardvark', 'turtle', 'cat']

array.sort do |a, b|
  if a == b
    0
  elsif a > b
    1
  elsif a < b
    -1
  end
end
# => ['aardvark', 'cat', 'dog', 'rabbit', 'turtle']
{% endhighlight %}

* The process is nearly the same for strings as it is for the set of numbers used above, however, there are some small differences worth addressing.

  * Our first pair is `dog` and `rabbit`. `.sort` will start with the first letter of each of these words and compare them. Remember, by default, Ruby considers letters that come earlier in the alphabet to be less than those that appear later.

  * In our case, since the `d` in `dog` appears earlier in the alphabet than the                        `r`
  in `rabbit`, our block will return a `0` to our `.sort` method and nothing will be
  swapped.

  * We then move on to our next pair, `rabbit` and `aardvark`. Since `r` comes after `a` in the alphabet, our block returns a `1` and our values are swapped.

  * The next pair would then be `rabbit` and `turtle` and so on.

* Up until now, we have been comparing words using their first letter. But, what happens if two words share the same first letter? Lucky for us, `.sort` easily handles this scenario as well!

  * Let's say we need to sort two words `aarb` and `aaab`. `.sort` will start with the first letter and then...since they are equal, will move on to the second letter. The second letters are equal as well, so `.sort` will move on to the third letter. Now, we have something to compare! Since `r` comes after `a` in the alphabet, our block will return a `1` and will swap our values so that `aaab` comes before `aarb` in our array.

### Shortcuts
{% highlight ruby linenos %}
array.sort do |a, b|
  a <=> b
end
{% endhighlight %}

Using the `spaceship` aka `combined-comparison operator` is the same thing as writing all of those `if/then` statements we looked over in our previous examples and much, much shorter.

### One more thing...
By default, one does not need to call a block on an array for regular alphabetical or numerical sorting in ascending order. One can simply type `array.sort` as in our basic examples at the start of this article. I wrote the `if/else` statements in order to illustrate what `.sort` does under the hood.


## The Intermediate Stuff
---

### Descending Order
As you now know, `.sort` sorts array items in **ascending** order by default. But, what if we want to sort our array in **descending** order?
We can do the following...

{% highlight ruby linenos %}
array = [1,2,4,5,9]

array.sort do |a, b|
  b <=> a
end
# => [9,5,4,2,1]
{% endhighlight %}

We use the `<=>` spaceship operator and simply switch the order of our block variables. Instead of `a <=> b`, we reverse them to `b <=> a`. This will sort our array in descending order.

### String.length
As long as we can generate numbers from something, `.sort` will know how to compare and order them.

{% highlight ruby linenos %}
array = ['dog', 'rabbit', 'aardvark', 'turtle', 'cat']

array.sort do |a, b|
  a.length <=> b.length
end
# => ["dog", "cat", "rabbit", "turtle", "aardvark"]
{% endhighlight %}

In the case of `dog` vs `cat` where both lengths are equal, `dog` ended up first in the array simply due to the order by which `.sort` travels (left to right). Unlike when comparing letters in the alphabet, when we compare on something such as `word.length`, `.sort` is not going to compare using `.length` **and** `alphabetical order`.

In this case, since we are comparing on `.length`. Our block passes a `0` because the `.length` of `dog` and `cat` are equal. Therefore, the array items are skipped and left in their current place. If `cat` was listed first in our array, it would have ended up first, once sorting was complete, just like `dog`.


## The Advanced Stuff
---

### .sort_by
Sometimes, we may need to sort an array by something other than the defaults offered by `.sort`.

`.sort_by` orders an array based on a set of keys returned from the block.

### Custom Alphabet

#### Objective
Sort an array of strings alphabetically based on the crazy alphabet below:

{% highlight ruby %}
CRAZY_ALPHABET = "scftxmwrgbdeaklvqnzphyouji"
{% endhighlight %}

#### Example
The array below:
{% highlight ruby %}
["this is crazy", "never again", "why", "afternoon", "start"]
{% endhighlight %}

Should become:

{% highlight ruby %}
["start", "this is crazy", "why", "afternoon", "never again"]
{% endhighlight %}

#### Solution
{% highlight ruby linenos %}
array = ["this is crazy", "never again", "why", "afternoon", "start"]

array.sort_by do |word|
  word.split('').map do |letter|
    CRAZY_ALPHABET.index(letter)
  end
end
# => ["start", "this is crazy", "why", "afternoon", "never again"]
{% endhighlight %}

### What's happening here?
The **most important** thing to remember is that Ruby's `.sort/.sort_by` methods *only* care about receiving `0, 1, and -1` values from the block. There is no magic here. Our sort methods still only understand that smaller numbers come before larger numbers and letters earlier in the alphabet are less than those that appear later in the alphabet.

Essentially, we can sort by whatever we want as long as we can come up with a way to get the block to compare our array value pairs and return `0, 1, or -1` to the `.sort_by` method.

In our case, outlined above, we are going to need to assign a numerical value to denote the order of the letters in our words based on the `CRAZY_ALPHABET`.

we first call `.sort_by` on our array and pass our first array value, `"this is crazy"` to the block. We then need to `.split` our word into it's individual characters and call `.map` because the `.map` method always returns an array which is what `.sort_by` needs to sort our original array of words.

Now that we have an array of letters for our first word, our `.map` method is going to search through our `CRAZY_ALPHABET` for a letter that matches the one we passed into the `letter` block variable for the current iteration. Once it finds a match, it will start building a new array with the index from the `CRAZY_ALPHABET` where the match occurred.

  * In our case, `letter == "t"` which is the 3rd index (starting from `0`) in the `CRAZY_ALPHABET` so after the first iteration we have an array of `[3]`

  * On the second iteration, `letter == "h"` which is the 20th index in the             `CRAZY_ALPHABET` so after the second iteration we have an array of `[3,20]`

  * This process continues until we have the indices of where our current word's letters fall within our custom alphabet.

{% highlight ruby %}
"this is crazy" == [3,20,25,0,25,0,1,7,12,18,21]
"never again" == [17,11,15,11,7,12,8,12,25,17]
"why" == [6,20,21]
"afternoon" == [12,2,3,11,7,17,22,22,17]
"start" == [0,3,12,7,3]
{% endhighlight %}

`.map` goes through each of our words and returns an array of indices upon which `.sort_by` can use to decide which word comes first according to our custom alphabet.

### Final Result
A beautiful, sorted array:

{% highlight ruby %}
["start", "this is crazy", "why", "afternoon", "never again"]
{% endhighlight %}

<iframe src="//giphy.com/embed/2WxWfiavndgcM" width="480" height="271" frameBorder="0" style="max-width: 100%;" class="giphy-embed" webkitAllowFullScreen mozallowfullscreen allowFullScreen></iframe>

---
Resources:
<a href="http://ruby-doc.org/core-2.2.2/Enumerable.html#method-i-sort" target="_blank">Ruby Documentation - Sort</a>
