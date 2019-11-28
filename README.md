# Perl Style Guide
The Perl Style Guide is based on [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html) with modifications and adaptations as necessary to make sense for the Perl language. The contents of this guide is a result of the many questions and discussions that a new developer team encounters when attempting to decide on a common format and style of coding.
Naturally a guide like this one won’t be able to cover all possible features of a language or all possible different styles of coding, always use your best own judgement when deciding on anything that isn’t stated in here.

# Code style
## Indentation
Indentation is done with 4 spaces. No tabs are allowed.

## Line length
Maximum line length is 80 characters.
Exception to this is
* Long URLs
* When importing packages with very long names

## Function calls
Built-in functions may be called without parentheses only if zero or one argument is required. Use parentheses if two or more arguments is required.
Functions that are not built-in should always be called with parentheses, even if no arguments is required.
The exception is getter functions for instance attributes.

No whitespace before the first opening parenthesis.

**OK:**
```perl
myfunc();
myfunc($a);
my $name = $person->name;
my @sorted_list = sort @list;
printf("Hello %n", $name);

foreach my $key (sort keys %hash) {
...
}
```

**Not OK:**
```perl
myfunc (); # Space before parentheses not allowed!
myfunc $a; # Must use parentheses for custom functions

my @split_words = split ' ', $string; # Must use parentheses for two or more arguments
```

## Shebang
All executable scripts should use the following shebang as the very first line in the script
```perl
#!/usr/bin/env perl
```

## Script file extensions
Executable scripts should not have a file extension. How the script is executed is determined by the shebang, making the file extension an unnecessary and ugly addition to the filename.

## Parentheses
Arguments within parentheses should immediately follow the opening `(`.
The closing `)` should come immediately after the last argument with no whitespace between.

Multiple arguments within parentheses are separated by a comma and a single space.

**OK:**
```perl
if ($a == 10) {
...
}
my $a = 2 * (5 + 10);
my @awesomeList = (1, 2, 3, 4, 5); # Elements properly separated with comma and space
```

**Not OK:**
```perl
if ( $a == 10 ) { # Space immediately after the opening ( and before the closing )
...
}

my @awesomeList  = (1,2,3,4,5); # No spaces between elements
```

## Return values
Functions should always return one type of value regardless of the situation.
Do not return status codes.

A function may return one of the following types
* `@array` or `$array`
* `%hash` or `$hash`
* `$scalar`
* Integer
* Object
* `undef`

Do not return `undef` to indicate empty results, return the empty array or hash in that case.
Always explicitly return from a function.

**OK:**
```perl
sub get_items {
    ...
    return wantarray ? @result : \@result; # context aware return types are OK
}

sub divide {
    my ($a, $b) = @_;
    if ($b == 0) {
        return undef; # Return undef to indicate error
    }
    return $a / $b;
}
```

**Not OK:**
```perl
sub get_items {
    ...
    if (@result) {
        return @result;
    }
    return undef; # Return the empty @result instead
}

sub get_items {
    ...
    if (@result > 1) {
        return @result;
    }
    return $result[0]; # Do not return mixed data types
}

sub get_items {
    ...
    @result; # Implicit returns is not OK
}
```

## Naming convention

| Type                         | Public             | Internal/Private    |
| ---------------------------- | ------------------ | ------------------- |
| Package/Module/Class         | Camel::CaseClass   | Camel::CaseClass    |
| Functions                    | lower_with_under() | _lower_with_under() |
| Global variables             | G_lower_with_under | _G_lower_with_under |
| Global constants             | CAPS_WITH_UNDER    | _CAPS_WITH_UNDER    |
| Instance variable/attributes | CamelCase          | _CamelCase          |
| Method names                 | CamelCase()        | _CamelCase()        |
| Function/method/parameters   | lower_with_under   |                     |
| Local variables              | lower_with_under   |                     |


## Code blocks
The opening curly braces `{` should be on the same line as the keyword it belongs to.
The closing curly braces `}` should always be on a separate line with the only exception being single statement functions.

**OK:**
```perl
sub do_something {
   ...
}

sub { do_something() }

sub get_instance { return My::Package->new() }
```

**Not OK:**
```perl
sub do_something
{
    ...
}

sub { do_something(); return 0 }
```

## Statements
All code should have one statement per line. One exception to this is when sending an anonymous function as an argument, as the content of that anonymous function technically is a separate statement.

**OK:**
```perl
my $a = some_function(sub { get_results(); });
```

**Not OK:**
```perl
print "Hello world"; return 0;
```

## Classes
Moose classes should be used whenever a package need to carry instance data (and therefore require instantiation). The only exception to this is when a subclass inherits from a base class that does not support using Moose.
All Moose classes should be immutable.

```perl
use Moose;

# Code goes here...

__PACKAGE__->meta->make_immutable();
no Moose;
```

## Function and Method comments
A function must have a description, unless it meets all of the following criteria:
* Not externally visible
* Very short (1-2 statements)
* Intention is clear

```perl
# delete_user
#
# Delete user with a given ID
#
#   Args:
#           id: id of user to be deleted
#
#   Returns:
#           1 on success, undef on failure
#
sub delete_user {
    my ($self, $id) = @_;
    ...
}
```

# Language rules
## Importing packages

Importing other packages into a module or script can be done using the `use` or `require` keywords depending on what time the package should be included (compile time or run time).

### Decision
Packages that are essential for the module or script, or packages that is known to be needed before execution should be imported at compile time with the `use` keyword.
Packages that may or may not be used by the module or script based on run time decisions should be imported using the `require` keyword, when they are needed.

## Global variables
While sometimes useful, global variables has the potential to change the behavior of a module or script in unexpected ways leading to confusion and bugs that are difficult to track down.
Global variables are by their very nature not thread safe.

### Decision
Avoid global variables whenever possible. Any module or script used in a multi-threaded application must never use global variables.
Global variables that are read/writable must use the `G_` prefix in the variable name.

```perl
$G_my_varible = 123;
```

Some examples where global variables are useful and acceptable:

* Default arguments for scripts
* Readonly constants (Does not require the G_ prefix. See naming conventions for details)

## Nested functions
Nested functions are subroutines inside another subroutine

```perl
sub first {
    sub second {
    }
}
```

### Pros
Nested functions does not expose functions externally.

### Cons
They most of the time break the single-responsibility principle for the outer function.

### Decision
Named functions should never be created within another function. Anonymous functions can be created inline when passed as an argument to a function call.
The body of Anonymous functions created this way should never exceed one or two lines of code. If more functionality is required, create a regular named function (not nested!) and call that function from within the Anonymous function.

**OK:**
```perl
sub someLargeFunction {
    # Couple of lines of code here...
}

sub first {
    some_function(sub{ someLargeFunction() }); # Inline anonymous function
}
```

**Not OK:**
```perl
sub first {
    sub second {
        ...
    }
    second();
}
```

## Dynamic function definition
Dynamic functions are named functions created at run time, which can be extremely powerful in certain situations. For example, `Moose` is using this feature to create setter/getter functions for class attributes.

### Pros
Dynamic functions can help to avoid code duplication for many functions that need the same functionality, such as setters/getters.

### Cons
They can be confusing and difficult to debug because the function name can’t be searched for within the code.

### Decision
Dynamic functions are allowed for extremely small use cases (such as setter/getter functions) and only if the interface to create such functions explicitly defines the entire name of the function.
Creating dynamic function by concatenating two or more strings (hard coded or otherwise) is not OK since it simply removes all possible traceability.

## Function reference
Perl supports storing references to functions, or function names as string, in variables that can later be used to call the referenced function.

### Pros
Allows for a more dynamic control flow as it makes it possible to decide what functions to call at run time.
Can make for cleaner code and avoids code duplication.

### Cons
They can be confusing and difficult to debug as it’s not always clear where in the code, or even if, a certain function is called.

### Decision
Function references are OK but should be used sparingly. Creating function references by concatenating two or more words (hard coded or otherwise) is not OK since it simply removes all possible traceability.

**OK:**
```perl
sub log {
    my ($level, $message) = @_;
    Log::Package->$level($message);
}

sub find {
    my ($self, $type, $filter) = @_;
    my %find_func_list = (
        "dogs" => \&find_dogs,
        "cats" => \&find_cats,
    );
    my $find_func = $find_func_list{$type};
    $self->$find_func($filter);
}
```

**Not OK:**
```perl
sub find {
    my ($self, $type, $filter) = @_;
    my $find_func = "find_".$type;
    $self->$find_func($filter);
}
```

## Postfix conditions
Postfix conditions are conditional expressions written behind/after a statement

```perl
print "This is true!" if $true;
print "It's cold!" unless $hot;
```

### Pros
Postfix conditions can make for cleaner and more readable code.

### Cons
When used to evaluate more than a single condition they can be difficult to read.

### Decision
Postfix conditions are fine when used on reasonably short one-liners and only evaluates a single condition.

**OK:**
```perl
print "This is true!" if $true; # Single boolean condition
print "That's a lot!" if $amount > 10; # Also a single condition
return unless user_is_logged_in();
```

**Not OK:**
```perl
print "This is true!" if $true and $temperature < 10; # Multiple conditions, use a regular if-statement instead.

return if some_check(
    name => $name,
    value => $value,
); # This isn't a one-liner. Use a regular if statement instead.
```

## unless keyword
`unless` is a perl built-in keyword that effectively means the same thing as `if not`.

### Pros
The `unless` keyword is closer to how statements are expressed in the english language.

### Cons
It can be confusing for developers not used to the keyword. `unless` will only evaluate to true if the condition is false.

### Decision
The unless keyword should be avoided whenever possible. The only time unless may be used is as a postfix condition. However care should be taken even in situations where `unless` is technically allowed, as it can easily cause confusion.

**OK:**
```perl
sub myFunc {
    ...
    return unless $something;
}

print "Hello" unless $greeting_disabled;
```

**Not OK:**
```perl
unless($not_logged_in) {
    redirect_to_login();
}

return $ref unless ref $ref eq 'ARRAY'; # While technically OK this is rather confusing for someone trying to understand this code.
```

## Function argument assignment
Perl populates the predefined `@_` variable with all arguments passed to a function and assigning variables is simply a case of reading/popping elements from that list.
This can be done in a number of ways and styles

```perl
sub myFunction {
   my $a = shift;
   my $b = shift;
}

sub myFunction {
   my ($a, $b) = @_;
}
```

### Decision
Assign all function arguments at the very top of the function block as a single statement/one-liner, even if only a single argument is used. All arguments used in a function must be properly assigned to a variable before used.

**OK:**
```perl
sub myFunction {
    my ($a) = @_;
}

sub myFunction {
    my ($a, $b) = @_; # This can easily be expanded with more arguments as requirements change over time.
}
```

**Not OK:**
```perl
sub myFunction {
    my $a = shift;
}

sub myFunction {
    my $a = shift;
    my $b = shift;
}

sub myFunction {
    # Some line of code
    my ($a, $b) = @_; # Argument assignment must be at the very top of a function body
}

sub myFunction {
    print $_[0]; # This says nothing about what is expected from this argument or what it is. Don't do this.
}
```

## Default argument values
Default argument values are not supported by perl in the traditional sense

```perl
sub myFunction($a = 10) { ... } # This isn't perl
```

However, the same effect can be achieved by checking if an argument is defined or not
```perl
sub myFunction {
   my ($a) = @_;
   $a //= 10;
}
```

### Pros
Default argument values makes function calls easier and enables the use of optional arguments.

### Cons
Default values can potentially cause a function to behave in an unexpected way if not properly documented.

### Decision
Default argument values are fine to use. They must however have their default values set immediately after assignment, at the top of the function body, so that it's easy for anyone looking at the code to see if an argument uses a default value or not.
Default argument values must never be conditional and should be documented in the [Function or Method comment](#function-and-method-comments).

**OK:**
```perl
sub myFunction {
    my ($a, $b) = @_;

    $a //= 10;
    $b //= $a; # It's OK to default to another argument
}
```

**Not OK:**
```perl
sub myFunction {
    my ($a) = @_;
    # Couple of lines of code...
    $a //= 10; # Not ok, this should be at the top of the function block
}

sub myFunction {
    my ($a) = @_;
    $a //= 10 if $someConditionHere; # Do not use conditionals for default argument values
}
```

## Implicit True/False evaluations
Perl evaluate certain values as false when in boolean context. The following is considered ‘false’
* 0 - Numerical zero
* undef - Undefined value
* ‘’ - Empty string
* ‘0’ - String that contains single zero(0) digit.

Everything else in perl is considered ‘true’.

### Pros
Conditions using booleans are easier to read and less error-prone.

### Cons
While it’s less error-prone overall, it does introduce the need to think extra carefully if it’s really `$foo == 0` or `not defined $foo` you want to test.

### Decision
Use the “implicit” true/false if at all possible, e.g. `if ($foo)` rather than `if ($foo == 1)`.
Sometimes it’s necessary however to use explicit conditions to avoid evaluating the wrong thing. For example `if ($foo)` will not only evaluate to true when `$foo == 1`, but also when `$foo == 2`, `$foo == -1`, etc. which may be undesirable.

**OK:**
```perl
if ($foo) {
...
}

if ($foo == 1) { # Not strictly checking for 'trueness', but a specific value instead
...
}
```

**Not OK:**
```perl
if (@myList > 0) {
...
}
```

## Warnings and Strict
Perl `warning` and `strict` help find potential errors in code that may technically be valid Perl code but not do what the developer intended with it.

### Pros
Finds certain errors such as typos, attempts to make arithmetic operations on undefined values, etc. much faster than human trial-and-error.

### Cons
May cause warnings even if the code behave as expected, also known as "false positive". This can be especially true in older legacy code.

### Decision
All modules and script should use `warning` and `strict`, either explicitly or by using the `Moose` module. Do **not** implicitly use `warning` and strict by relying on other modules to use them, the one and only exception being the `Moose` module. If there is a case where `warning` and/or `strict` need to be disabled they can be temporarily turned off. This must always be done inside a lexical scope and with proper comments why it’s needed.

**OK:**
```perl
package My::Package;

use warnings;
use strict;

package Another::Package;

use Moose;

{ # Start a new scope so that warnings and strict is disabled only temporarily
    no warnings;
    no strict;

    # Do naughty stuff here...
}
```

## Packages
Packages is a way to group logically similar code/functionality together into manageable modules. This enables and encourages code reuse which is an important part of developing an application.

### Pros
Enables and encourages code reuse. Makes it easier to maintain large code bases as functionality is logically divided and separated into modules.
Makes unit testing easier.

### Cons
Packages introduces extra overhead and code.

### Decision
Use packages (modules) extensively and whenever possible. Even smaller packages with a single method can still be very much suitable as a package.


## Predefined variables
Perl has a number of predefined, global, variables such as `$_` and `@_`.

### Pros
Predefined variables can make code more compact, especially suitable for one-liners.

### Cons
Since predefined variables are by default and almost always used as globals they also share the same limitations. Predefined variables are not very descriptive and their intentions can be lost quickly when used across multiple lines of code. The contents of a predefined variable can change unexpectedly if other functions also rely on them.

### Decision
Avoid using predefined variables whenever possible. Some built-in perl functions and 3rd party modules make use of the predefined variables, in such cases assign the contents of the variable to a new variable as soon as possible.
Do not ever work directly on predefined variables.

**OK:**
```perl
while (<>) {
    my $input = $_;
    # Work on $input....
}

sub myFunc {
    my ($argument) = @_;
    # Work on $argument
}
```

**Not OK:**
```perl
while (<>) {
    chomp($_); # Don’t work directly on predefined variables
    ...
}

sub myFunc {
    ...
    if ($_[0] == 1) { # Is this an argument, or something else?
        return 0;
    }
}
```

## Fat comma
Perl require the *fat comma* `=>` for key/value pairs within a hash. The fat comma can also replace a normal comma in almost any situation.

### Pros
Fat comma is easier to spot as a separator than a normal comma within the code.

### Cons
It’s difficult to clearly see the intentions when using fat comma instead of a normal comma. *“Are these key/value pairs or just elements in a list?”*

### Decision
Fat comma should be limited to key/value pairs in a hash and avoided in any other situation.

**OK:**
```perl
sub func {
    my (%args) = @_;
}
func(arg1 => 1, arg2 => 2);
```

**Not OK:**
```perl
my @list = ("one" => "two");
```

## Documentation (POD)
Perl uses the Plain Old Documentation markup language for writing documentation within a perl script or module.

### Pros
POD is the default documentation format for perl programs and supported natively by perl.

### Cons
POD is a simple language and lacks numerous features when it comes to formatting. The format takes up a lot of space and can potentially make the actual code harder to read.

### Decision
Use POD to document modules. The POD should be aimed to the consumer of the module and document publicly available methods and attributes, mainly focusing on input parameters, description of functionality and return values, not implementation.

Always put the POD at the very end of the file.

**OK:**
```perl
package My::Package;

# all package code...

=head1 ALL POD GOES HERE

=cut
```

**Not OK:**
```perl
package My::Package;

# package code...

=head1 SOME TITLE
some POD stuff
=cut

# more package code...
```

# Definitions within this document
* **Function/Subroutine** - A perl subroutine, defined using the `sub` keyword, that is **not** part of a class instance. A function most of the time also refer to a method but it depending on the context where they are used.

* **Method** - A perl subroutine, defined using the `sub` keyword, that is part of a class instance. Methods always take the instance object as the first argument (i.e. `$self`). Methods never refer to a non-instanced function or subroutine.

* **Package/Module** - A perl package, also commonly known as a module. Code namespaced using the `package` keyword.


# Other resources
* https://juerd.nl/site.plp/perlstyle
* https://perldoc.perl.org/perlstyle.html
