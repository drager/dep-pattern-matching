# Pattern matching

## Contact information

Authors:
* [Jesper Håkansson](https://github.com/drager) [jesper@jesperh.se](mailto:jesper@jesperh.se)
* [Rasmus Eneman](https://github.com/Pajn) [rasmus@eneman.eu](mailto:rasmus@eneman.eu)

Stakeholders:
* [Github issue](https://github.com/dart-lang/sdk/issues/2949)

[Github repository](https://github.com/drager/dep-pattern-matching)

## Summary
A match expression is a more powerful version of a switch statement while still beeing simpler to reason about due to no fall-though or labels.

```dart
var x = 5;

match (x) {
  1 => print("x is one");
  2 => print("x is two");
  3 => print("x is three");
  4 => print("x is four");
  5 => print("x is five");
  _ => print("something else");
}
```

## Motivation

Pattern matching is declarative way to describe multiple possible states of a value.

The switch statement is unwieldy because of fall through that works different in Dart compared to other languages. It also looks pretty different from the rest of the language because blocks start with a colon and ends by a return or a break statement. The match expression solves this by not having fall through and not having a heritage of fall through from other languages.

A common use of a switch statement is to either assign or return a value in every case. By having an expression you can assign or return the whole match expression which showes the intentation more clearly.

Unlike a chain of if/else if statements, the match expression enforces exhaustivenss. This makes sure that you do not forget a value. If none of the patterns match, the match expression will throw a MatchError.

## Examples

### Values and default state
Identifiers or variabels defines a "default" state which matches all values. The variable will be declared and assigned the value of the object thats matching.

```dart
fib(int n) => match (n) {
  0 => 0;
  1 => 1;
  n => fib(n-1) + fib(n-2);
};
```

```dart
fib(int n) {
  if (n == 0 || n == 1) return n;
  return fib(n-1) + fib(n-2);
}
```

### Type tests

```dart
match (90) {
  x is int => print('x is an integer');  // As 90 is an int this case would match and the other would be ignored
  x is num => print('x is another type of number');
  90 => print('x is equal to 90');
};
```

```dart
var _ = 90;

if (_ is int) {
  int x = _;
  print('x is an integer');
} else if (_ is num) {
  num x = _;
  print('x is another type of number');
} else if (_ == 90) {
  var x = _;
  print('x is equal to 90');
}
```

### Enums
```dart
enum Fruit {
  apple, banana
}

var x = Fruit.banana;

var name = match (x) {
  Fruit.apple  => 'apple';
  Fruit.banana => 'banana';
};

// Todays syntax equivalent
var name;
switch (x) {
  case Fruit.apple:
    name = 'apple';
    break;
  case Fruit.banana:
    name = 'banana';
    break;
}
```

### List deconstructuring
Allows pattern matching on indices in a list as well as declares the properties in the scope.
Can be used to match on multiple variables in the same match expression.

```dart
match ([2, 5]) {
  [2, b] => print('x is 2, b is $b');
  [a, 5] => print('x is $a, b is 5');
  [a, b is int] => print('a is $a, b is the integer $b');
  [a, b] => print('a is $a, b is $b');
  [a, b, ...] => print('a is $a, b is $b but the list has more than two elements');
  _ => print('the list has fewer than 2 elements');
};
```

```dart
var x = [2, 5];

if (x is List && x.length == 2 && x[0] == 2) {
  var b = x[1];
  print('x is 2, b is $b');
} else if (x is List && x.length == 2 && x[1] == 5) {
  var a = x[0];
  print('x is $a, b is 5');
} else if (x is List && x.length == 2 && x[1] is int) {
  var a = x[0];
  int b = x[1];
  print('a is $a, b is the integer $b');
} else if (x is List && x.length == 2) {
  var a = x[0];
  var b = x[1];
  print('a is $a, b is $b');
} else if (x is List && x.length >= 2) {
  var a = x[0];
  var b = x[1];
  print('a is $a, b is $b but the list has more than two elements');
} else {
  print('the list has fewer than 2 elements');
}
```

### Object deconstructing
Allows specification of patterns on properties as well as declares the properties in the scope.

```dart
match (new Point(2, 5)) {
  Point {x: 2, y} => print('x is 2, y is $y');
  Point {x, y: 5} => print('x is $x, y is 5');
  Point {x, y} => print('x is $x, y is $y);
};
```

```dart
if (x is Point && x.x == 2) {
  var x = x.x;
  var y = x.y;
  print('x is 2, y is $y');
} else if (x is Point && x.y == 5) {
  var _ = x;
  var x = _.x;
  print('x is $x, y is 5');
} else if (x is Point) {
  var _ = x;
  var x = _.x;
  var y = _.y;
  print('x is $x, y is $y);
}
```

### Alternatives
Alternatives can be used where a fall through had been used in a switch statement.
```dart
var message = match (new DateTime.now().day) {
  1 | 2 | 3 | 4 | 5 => 'This is a weekday';
  6 | 7 => 'This is a weekend day';
  _ => 'Not a legal day';
}
```

```dart
var message;

switch (new DateTime.now().day) {
  case 1:
  case 2:
  case 3:
  case 4:
  case 5:
    message = 'This is a weekday';
    break;
  case 6:
  case 7:
    message = 'This is a weekend day';
    break;
  default:
    message = 'Not a legal day';
}
```

### Ranges
The above case can be even nicer with ranges
```dart
var message = match (new DateTime.now().day) {
  1...5 => 'This is a weekday';
  6 | 7 => 'This is a weekend day';
  _ => 'Not a legal day';
}
```

### Pattern guards
Pattern guards allows to use expressions to specify rules that is not possible using just declarative patterns.

```dart
match (10) {
  x if (x % 2 == 0) => print('x is even');
  x if (x % 2 == 1) => print('x is odd');
};
```

```dart
var x = 10;

if (x % 2 == 0) {
  print('x is even');
} else if (x % 2 == 1) {
  print('x is odd');
}
```

## Proposal

The match expression tries to match the patterns in order from first specified to last. For arms with alternatives the patterns are tried from left to right.
If a pattern matches that do have a pattern guard, the guard expression is evaluated. If the produced object evaluates to false using the same semantics as an if statement, the pattern is considered to not have matched. When a matching pattern is found the arm's expression is evaluated and the produced object is returned from the match expression.
If no matching pattern is found the match expression throws a MatchError. It is a static warning if the match expression is not exhaustive. A match expression if considered exhastive if any of the following condifitions hold:
- the match expression does have a default/identifier pattern
- the static type of the expression being matched is an enumerated typed and all of its elements are covered
- the static type of the expression being matched is a List and destructured list pattern allowing all lists exists (`[...]`)
- a destructured object pattern that allows all objects of the static type of the expression being matched exists.
- a type test pattern that allows objects of the static type of the expression being matched exists.

Non qualified identifiers in patterns is implicitly declared as var, or in a type test pattern as the type beeing tested, and then assigned the value of the object beeing matched against. This is true also for identifiers in list or object destructors.
It is a compile-time error if the same identifier binding occurs multiple times in the same match clause.

Values in pattern is compared to the object using the == operator on the object beeing matched.

Type tests are performed in the same way as with the **is** operator exept that the left-hand side can only be a non qualified identifier.

List destructuring patterns first checks that the object beeing tested is a List to avoid NoSuchMethod errors. Then it validate the length to be the same or, if `...` is specified, at least the same as the number of element as beeing destructed. This is to avoid a RangeError if the List is smaller than the number of destructured elements. After those saftey checks the specified elements is read from the list and matched against their patterns.

Object destructuring patterns first does a type check to avoid NoSuchMethod errors. Then the specified properties are read and matched against their patterns. It is a static warning if getting the specified property of an object the specified type would issue a static warning.

## Alternatives

As if and switch statements already exists in the language this alternatives description will focus on different pattern matching implementations.

### Extractor objects
Scala has the concept of extractor objects where using apply and unapply methods objects can be destructured and their properties matched. This is instead solved in this DEP by using object destructuring which doesn't need implementation of additional methods and allows the reader of the pattern to see which properties are matched against.

### Different syntax
During discussion on the [mailing list]() different syntax propassals have come up.

#### case keyword
Case could be used as a keyword in each match clause so that the syntax is  
**case** pattern (‘|’ pattern)* patternGuard? ‘=>’ expressionStatement

While you may think of your solution as a perfect snowflake, there are other pretty snowflakes out there. Describe alternate solutions that cover the same space as this proposal. Compare them to the proposal and each other and explain why they were rejected.

What advantages do these alternatives have that the chosen one lacks? What does the selected proposal offer to outweigh them? Being rigorous here shows that you've done your due diligence and fully explored both the problem and the potential solution space. We want to be confident that we have not just *a* solution to the problem, but the *best* one.

## Implications and limitations

As an expression, match will behave differently than if or switch statements. One suprising behaviour might be
that a semicolon is required to end the statement which is not needed with if or switch which are full statements on there own.

While the match expression doesn't have a big inpact on the future evolution of the languages, range and destructuring patterns might. If Dart ever gets similar features for other parts than patterns the syntax and behaviour of them will be partly decided by the patterns to avoid inconsitencies.

A good language is a cohesive whole. Even a small, well-defined proposal still has to hang together with the rest of the Dart platform.

- How does this feature interact with the other language features&mdash;in both good and bad ways&mdash;?
- If we adopt this feature what will it prevent us from doing now? What do we sacrifice for this?
- How does it affect the future evolution of the language?

## Deliverables

### Language specification changes

#### New syntax
    rangePattern:
      numericLiteral ‘...’ numericLiteral
    ;

    destructuredListPattern:
      ‘[’ (pattern (‘, ’ pattern)* ‘, ’? )? ‘...’? ‘]’
    ;
    
    destructuredObjectPatternEntry:
      identifier (‘:’ pattern)?
    ;
    
    destructuredObjectPattern:
      type ‘{’ (destructuredObjectPatternEntry (‘, ’ destructuredObjectPatternEntry)* ‘,’?)? ‘}’
    ;
    
    pattern:
      nullLiteral |
      numericLiteral |
      booleanLiteral |
      stringLiteral |
      symbolLiteral |
      typeTest |
      rangePattern |
      destructuredListPattern |
      destructuredObjectPattern |
      identifier |
      qualified
    ;
    
    patternGuard:
      **if** ‘(’ expression ‘)’
    ;
    
    matchClause:
      pattern (‘|’ pattern)* patternGuard? ‘=>’ expressionStatement
    ;
    
    matchExpression:
      **match** ‘(’ expression ‘)’ ‘{’ matchClause* ‘}’
    ;

#### Changes
In *20.1.1 Reserved Words* add **match**

### A working implementation

We have started working on analyzer support and a transpiler. Both are still work in progress though.
The analyzer should have pretty good support except for the object destructuring pattern. The fork can currently be found in https://github.com/Pajn/dart-analyzer

The transpiler have parsing support by using the forked analyzer and can transpile some of the cases but have no support for writing out the transpiled files yet. It can be found in https://github.com/drager/match-expression-transpiler

### Tests

Write tests that could be run under an implementation to determine if it implements your proposal correctly. Make sure to test both success and failure cases. Be thorough. Programming languages have combinatorial input, so defining a comprehensive test suite is difficult.

Imagine an adversary is going to write a malicious implementation of your proposal that passes its tests but otherwise does things so horrible it will cause the language team to run screaming from your proposal. Don't let the adversary win.

## Patents rights

TC52, the Ecma technical committee working on evolving the open [Dart standard][], operates under a royalty-free patent policy, [RFPP][] (PDF). This means if the proposal graduates to being sent to TC52, you will have to sign the Ecma TC52 [external contributer form][] and submit it to Ecma.

[tex]: http://www.latex-project.org/
[language spec]: https://www.dartlang.org/docs/spec/
[dart standard]: http://www.ecma-international.org/publications/standards/Ecma-408.htm
[rfpp]: http://www.ecma-international.org/memento/TC52%20policy/Ecma%20Experimental%20TC52%20Royalty-Free%20Patent%20Policy.pdf
[external contributer form]: http://www.ecma-international.org/memento/TC52%20policy/Contribution%20form%20to%20TC52%20Royalty%20Free%20Task%20Group%20as%20a%20non-member.pdf
