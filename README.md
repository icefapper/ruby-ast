#Ruby AST - an endeavor to make an ESTree equivalent for Ruby language
This project aims to standardize an AST format for Ruby in the same way [ESTree](https://github.com/estree/estree) project is doing so for ECMAScript. It is still an a very early state, and any contributions or suggestion are most welcome.



```js 
interface Node { // A syntax node
     start: number;
      end: number;
     loc: SourceLocation;
}
```

```js
interface Location {
     line: number;
     column: number;
}
```

```js
interface SourceLocation {
     source: string; // program's code
     start: Location;
      end: Location;
}
```

```js
interface Program {
   body: [StatementExpression|BEGIN];

}
```

```js
interface BEGIN { 
   body: [StatementExpression|BEGIN]

}
```
A 'BEGIN' expression:
```ruby
BEGIN {
  def l() return end
  alias l _l
}
```

```js
interface Body {
    block: [StatementExpression]
    handler: EnsureClause | null 
    finalizer: [StatementExpression]
    alternate: [StatementExpression]
}
```
A method or block or 'begin' body:
```ruby
begin #<body>
              #<block>
   print "this is the body's block"
   raise "some exception"
              #</block>
rescue exception #handler
   print "Caught #{exception}"

else #alternate
   print "No exception"

ensure #finalizer
   print "Exception or not, this is going to execute"

end #</body>
```
       
```js
interface AliasExpression <: StatementExpression {
    old: Identifier
    new: Identifier
}
```
An 'alias' expression:
```ruby
alias _new _old
```

```js
interface UndefExpression <: StatementExpression {
     arguments: [Identifier]
}
```
An 'undef' expression; think of it as reversing a 'def':
```ruby
def l() 12 end
print l # prints '12'

undef l
print l # will throw, because 'l' is no undefined
```

```js 
interface ModiefiedStatementExpression <: StatementExpression {
    modifier: 'if'|'unless'|'while'|'until';
    test: Expression;
}
```
A top level expression followed by 'if', 'unless', 'while', or 'until':
```ruby
alias _new _old if false and _old < 12
def l() 12 end unless defined? :l
class L; def l() 12 end end while false
print "12" until true
l = nil or l = 12 if false
```

```js
interface END {
      body: [StatementExpression];
}
```
An 'END' top-level expression:
```ruby
END {
   print "this runs on exit"
}

print "this runs on start"
```

```js
interface AssignmentExpression {
    operator: string;
    left: [MultipleAssignmentList|Expression]
    right: [MultipleAssignmentList|Expression] 
}
```
An assignment expression; a list is allowed on its left as well as on its right if it is in top-level:
```ruby
a, b = 12, 'l' #is the same as (a, b) = (12, 'l')
print a, b = 12, 'l' #is the same as print( a, (b=12), 'l' )
```

```js
interface Command {
   block: CommandBlock | (null if arguments != null)
   arguments: Arguments | (null if block != null)
   name: Identifier
}
```
A command with non-null argument list and/or non-null block:
```ruby
print "l" do 12 end
print "l" { 12 }
print "l", 12 { 12 }
print "l", 12 do 12 end

#please note the '(' must _immediately_ follow print; otherwise is an argument list only if the construct it contains
# is not a comma-separated list
print("l", 12) do 12 end
print("l", 12) { 12 }
print do 12 end
print { 12 } # please note the {12} thing is not a dictionary; it is a block
print "l"
print "l", 12
print( "l", 12) 
```ruby

```js
interface IdCommand <: Identifier, Command {
   block: null
   arguments: null
   name: Identifier
}
```
Either a command with neither an argument list nor a block, or a simple identifier:
```ruby
def l() 12 end
l #this is an IdCommand resolving to a command at run time

l = 12
l # this is an IdCommand resolving to a simple identifier at run time
```   

```js
interface MethodCall <: Command {
     name: MethodIdentifier
     object: Expression;
}
```
A method being called on an object:
```ruby
a.b 12 # object: a, name: b
a.- 12 # object: a, name: -
(12 * 12).-@ # object: 12 * 12, name: -@
(a - 12).l 12 { 12 } # object: a - 12, name: l
0.~ # object: 0, name: ~
```
 
```js
interface SuperCall <: Command {
    type: 'SuperCall';
}
```
A super call; the syntax is identical to that of an ordinary command:
```ruby
class L < SomeOtherClass
   def e()
    
      super "l" do 12 end
      super do 12 end
   end
end
```
   
```js
interface YieldCall <: Command {
     type: 'YieldCall';
}
```
A yield call; the syntax is identical to that of an ordinary command:
```ruby
print { yield "l" do 12 end }
```

```js
interface VoidExpression {}
```
A void expression; it is mostly a euphemism for a 'statement' since Everything Is An Expression (TM) in ruby; the only reason things like 'return', 'break', 'continue', or 'next' are still called expressions in ruby is that, they can be used in a conditional:
```ruby
while true do
  l = false
  l ? continue : break
end
```
Please note the conditional above is itself a void expression; actually, if the consequent and/or alternate of a conditional is a void expression, the conditional is void as a whole; one caveat to beware with void 'expression's is that, they are not combinable(nor compatible) with non-void expressions (a 'non-void expression' is, tautologically, an expression with a value)
```ruby
false ? 12 : 'l' # will parse; 12 and 'l' are 'value' expressions
false ? next: 12 # won't parse; 12 and next are incompatible
false ? next: break # will parse; next and break are compatible
```

furthermore, they can not be used as sub-expressions, because they have no inherent value (i.e., they are not even nil; hence the name 'void'):
```ruby
12 - 'l' # will parse; 12 and 'l' are 'value' expressions
12 - (next) # won't parse because 'next' has no value
```

Even more, any 'block' expression like 'if', 'begin', etc., will be a void expression if its last expression is a void expression:
```ruby
if false then 12; break else "l"; c end # this is a void expression
begin 12; break end # this is a void too
```

Finally, multi-component block expressions (like 'if' with an 'else', or 'begin' with a 'rescue') will turn into void expressions if any of their composing blocks is a void expression (please note this rule applies recursively):
```ruby
  #below, the 'if' expression will not be void, because component block #1 and component block #2 have inherent value
  if false then
  #component block number 1: consequent; its value is that of its last expression, i.e., "l"
      print "l"
      "l"
  else
  #component block number 2: alternate; its value is that of its last expresion, i.e., 12
      print 12
      12
  end #result: either component block #1 or component block number #2

  while false do  
     #below, the 'if' expression will be void because the values of its component #1 and component #2 are incompatible, i.e.,
     # one is a 'value' expression, and the other one is not

     if false then
            #component block number 1: consequent; evaluates to a void expression (in this case, 'break') 
            l = "l"
            break
      else
            #component block number 2:
            l = 12
      end #result: either the value of `l=12` (i.e., 12) or `break` (i.e., no value, not even a nil) 
  end
```         

```js
interface ReturnExpression <: VoidExpression {
    arguments: Arguments;
}
```
A return expression:
```ruby
def l()
   return 12, *[12, 12]
   
end

def w(l)
  while true do
     ( l <= 0 ) ? return : 
     ( l -= 1 ) ? (return l)
  end
end
```  

```js
interface BreakExpression <: VoidExpression {
   arguments: Arguments;
}
```
A break expression:
```ruby
while false;  break 12, *[12] end
while false;  break end
```

```js
MultipleAssignmentListPattern {
    head: [AssignmentPatternElement]|null
    splat: AssignmentPatternElement|null
    tail: [AssignmentPatternElement]|null
}
```
The left hand side of a top-level assignment expression:
```ruby
a, (b[0], (c, *e)), (*), = [12, [12, [12]], 12, 12, 12, 12, 12, 12, 12, 12, 12]
# a: 12, b[0]: 12, c: 12, e: []
```

```js
MultipleAssignmentList {
    head: [AssignmentElement]|null
    splat: AssignmentElement|null
    tail: [AssignmentElement]|null
}
```
The right hand side of a top-level assignment expression:
```ruby
l = 12, *[12], *[12]
```

```js
interface NextExpression <: VoidExpression {
   arguments: Arguments
}
```
A next expression.
```ruby
while false; next 12, *[12] end
while false; false ? next : return end
```
 

       
