# yeanpypa
YEt ANother PYthon PArser (generator)

Yeanpypa is a framework written in [Python](http:://www.python.org) and heavily
inspired by tools like [pyparsing](http://pyparsing.sourceforge.net/) and
[Boost.Spirit](http://www.boost.org/libs/spirit/index.html). It can be used to 
contruct recursive-descent parsers directly in Python in a nearly "natural" way, 
meaning that the necessary Python source code looks much like the original EBNF that
defined the grammar.

## Getting & installing yeanpypa

As yeanpypa is written in Python, its source code can be used without compiling. Just
fetch it from GitHub like this:
   
`$ git clone https://github.com/DerNamenlose/yeanpypa`

The file _yeanpypa.py_ contains everything that's necessary. Just copy this file either
into your project or into the appropriate directory of your Python installation (something
like _/usr/lib/python2.7/_ under Linux. See your distribution's documentation for details).

## Using yeanpypa

First of all: construct an [EBNF](http://en.wikipedia.org/wiki/EBNF) grammar for the language 
you'd like to parse. A short example for parsing floating point numbers in C:

```python
digit                 ::= "1" | "2" | "3" | "4" | "5" | "6" | "7" | "9" | "0" ;
number                ::= digit+ ;
floating point number ::= [ number ], ".", number ;
```

This example defines a digits as one of the set from 0 to 9, a number as at least one digit and 
a floating point number as an optional number, followed by a dot, followed by a number.
        
_Note:_ This definition isn't totally correct as it would allow constructions like "0002.00". 
For simplicities sake, we'll go with it.

### Constructing rule objects from an EBNF

In order to construct a yeanpypa-grammar from the EBNF, we write:
```python
from yeanpypa import *

digit     = Literal('1') | Literal('2') | Literal('3') | Literal('4') | \
            Literal('5') | Literal('6') | Literal('7') | Literal('8') | \
            Literal('9') | Literal('0')
number    = Word(digit)
float_num = Optional(number) + Literal('.') + number
```

_Note:_ In order to save typing, yeanpypa already provides a set of abstractions. The 
whole `digit` thing for example could be left out, as it is already provided
by yeanpypa.
        
The resulting `float_num` is a complex parser object that can be used to parse a 
floating point number like this:

```python
result = parse(float_num, '123.123')
if result.full():
    print result.getTokens()
else:
    print 'The parser did not consume all input.'
```

This will print the following: `['123', '.', '123']` The parser validated the input and created a list of token
according to the grammar specification. As the validation was successfull, it returned a list of tokens. In this case the token list is flat. In case of more complex grammars it is going to be a tree represented by (at least by default. We'll see later on how this behaviour can be modified using semantic actions) lists within lists.

### Ignoring rule results

Sometimes we want to ignore parts of the parse result. In the example in order to use the result, we can ignore the dot, as it does not tell us anything apart from the fact that we saw a floating point number (which we know because of the validation anyway). That's where `hide()` comes into play. The `hide()` method of a rule (the basic building block of a grammar) tells the parser to ignore any token created by the rule. We change the grammar like this:
        
```python
number    = Word(digit)
float_num = Optional(number) + Literal('.').hide() + number
```

_Note:_ We removed the `digit` declaration and rather use the abstraction provided by
yeanpypa to make the examples shorter and - hopefully - more readable.

Note the `hide()`-call at the `Literal(...)`-rule. This instructs the
parser to ignore the token created by that rule (i.e. the '.') and not create any output.

Using this parser yields the following output: `['123', '123']` We have successfully eliminated 
the superfluous dot token from the output.

### Transforming results

As we're parsing numbers, we would like to see the token as actual numbers instead of strings representing
numbers. Normally tokens are represented as strings (i.e. the substring that was matched by the given
rule) Yeanpypa provides the tools to transform the strings while matching using a //semantic action//:

```python
number    = Word(digit).setAction(lambda x: int(x[0]))
float_num = Optional(number) + Literal('.').hide() + number
```

Semantic actions are functions taking a single parameter as input (the list of tokens generated by this
rule) and return a transformed version of this input. We have attached a semantic action to the `number`-rule, 
which transforms its input from a string into an integer. The action _MUST_ return a value which is
used as the rule's output. This may be the original input list (in case the action merely outputs some
debug information or generates some external data structure) or it may be a transformed token (list) as 
in the example given.

Using this version of the parser yields the following output: `[123, 123]`
As you can see, the result token list now contains two integers instead of string representing them.

_Note:_ Attaching an action to a rule where `hide()` was called causes the action to be executed, but the 
output to be thrown away. Keep that in mind if you intend to mix these two facilities.

_Note:_ When nesting rules be aware of the following: if a semantic action for a sub-rule was called and
the matching of the parent rule fails at some later point (e.g. in case of the floating point example there
would be no part after the point), then the sub-rule will _NOT_ be notified of the failure. If you do
any changes to external data structures you will have to keep track of the changes and undo them manually
on failure. It is much more convenient to refrain from having side effects in the sematic actions and rather
resort to purely functional approach.

## Further documentation

For details on how to yeanpypa works and what is available see the [reference documentation](http://www.bitbucket.org/namenlos/yeanpypa/wiki/html/index.html).
Of course in case of doubts have a look at the source code. If you find any errors or have any wishes I'm happy to hear of you.
