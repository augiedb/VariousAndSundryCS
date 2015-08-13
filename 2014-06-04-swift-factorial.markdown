---
layout: post
title: "Swift Factorial"
date: 2014-06-04 08:41:23 -0400
comments: true
categories: 
---
When learning a new language, it's a good time to go back to the computer programming classics and see what you can do with them.

The first thing I went for was Factorial. Here's my first script:

		func factorial(var value: Int, var result: Int = 1) -> Int {

		    if (value == 0) {
		            return result
		    }
		    
		    result *= value
		    value--
		    
		    return factorial(value, result: result)
		}

		> factorial(3)
		6

There's some stuff to cover in here.  Take the function declaration for starters:

		func factorial(var value: Int, var result: Int = 1) -> Int {

Coming from Perl and Elixir and Ruby, it feels crowded.  It is, however, useful. It defines a lot of stuff ahead of time for the compiler and IDE.  Xcode can sense it a mile away now when you're using the function and auto-completes stuff for you. (Unlike Objective-C, you don't need a separate header file now.) The compiler can test stuff along the way, too, like making sure you're sending in the correct type, number of parameters, etc.

So, you start with `func` as the function keyword.  Each parameter has a name and type specified.  I'm using the 'var' in front of their names because I'm mutating their state inside the function. I need to define them as new variables.  Without that ```var```, Swift assumes these are constants.  An error message will tell you that these are 'let' values, which is Swift-ese for "constants", which is to say immutable.

With the second parameter, I thought I'd make it easy on the user and define a default value. ```result``` is the accumulator.  Its state will change along the way as you multiply the numbers together. Since this is planned to be a recursive function, we can't ignore it completely.  It has to be there.  So it gets both the `var` tag and the `= 1` part at the end to define a default value.

When you call the function this way, you only need to supply one value, like so:

		> factorial(3)
		6

_(The Swift playground displays the results off on the right sidebar of the window, not below it.  But I don't want to insert a bunch of screen grabs here, so I'll stick to a more commmand line-looking style to show things off like this.  The Xcode REPL is pretty cool, though, and worth using.)_

You can be more explicit, of course, and get the same answer:

		> factorial(3, 1)
		6

And you can be flat out silly:

		> factorial(3, 0)
		0

		> factorial(3, 2)
		12


Getting back to the code, the ```if``` statement is pretty obvious.  I didn't even look at the Swift book to program this part.  It's fairly standard across multiple modern languages.  No semi-colons. Use curly braces. Double equals for testing equality.  This is the base case.  Once we've counted back to 0, we return the value.

After that, we mutate our variables. Again, standard coding stuff.   Double minus for decrement and star-equals to multiply a variable by some other value.

Finally we have the recursive call, where we need to specify the accumulator now.  **If your function provides a default value, you have to name that parameter.** Xcode is pretty good about pointing that out, but it's worth emphasizing here.

This works, but it's not elegant. First, there's mutable state in there. That's bad.  Second, the user should _never_ have to type in the default value or even have the option to.  This is Factorial.  The accumulator should always start at 1.  I'm also not a big fan of naming the parameters if you don't have to or in very simple examples like this one.  (For the rest of the Apple frameworks, they're very handy.)  So let's hide all that stuff separately.  **In Swift, functions are defined by their name and arity.**  (_Arity_ means the number of parameters.)  This is just like in Elixir.  This makes me happy and makes for even cleaner code:


		func factorial(value: Int) -> Int {
		        return factorial(value, 1)
		}

		func factorial(value: Int, result: Int) -> Int {
		    
		    if (value == 0) {
		        return result
		    }
		    
		    return factorial(value-1, result * value)
		}

		> factorial(3)
		6

		> factorial(3, 1)
		6


So the user has the option of putting in a value for the accumulator, but they don't have to. I haven't read all the documentation yet, so I'm not sure if there's an option to make a function private.  If there is, I'd hide the factorial/2 function in there, and only allow the user to use the one parameter version of the function (factorial/1), which would then make the call to the two parameter version.

The function is now also immutable. No variable state is ever changed. This is enforced by creating the function with parameters that are constants.  That's a default and unseen `let` before each in there.  There's no assignment operator (`=`) anywhere to be seen inside the code, either. That will make testing much easier as well as making my code more portable. 

The more I look at the code, the more it bothers me that there's an ```if``` statement inside the function at all.  But since there's no pattern matching for the parameters in Swift, we have to live with it.  This is the extra code I'd love to write to get rid of that ```if``` statement all together:

		func factorial(0, result: Int) -> Int {
		    return result
		}

Place that between the two functions and it'll match the base case and take care of returning the final value for you, if your language defaults to the first-matching function it finds, top down. Elixir does that. I like it.