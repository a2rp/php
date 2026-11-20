# 065 - Visibility and Encapsulation: Q&A

Practice the access rules for properties, methods, and constants, then apply them to encapsulated object state. Every answer includes an explanation. See the [chapter lesson](../lessons/065-visibility-and-encapsulation.md) and [complete example](../examples/065-visibility-and-encapsulation.md).

## Multiple-choice questions

### 1. Which visibility allows access from anywhere that can access the class or object?

A. `private`
B. `protected`
C. `public`
D. `internal`

**Answer:** C. `public`.

**Explanation:** Public members are available to the declaring class, child classes, and unrelated outside code.

### 2. Which visibility is available to a class and its child classes, but not unrelated outside code?

A. `public`
B. `protected`
C. `private`
D. `global`

**Answer:** B. `protected`.

**Explanation:** Protected visibility is intended for a class and its subclasses. Callers outside that inheritance hierarchy cannot access the member directly.

### 3. Which statement about a parent's private property is correct?

A. Every child can read it.
B. It becomes protected in a child.
C. It is accessible only to code declared in the class that defines it.
D. It is public if the child has a getter.

**Answer:** C. It is accessible only to code declared in the class that defines it.

**Explanation:** A child does not gain direct access to a parent's private member. A parent can expose a deliberate public or protected operation if that is part of its design.

### 4. What is the main purpose of encapsulation?

A. Put every property in a separate file.
B. Keep state and its change rules together behind an intentional API.
C. Make every class final.
D. Avoid using methods.

**Answer:** B. Keep state and its change rules together behind an intentional API.

**Explanation:** Encapsulation lets an object validate changes and maintain a consistent state instead of allowing arbitrary writes.

### 5. A public writable balance can be set to a negative value by any caller. What is the central design problem?

A. The class has too many constructors.
B. Callers can bypass the account's rules.
C. Public properties are always slower.
D. Integer properties cannot be changed.

**Answer:** B. Callers can bypass the account's rules.

**Explanation:** The object cannot enforce its invariant when outside code can replace the value directly.

### 6. Which design best protects a bank account's balance?

A. A public property with a descriptive name.
B. A private property with validated deposit and withdrawal methods.
C. A protected property and no methods.
D. A getter and an unrestricted setter.

**Answer:** B. A private property with validated deposit and withdrawal methods.

**Explanation:** The object owns its state and can reject invalid amounts or withdrawals that exceed the available balance.

### 7. A setter accepts any integer and assigns it to a private balance. What is true?

A. The setter guarantees encapsulation.
B. The setter is useful only if it validates or intentionally defines the allowed change.
C. A private property cannot be set from a class method.
D. Every setter must be static.

**Answer:** B. The setter is useful only if it validates or intentionally defines the allowed change.

**Explanation:** Hiding a property but providing an unrestricted setter preserves the same invalid state changes through another route.

### 8. Which member should usually be public in a well-encapsulated class?

A. Every property.
B. Every helper method.
C. Operations callers are intentionally allowed to perform.
D. Every internal constant.

**Answer:** C. Operations callers are intentionally allowed to perform.

**Explanation:** Public members form the class's external API. Implementation details should generally remain private unless subclasses have a real need for them.

### 9. Which is true about a protected property?

A. It is available to unrelated code that has the object.
B. It is available to the declaring class and its children.
C. It is available only in the constructor.
D. It is always read-only.

**Answer:** B. It is available to the declaring class and its children.

**Explanation:** Protected visibility grants access within the class hierarchy, not to arbitrary callers.

### 10. A child overrides an inherited public method. Which visibility may it use?

A. Only private.
B. Protected or private, but never public.
C. Public; it cannot reduce the inherited method's accessibility.
D. Any visibility because overriding resets the API.

**Answer:** C. Public; it cannot reduce the inherited method's accessibility.

**Explanation:** An override may keep the same visibility or widen it, but cannot make an inherited public method less accessible.

### 11. What happens if a child defines a method with the same name as a private method in its parent?

A. The child automatically overrides the private method.
B. The child gets access to the parent's private implementation.
C. The child method is separate and does not override the parent's private method.
D. PHP changes the parent's method to protected.

**Answer:** C. The child method is separate and does not override the parent's private method.

**Explanation:** The private method is not accessible to the child and is not part of the normal inherited override contract.

### 12. Since which PHP version have class constants supported public, protected, and private visibility?

A. PHP 5.0
B. PHP 5.6
C. PHP 7.1
D. PHP 8.4

**Answer:** C. PHP 7.1.

**Explanation:** Class-constant visibility was introduced in PHP 7.1. An omitted visibility modifier means public.

### 13. In which PHP version was asymmetric property visibility introduced?

A. PHP 7.4
B. PHP 8.0
C. PHP 8.1
D. PHP 8.4

**Answer:** D. PHP 8.4.

**Explanation:** PHP 8.4 allows a typed property to have different read and write visibility, such as public read access with private write access.

### 14. With asymmetric property visibility, which rule applies to the set visibility?

A. It must be more permissive than the read visibility.
B. It must be the same as or more restrictive than the read visibility.
C. It must always be public.
D. It can only be protected.

**Answer:** B. It must be the same as or more restrictive than the read visibility.

**Explanation:** For example, public read with private write is valid; protected read with public write would make writing more permissive and is not allowed.

### 15. Which statement about PHP visibility and security is most accurate?

A. A private property replaces authorization checks.
B. Protected members are encrypted.
C. Visibility helps structure a class API but does not replace application security.
D. Public methods cannot reveal sensitive data.

**Answer:** C. Visibility helps structure a class API but does not replace application security.

**Explanation:** Authentication, authorization, safe secret storage, and trust-boundary validation remain necessary even when class properties are private.

## Short-answer questions

### 16. Define public visibility.

**Answer:** A public member can be accessed from the declaring class, child classes, and unrelated outside code.

**Explanation:** Public members are the class's externally available API, so changes to them can affect callers.

### 17. Define protected visibility.

**Answer:** A protected member can be accessed within its declaring class and by classes that extend it, but not by unrelated outside code.

**Explanation:** Use it for intentional subclass extension points, while remembering that subclasses become coupled to the member.

### 18. Define private visibility.

**Answer:** A private member can be accessed only by code declared in the class that defines that member.

**Explanation:** Child classes and unrelated callers cannot directly use it. This keeps implementation details under the class's control.

### 19. What is an invariant in the context of an object?

**Answer:** An invariant is a rule that should remain true for the object's state, such as a bank account balance never being negative.

**Explanation:** Encapsulated methods validate state changes so that callers cannot violate the invariant through ordinary use of the public API.

### 20. Why should a class validate input in deposit rather than rely on every caller to validate it?

**Answer:** Because the class is the authority that owns the balance and should enforce the rule for every caller.

**Explanation:** Caller-side validation can be forgotten or applied differently. Validation at the state-change method provides one consistent enforcement point.

### 21. Why might deposit be a better public operation than setBalance?

**Answer:** deposit expresses a meaningful action and can validate a positive amount; setBalance may allow arbitrary replacement of the state.

**Explanation:** A domain-specific method lets the class preserve invariants and communicates intent more clearly to callers.

### 22. Why can a public property be risky for a class with business rules?

**Answer:** Any caller can read or change it without going through the class's validation logic.

**Explanation:** A public writable property lets external code create states the class was supposed to prevent.

### 23. Should every protected member be made protected in case a future child might need it? Why or why not?

**Answer:** No. Keep implementation details private until a real subclass use case requires protected access.

**Explanation:** Protected members become dependencies for child classes and can make future changes harder.

### 24. Can a child class directly read a parent's private property?

**Answer:** No.

**Explanation:** Private access belongs to the class that declares the property. The parent can offer an appropriate method if subclass access is part of the design.

### 25. Can a method of a class access private data on another instance of that same class?

**Answer:** Yes.

**Explanation:** Private access is based on the class where the method is declared, not solely on whether the target is the current object.

### 26. What is the difference between a private helper method and a public method?

**Answer:** A public method is an operation callers may use; a private helper is an implementation step available only to the declaring class.

**Explanation:** Keeping helpers private lets the class change its internal implementation without making those details part of its public API.

### 27. What does an omitted visibility modifier mean for a class constant?

**Answer:** It means the class constant is public.

**Explanation:** The same public default applies to methods and properties, but explicit visibility is clearer and should be preferred in new class definitions.

### 28. Why is reducing an inherited public method to protected a breaking change?

**Answer:** Existing outside callers that previously called the method can no longer do so.

**Explanation:** Public visibility is part of the class contract. Overrides cannot reduce access to an inherited public or protected method.

### 29. What does public read with private set mean for a typed property in PHP 8.4+?

**Answer:** The property can be read publicly, but writing it is allowed only within the declaring class.

**Explanation:** This permits outside inspection while reserving modifications for the class's own methods and initialization logic.

### 30. Does a private property guarantee that a secret is safe to expose from the class?

**Answer:** No.

**Explanation:** A public method can still reveal the secret. Visibility structures code access; developers must also decide which information the application is authorized to return.

## Scenarios and code review

### 31. Review this class. What is the problem, and what design would protect its invariant?

~~~php
class Account
{
    public int $balanceInCents = 0;
}

$account = new Account();
$account->balanceInCents = -1000;
~~~

**Answer:** The public writable property lets any caller create a negative balance. Make it private and expose validated operations such as deposit and withdraw.

**Explanation:** The object must enforce its own balance rules. Callers should not be able to bypass those rules by assigning state directly.

### 32. Will this child method work? Explain.

~~~php
class ParentRecord
{
    private string $id = 'P-1';
}

class ChildRecord extends ParentRecord
{
    public function id(): string
    {
        return $this->id;
    }
}
~~~

**Answer:** No. ChildRecord::id() cannot access the parent's private id.

**Explanation:** Private members are visible only within the class that declared them. The parent would need an appropriate method to provide access.

### 33. Is this override valid?

~~~php
class BaseHandler
{
    public function handle(): void
    {
    }
}

class SpecialHandler extends BaseHandler
{
    protected function handle(): void
    {
    }
}
~~~

**Answer:** No. The child reduces the inherited method from public to protected.

**Explanation:** A child override may keep the same visibility or widen access, but it cannot restrict a public parent method.

### 34. Does the following setter preserve a non-negative balance invariant?

~~~php
class Account
{
    private int $balance = 0;

    public function setBalance(int $balance): void
    {
        $this->balance = $balance;
    }
}
~~~

**Answer:** No. It accepts negative values and writes them directly to the private property.

**Explanation:** Private storage alone is not enough. The public state-change operation must enforce the rule, or the API should expose validated domain operations.

### 35. Can Record::copyIdFrom() read other.id in this code?

~~~php
class Record
{
    private string $id;

    public function __construct(string $id)
    {
        $this->id = $id;
    }

    public function copyIdFrom(Record $other): string
    {
        return $other->id;
    }
}
~~~

**Answer:** Yes.

**Explanation:** The access occurs inside Record, the class that declares id. A method of a class can access that class's private state on another instance of the same class.

### 36. Does ChildJob::runInternal() override BaseJob::runInternal()?

~~~php
class BaseJob
{
    private function runInternal(): string
    {
        return 'base';
    }
}

class ChildJob extends BaseJob
{
    private function runInternal(): string
    {
        return 'child';
    }
}
~~~

**Answer:** No. The child declares a separate private method with the same name.

**Explanation:** The parent's private method is not available for overriding in the child. Each method is accessible only from its declaring class.

### 37. In PHP 8.4+, can outside code assign to this property after constructing it?

~~~php
class Profile
{
    public private(set) string $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }
}

$profile = new Profile('Kai');
~~~

**Answer:** No. Outside code can read profile.name, but cannot assign a new value to it.

**Explanation:** The public read visibility permits reading; private set restricts writes to the class that declares the property.

### 38. Is this asymmetric visibility declaration allowed in PHP 8.4+?

~~~php
class Item
{
    protected public(set) string $code;
}
~~~

**Answer:** No.

**Explanation:** Public set is more permissive than the protected read visibility. A property's set visibility must be the same as or more restrictive than its read visibility.

### 39. Is protected a good default for every helper method in a reusable class?

**Answer:** No. A helper should usually be private unless subclasses have a concrete need to call or customize it.

**Explanation:** Protected methods become part of the inheritance contract, coupling child classes to implementation details.

### 40. A private token property is returned by a public getToken method. Is the token protected from callers?

**Answer:** No. The public method exposes it, regardless of the property's private visibility.

**Explanation:** Visibility controls direct member access, not the information returned by a public method. The application must authorize and limit disclosure.

## Coding and applied questions

### 41. Write a class that stores a non-negative balance privately, reports it through a getter, and accepts only positive deposits.

**Answer:**

~~~php
class Wallet
{
    private int $balanceInCents = 0;

    public function balanceInCents(): int
    {
        return $this->balanceInCents;
    }

    public function deposit(int $amountInCents): void
    {
        if ($amountInCents <= 0) {
            throw new InvalidArgumentException(
                'Deposit must be greater than zero.'
            );
        }

        $this->balanceInCents += $amountInCents;
    }
}
~~~

**Explanation:** The property is private, reading is explicit, and the only provided balance-changing operation rejects zero and negative amounts.

### 42. Add a private helper to normalize a title and a public method that returns a URL slug.

**Answer:**

~~~php
class Slug
{
    public static function fromTitle(string $title): string
    {
        return self::normalize($title);
    }

    private static function normalize(string $title): string
    {
        $title = strtolower(trim($title));
        $title = preg_replace('/[^a-z0-9]+/', '-', $title) ?? '';

        return trim($title, '-');
    }
}
~~~

**Explanation:** Callers use the public operation. The normalization steps remain a private implementation detail that the class may change later.

### 43. Create a parent class with a protected title and a child method that uses it, while exposing the result through a public method.

**Answer:**

~~~php
class Page
{
    protected string $title;

    public function __construct(string $title)
    {
        $this->title = $title;
    }
}

class HelpPage extends Page
{
    public function heading(): string
    {
        return 'Help: ' . $this->title;
    }
}

$page = new HelpPage('Account settings');
echo $page->heading(); // Help: Account settings
~~~

**Explanation:** HelpPage can use the protected property because it extends Page. Outside code uses the public heading method.

### 44. What PHP version is needed for this property, and what can outside code do with it?

~~~php
public private(set) string $title;
~~~

**Answer:** PHP 8.4 or later is required. Outside code can read the property but cannot assign to it.

**Explanation:** Asymmetric property visibility separates read access (public) from write access (private set).

### 45. Refactor this code so callers cannot set an invalid negative page size.

~~~php
class PageOptions
{
    public int $pageSize = 20;
}

$options = new PageOptions();
$options->pageSize = -1;
~~~

**Answer:**

~~~php
class PageOptions
{
    private int $pageSize;

    public function __construct(int $pageSize = 20)
    {
        $this->setPageSize($pageSize);
    }

    public function pageSize(): int
    {
        return $this->pageSize;
    }

    public function setPageSize(int $pageSize): void
    {
        if ($pageSize < 1 || $pageSize > 100) {
            throw new InvalidArgumentException(
                'Page size must be from 1 to 100.'
            );
        }

        $this->pageSize = $pageSize;
    }
}
~~~

**Explanation:** The property is private and both initialization and later changes use the same validation rule. A domain-specific method such as changePageSize could be used instead if that better describes the class's public API.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [064 - Constructors and destructors](../lessons/064-constructors-and-destructors.md) | [Lesson](../lessons/065-visibility-and-encapsulation.md) | [Example](../examples/065-visibility-and-encapsulation.md) | [066 - Static properties and methods](../lessons/066-static-properties-and-methods.md) |
