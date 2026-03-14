# Language Internals

## Type juggling
Type juggling in PHP happens when the language automatically converts one data type to another depending on context — especially during comparisons or arithmetic operations.

PHP is loosely typed, so this can sometimes lead to surprising behavior.
```php
<?php

var_dump("10" == 10);     // true
var_dump("10" === 10);    // false
```

## Strict types (declare(strict_types=1))
```php
<?php

declare(strict_types=1);

function add(int $a, int $b) {
    return $a + $b;
}
// add("1", 2); // Throws TypeError
?>
```

## References
PHP references are a mechanism that allows different variables to access the same underlying data content. They are aliases for a shared memory location, not pointers to memory addresses like in C/C++.

Key Concepts
- **Aliases**: When you use a reference, the variable names are entries in the symbol table, and the reference allows multiple entries to point to the same value in memory.
- **Not Pointers**: You cannot perform pointer arithmetic with PHP references, nor are they actual memory addresses you can directly manipulate.
- **& Symbol**: The ampersand (&) operator is used to establish a reference.

```php
$a = 10;
$b = &$a; // $b is a reference to $a
$b = 20; // $a also becomes 20
echo $a; // Output: 20
```

```php
function add_five(&$value) { // Note the & symbol
    $value += 5;
}
$num = 10;
add_five($num);
echo $num; // Output: 15
```

## Symbol table
In PHP, a symbol table is an internal mechanism used by the PHP engine to store and manage information about all the declared identifiers (variables, functions, classes, etc.) in a program. It is essentially a hash map that links variable names (the keys) to their corresponding data (the values, known as zvals internally).

## Arrays (associative vs indexed)
In PHP, an array is an ordered map that associates values to keys and can store multiple values of different data types in a single variable. PHP arrays are highly flexible and can function as lists, hash tables, dictionaries, stacks, or queues. 

- Indexed Arrays
```php
<?php
$fruits = array("Apple", "Banana", "Cherry");
// or using the shorthand syntax (PHP 5.4+)
$fruits = ["Apple", "Banana", "Cherry"];
echo $fruits[0]; // Outputs: Apple
?>
```
- Associative Arrays: Use named keys (strings or integers) that you define, making the data more meaningful.
```php
<?php
$age = array("Peter" => "35", "Ben" => "37", "Joe" => "43");
// or using the shorthand syntax
$age = ["Peter" => "35", "Ben" => "37", "Joe" => "43"];
echo $age["Peter"]; // Outputs: 35
?>
```

## Closures & anonymous functions
A PHP closure is an anonymous function that can access variables from the scope in which it was created, even after that scope has finished executing. Closures are implemented as instances of the built-in [Closure]

- Anonymous: Closures do not have a specified name and are often assigned to variables or passed as arguments to other functions, such as callback functions for array_map or array_filter.

- Scope Access (use keyword): Unlike regular functions that are isolated from the parent scope, closures can inherit variables from the parent (lexical) scope using the use language construct.

- Variable Binding: By default, variables passed via use are passed by value (a copy is made). To modify the original variable from within the closure, it must be passed by reference using an ampersand (&)

```php

<?php
function createGreeter($who) {
    // The inner function is a closure
    // It "uses" the $who variable from the parent scope
    return function() use ($who) {
        echo "Hello $who";
    };
}

$greeter = createGreeter("World");
$greeter(); // Output: Hello World
?>
```

Arrow Functions (PHP 7.4+)
PHP 7.4 introduced a shorter, more concise syntax for closures called arrow functions, using the fn keyword. They implicitly capture variables from the parent scope by value, without needing the use keyword.
```php
<?php
$modifier = 5;
$numbers = [1, 2, 3];

// Short closure using fn
$result = array_map(fn($x) => $x * $modifier, $numbers);

print_r($result); // Output: Array ( [0] => 5 [1] => 10 [2] => 15 )
?>
```