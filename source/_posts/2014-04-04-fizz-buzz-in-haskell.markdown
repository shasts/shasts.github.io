---
layout: post
title: "Fizz Buzz in Haskell - Power of filter and map"
comments: false
categories: 
---
<p>Off late, I have been learning Haskell programming language. Suddenly the thought of how <a href="http://en.wikipedia.org/wiki/Fizz_buzz">Fizz Buzz</a> would look like in Haskell strike my mind.</p>
<p> After 5 minutes, a working version using guards and map was ready.</p>
{% codeblock lang:haskell%}
fizzBuzz :: Int -> [Char]
fizzBuzz n 
	| mod n 15 == 0 = "FizzBuzz"
	| mod n 3  == 0 = "Fizz"
	| mod n 5 == 0 = "Buzz"
	| otherwise = show n
{% endcodeblock %}
<p>Then simply mapping the function to a range of first 100 was enough.</p>>
{% codeblock lang:haskell%}
map fizzBuzz [1..100]
{% endcodeblock %}
<p>The beauty is the ability to chain multiple functions and get a working program. More or less the similar is possible in imperative languages as well. But, imagine a scenario where one need to know how many numbers are there which are divisible by both 3 and 5. Same can be obtained from the FizzBuzz function, just by chaining two more functions.</p>
<p>This may be a bad example of reusing function, but one gets the point. Just like the *NIX shell's ability to chain multiple programs and get the output quickly and elegantly.</p>
{% codeblock lang:haskell%}
length (filter (=="FizzBuzz") (map fizzBuzz [1..100]))
{% endcodeblock %}