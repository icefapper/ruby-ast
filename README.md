#Ruby AST - an endeavor to make an ESTree equivalent for Ruby language
This project aims to standardize an AST format for Ruby in the same way [ESTree](https://github.com/estree/estree) project is doing so for ECMAScript. It is still an a very early state, and any contributions or suggestion are most welcome.



```js 
interface Node {
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

```js
interface Body {
    block: [StatementExpression]
    handler: EnsureClause | null 
    finalizer: [StatementExpression]
    alternate: [StatementExpression]
}
```

```js
interface AliasExpression <: StatementExpression {
    old: Identifier
    new: Identifier
}
```

```js
interface UndefExpression <: StatementExpression {
     arguments: [Identifier]
}
```

```js 
interface ModiefiedStatementExpression <: StatementExpression {
    modifier: string;
    test: Expression;
}
```

```js
interface END {
      body: [StatementExpression];
}
```

```js
interface AssignmentExpression {
    operator: string;
    left: [MultipleAssignmentList|Expression]
    right: [MultipleAssignmentList|Expression] 
}
```

```js
interface Command {
   block: CommandBlock | null
   arguments: Arguments | null
   name: Identifier
}
```

```js
interface MethodCall <: Command {
     name: MethodIdentifier
     object: Expression;
}
```

```js
interface SuperCall <: Command {
    type: 'SuperCall';
}
```

```js
interface YieldCall <: Command {
     type: 'YieldCall';
}
```

```js
interface VoidExpression {}

```js
interface ReturnExpression <: VoidExpression {
    arguments: Arguments;
}
```

```js
interface BreakExpression <: VoidExpression {
   arguments: Arguments;
}
```

```js
MultipleAssignmentList {
    head: [AssignmentElement]|null
    splat: AssignmentElement|null
    tail: [AssignmentElement]|null
}
```

```js
interface NextExpression <: VoidExpression {
   arguments: Arguments
}
```
 

       
