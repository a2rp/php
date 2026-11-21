# Constructor property promotion - Q&A

These questions review promoted properties, constructor behavior, defaults, readonly promotion, and syntax restrictions. Each question is followed immediately by its answer and explanation.

## Multiple-choice questions

### 1. Which PHP version introduced constructor property promotion?

- A. PHP 7.4
- B. PHP 8.0
- C. PHP 8.1
- D. PHP 8.2

**Answer: B.** Constructor property promotion was introduced in PHP 8.0.

### 2. Which declaration promotes a constructor parameter into a property?

- A. <code>public function __construct(string $name)</code>
- B. <code>public function __construct(public string $name)</code>
- C. <code>public function name(public string $name)</code>
- D. <code>public string function __construct($name)</code>

**Answer: B.** A property modifier such as <code>public</code> makes the constructor parameter promoted.

### 3. What does promotion do for a parameter?

- A. Creates a class constant
- B. Creates a property and initializes it from the constructor argument
- C. Makes the method static
- D. Converts the argument into a global variable

**Answer: B.** Promotion is shorthand for declaring a matching property and assigning the constructor argument to it.

### 4. Does this parameter create a property?

~~~php
public function __construct(string $email)
~~~

- A. Yes, every constructor parameter is promoted
- B. No, it is an ordinary parameter
- C. Only if the constructor is private
- D. Only if the class is readonly

**Answer: B.** A parameter without a property modifier remains an ordinary parameter and does not create a property.

### 5. Which property visibility results from this declaration?

~~~php
public function __construct(private string $token)
~~~

- A. Public
- B. Protected
- C. Private
- D. Static

**Answer: C.** The modifier on the promoted parameter becomes the generated property's visibility.

### 6. When is a promoted property initialized?

- A. Before the constructor is called
- B. At the start of the constructor body
- C. The first time the property is read
- D. Only when the constructor returns

**Answer: B.** PHP assigns promoted arguments to their properties before executing the written statements in the constructor body.

### 7. Where does a default value on a promoted parameter apply?

- A. Only to the constructor parameter
- B. Only to a class constant
- C. Only to a property declaration default
- D. To every instance of the class globally

**Answer: A.** The default is a constructor parameter default. If the argument is omitted, the resulting parameter value is assigned to the promoted property during construction.

### 8. Which declaration correctly permits a missing value to become <code>null</code>?

- A. <code>public string $nickname = null</code>
- B. <code>public ?string $nickname = null</code>
- C. <code>public null $nickname</code>
- D. <code>public readonly $nickname = null</code>

**Answer: B.** The type must explicitly allow null. A null default does not make a non-nullable type nullable.

### 9. Which version supports readonly promoted properties?

- A. PHP 8.0
- B. PHP 8.1
- C. PHP 8.2
- D. PHP 8.3

**Answer: B.** Readonly properties were introduced in PHP 8.1, so readonly promotion requires PHP 8.1 or later.

### 10. What happens if a readonly promoted property is reassigned in the constructor body?

- A. The value is silently ignored
- B. It becomes mutable
- C. PHP rejects the second assignment
- D. The parameter default is used instead

**Answer: C.** Promotion initializes the property before the constructor body. A readonly property cannot be assigned again.

### 11. Can a promoted parameter appear in an ordinary method?

- A. Yes, in any method
- B. No, promotion is for constructors
- C. Only in static methods
- D. Only in private methods

**Answer: B.** Promoted parameters are supported in non-abstract constructors, not ordinary methods.

### 12. Can an abstract constructor use property promotion?

- A. Yes
- B. No
- C. Only when it has no parameters
- D. Only in PHP 8.6

**Answer: B.** Promotion generates property assignments in a constructor body. An abstract constructor has no body in which to perform them.

### 13. Can an interface constructor declare promoted parameters?

- A. Yes, interfaces create the properties
- B. No, an interface has no constructor implementation body
- C. Only for public properties
- D. Only when implementing a readonly class

**Answer: B.** Interfaces cannot provide the constructor body required for promotion.

### 14. Why can a promoted property not use the <code>callable</code> type?

- A. Callable values cannot be passed to constructors
- B. PHP properties do not support the callable type
- C. Callable is a reserved class name
- D. Promotion only supports integer values

**Answer: B.** Promoted parameters also declare properties, and PHP properties cannot use <code>callable</code> as their type.

### 15. Can a variadic constructor parameter be promoted?

- A. Yes, into an array property automatically
- B. No
- C. Only if it is private
- D. Only if it has a default value

**Answer: B.** Variadic parameters cannot be promoted because the parameter's individual argument type and collected array property type would differ.

### 16. What happens if the class already declares a property with the promoted name?

- A. The promoted property silently replaces it
- B. PHP creates two properties with that name
- C. PHP reports a redeclaration error
- D. The explicit property becomes a constant

**Answer: C.** A promoted parameter declares a property, so a same-named property cannot also be declared in the class.

### 17. What does <code>var</code> do in a promoted parameter?

- A. It is supported as an older visibility keyword
- B. It promotes the parameter to a public property
- C. It is not supported for promotion
- D. It makes the property readonly

**Answer: C.** Use a visibility modifier such as <code>public</code>, <code>protected</code>, or <code>private</code>; <code>var</code> is not supported.

### 18. What does this constructor declare?

~~~php
public function __construct(public int $id, string $source)
~~~

- A. Two promoted properties
- B. One promoted property and one ordinary parameter
- C. Two ordinary parameters
- D. One property and one class constant

**Answer: B.** <code>$id</code> is promoted because it has visibility. <code>$source</code> remains a normal parameter.

### 19. Where can a readonly modifier be used with promotion?

- A. Only on a class constant
- B. On a promoted property, starting in PHP 8.1
- C. On any parameter in PHP 8.0
- D. Only on a static property

**Answer: B.** Starting with PHP 8.1, readonly properties can be promoted from constructor parameters.

### 20. What is the safest reason to use promotion?

- A. To promote every constructor argument
- B. To reduce repeated declarations when the input maps directly to stored state
- C. To avoid validating constructor input
- D. To expose every property publicly

**Answer: B.** Promotion is a concise form for straightforward property initialization. It does not remove the need to choose visibility or validate input.

## Short-answer questions

### 21. What does a promoted property's type come from?

**Answer:** From the type declaration on the constructor parameter.

**Explanation:** Promotion uses the parameter's type for the generated property as well as the constructor argument.

### 22. Does promotion require every constructor parameter to become a property?

**Answer:** No.

**Explanation:** Use a visibility modifier for parameters that should be promoted. Keep temporary values, transformed inputs, and parent-constructor arguments as ordinary parameters.

### 23. Can a constructor mix promoted and ordinary parameters?

**Answer:** Yes.

**Explanation:** Each parameter is promoted independently based on its declaration. This is useful when some values map directly to properties and others need processing.

### 24. Why might a parameter be ordinary even if it supplies a property value?

**Answer:** The constructor may need to normalize or validate it before storing it, or store it under a different property name.

**Explanation:** An ordinary parameter gives the constructor control over what value becomes object state.

### 25. What is the difference between a promoted property default and a property default?

**Answer:** The default is applied to the constructor parameter, not declared as a property default.

**Explanation:** If the caller omits that argument, the constructor still receives the default and promotion assigns that value to the new object's property.

### 26. What does a named argument call use to identify a promoted constructor value?

**Answer:** The constructor parameter name.

**Explanation:** Promoted property names and parameter names are the same. Renaming a public constructor parameter can break callers using named arguments.

### 27. What happens if a promoted mutable property is assigned again later in the constructor body?

**Answer:** The later assignment replaces the initial value.

**Explanation:** Promotion initializes the property first. A normal mutable property may be assigned again, which can be useful for trimming or normalizing input.

### 28. Why does the same second assignment fail for a readonly promoted property?

**Answer:** Promotion already initialized the readonly property before the constructor body begins.

**Explanation:** Readonly properties can only be initialized once. Validate a readonly promoted parameter without reassigning the property, or use an explicit property if transformation is needed first.

### 29. What does an attribute on a promoted parameter apply to?

**Answer:** It is replicated to both the constructor parameter and the generated property.

**Explanation:** Attribute-based tools may inspect either reflection target. Ensure the attribute is designed for the targets used by the application.

### 30. How can reflection check whether a property or parameter was promoted?

**Answer:** Call <code>ReflectionProperty::isPromoted()</code> or <code>ReflectionParameter::isPromoted()</code>.

**Explanation:** These methods are mainly useful for frameworks and tooling; most application code does not need them.

## Code reading and application

### 31. What does this print?

~~~php
class Greeting
{
    public function __construct(public string $name)
    {
    }
}

$greeting = new Greeting('Sam');
echo $greeting->name;
~~~

**Answer: <code>Sam</code>.**

**Explanation:** The constructor parameter is promoted to a public property and initialized from the argument.

### 32. What is the value of <code>$settings->page</code>?

~~~php
class Settings
{
    public function __construct(public int $page = 1)
    {
    }
}

$settings = new Settings();
~~~

**Answer: <code>1</code>.**

**Explanation:** The caller omitted the argument, so the constructor parameter default is used and assigned to the promoted property.

### 33. What does this print?

~~~php
class Profile
{
    public function __construct(public ?string $nickname = null)
    {
    }
}

$profile = new Profile();
var_export($profile->nickname);
~~~

**Answer: <code>NULL</code>.**

**Explanation:** The explicit nullable type allows null, and the constructor parameter default supplies null when the argument is omitted.

### 34. What does this print?

~~~php
class Tag
{
    public function __construct(public string $name)
    {
        $this->name = trim($this->name);
    }
}

$tag = new Tag('  php  ');
echo $tag->name;
~~~

**Answer: <code>php</code>.**

**Explanation:** The promotion assignment occurs before the body. Because the property is mutable, the constructor can then store its trimmed value.

### 35. Is this constructor valid on PHP 8.1?

~~~php
class UserId
{
    public function __construct(public readonly int $value)
    {
    }
}
~~~

**Answer: Yes.**

**Explanation:** Readonly properties were introduced in PHP 8.1, and a promoted readonly property is initialized from the constructor argument.

### 36. Why does this fail?

~~~php
class Email
{
    public function __construct(public readonly string $value)
    {
        $this->value = strtolower($value);
    }
}
~~~

**Answer:** Promotion has already initialized <code>$value</code>, so the assignment in the body tries to initialize a readonly property a second time.

**Explanation:** Use a non-promoted parameter and an explicitly declared readonly property if the value must be normalized before storage.

### 37. Is <code>$input</code> available as a property here?

~~~php
class ParsedValue
{
    public string $value;

    public function __construct(string $input)
    {
        $this->value = trim($input);
    }
}
~~~

**Answer: No.**

**Explanation:** <code>$input</code> is an ordinary parameter. The class declares only the <code>$value</code> property and stores the trimmed result there.

### 38. Why is this declaration invalid?

~~~php
class Handler
{
    public function __construct(public callable $callback)
    {
    }
}
~~~

**Answer:** PHP does not allow properties with the <code>callable</code> type.

**Explanation:** Promotion also declares a property, so the property type restriction applies. A <code>Closure</code> type is a common alternative when a closure is required.

### 39. Convert this class to use promotion.

~~~php
class Coordinates
{
    public int $x;
    public int $y;

    public function __construct(int $x, int $y)
    {
        $this->x = $x;
        $this->y = $y;
    }
}
~~~

**Answer:**

~~~php
class Coordinates
{
    public function __construct(
        public int $x,
        public int $y,
    ) {
    }
}
~~~

**Explanation:** Each constructor parameter now also declares a public property and receives the argument value.

### 40. When should you keep an explicit property instead of promoting the constructor input?

**Answer:** When the stored value must be normalized, validated into a different representation, stored under another name, or initialized through more involved logic.

**Explanation:** Promotion is optional shorthand. Explicit properties make it clear when the final property value differs from the raw constructor argument.

## Key takeaways

- Constructor property promotion was introduced in PHP 8.0.
- A modifier such as <code>public</code>, <code>protected</code>, or <code>private</code> promotes a parameter into a property.
- A parameter without a promotion modifier remains an ordinary parameter.
- Promotion initializes the property before the constructor body.
- Readonly promotion requires PHP 8.1 and cannot be reassigned in the constructor body.
- Promotion cannot be used for callable or variadic parameters, abstract constructors, or duplicate properties.
- Use explicit properties when the stored value needs transformation or a different name.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [073 - Readonly properties and classes](../lessons/073-readonly-properties-and-classes.md) | [Lesson](../lessons/074-constructor-property-promotion.md) | [Example](../examples/074-constructor-property-promotion.md) | [075 - Property hooks](../lessons/075-property-hooks.md) |
