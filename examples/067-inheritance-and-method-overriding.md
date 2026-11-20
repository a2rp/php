# 067 - Inheritance and Method Overriding: Example

## Question

How can a child class reuse parent initialization, override selected methods, and still be used wherever the parent type is expected? How can a class collaborate with another object when inheritance is not the right relationship?

## Complete example

Save the following PHP 8.0+ code as `inheritance-and-method-overriding.php` and run `php inheritance-and-method-overriding.php` from a terminal.

~~~php
<?php

declare(strict_types=1);

class Member
{
    protected string $name;
    private string $memberId;

    public function __construct(string $name, string $memberId)
    {
        $name = trim($name);
        $memberId = trim($memberId);

        if ($name === '') {
            throw new InvalidArgumentException('Name cannot be empty.');
        }

        if ($memberId === '') {
            throw new InvalidArgumentException('Member ID cannot be empty.');
        }

        $this->name = $name;
        $this->memberId = $memberId;
    }

    public function role(): string
    {
        return 'Member';
    }

    public function profile(): string
    {
        return sprintf(
            '%s (%s) - %s',
            $this->name,
            $this->memberId,
            $this->role()
        );
    }

    final public function memberId(): string
    {
        return $this->memberId;
    }
}

class Moderator extends Member
{
    private string $section;

    public function __construct(
        string $name,
        string $memberId,
        string $section,
    ) {
        parent::__construct($name, $memberId);

        $section = trim($section);

        if ($section === '') {
            throw new InvalidArgumentException('Section cannot be empty.');
        }

        $this->section = $section;
    }

    public function role(): string
    {
        return 'Moderator';
    }

    public function profile(): string
    {
        return parent::profile() . ' - manages ' . $this->section;
    }

    public function badge(): string
    {
        return strtoupper($this->name) . ' / ' . $this->section;
    }
}

function printProfile(Member $member): void
{
    echo $member->profile() . PHP_EOL;
}

printProfile(new Member('Mina', 'M-01'));

$moderator = new Moderator('Omar', 'M-02', 'community');
printProfile($moderator);
echo 'Badge: ' . $moderator->badge() . PHP_EOL;

// The child cannot access or replace the parent's private memberId directly.
// $moderator->memberId = 'M-99'; // Error: memberId is private.

class Food
{
}

class AnimalFood extends Food
{
}

class Animal
{
    public function __construct(
        public string $name,
    ) {
    }
}

class Dog extends Animal
{
}

class Shelter
{
    public function adopt(string $name): Animal
    {
        return new Animal($name);
    }

    public function feed(AnimalFood $food): string
    {
        return 'Shelter accepts animal food.';
    }
}

class DogShelter extends Shelter
{
    public function adopt(string $name): Dog
    {
        return new Dog($name);
    }

    public function feed(Food $food): string
    {
        return 'Dog shelter accepts ' . $food::class . '.';
    }
}

$dogShelter = new DogShelter();
$dog = $dogShelter->adopt('Rex');

echo 'Adopted type: ' . $dog::class . ' named ' . $dog->name . PHP_EOL;
echo $dogShelter->feed(new Food()) . PHP_EOL;

class TextFormatter
{
    public function format(string $text): string
    {
        return strtoupper($text);
    }
}

class Report
{
    public function __construct(
        private TextFormatter $formatter,
    ) {
    }

    public function render(string $text): string
    {
        return $this->formatter->format($text);
    }
}

$report = new Report(new TextFormatter());
echo 'Report: ' . $report->render('monthly results') . PHP_EOL;
~~~

## Expected output

~~~text
Mina (M-01) - Member
Omar (M-02) - Moderator - manages community
Badge: OMAR / community
Adopted type: Dog named Rex
Dog shelter accepts Food.
Report: MONTHLY RESULTS
~~~

## Explanation

### The child reuses parent initialization

Moderator declares its own constructor, so it calls parent::__construct() explicitly. The parent validates and stores the name and member ID before the child validates its own section.

Member keeps memberId private and exposes a final public method to read it. Moderator inherits and can call that method, but cannot directly access the private property. The name is protected because the child badge method intentionally uses it.

### Overriding enables polymorphism

printProfile() accepts a Member. It works for both Member and Moderator objects. When the argument is a Moderator, PHP dispatches profile() and role() to the Moderator implementations.

Moderator::profile() uses parent::profile() to preserve the parent's formatting, then adds the moderator's section. Inside the parent profile method, the call to $this->role() still dispatches to Moderator::role() for a Moderator object.

### The override keeps a compatible contract

DogShelter::adopt() returns Dog, which is more specific than the parent promise of Animal. That is a covariant return type.

The parent feed() accepts AnimalFood. DogShelter::feed() accepts Food, the broader parent type. It still accepts every AnimalFood value the parent contract allowed, and this broader parameter is contravariant.

PHP 7.4 and later support full covariance and contravariance for method signatures.

### Composition models collaboration

Report is not a TextFormatter. It receives one and delegates formatting to it. This is composition: an object uses another object instead of inheriting its type.

## Try these changes

1. Create another Moderator object with a different name and section. Which methods are inherited and which are overridden?
2. Remove the parent constructor call. What parent initialization is skipped?
3. Change Moderator::profile() so it does not call parent::profile(). What output changes?
4. Try to override Member::memberId(). Why does the final declaration prevent it?
5. Change DogShelter::adopt() to return object instead of Dog. Is the broader return type compatible with the parent's promise?
6. Try passing AnimalFood to DogShelter::feed(). Does the broader Food parameter accept it?
7. Replace inheritance between Report and TextFormatter with a subclass relationship. Does that describe a true "is a" relationship?

## Key takeaways

- A child can inherit public and protected members but cannot directly access a parent's private members.
- A child-defined constructor must explicitly call the parent constructor when its initialization is needed.
- Runtime method dispatch selects the implementation for the actual object.
- parent:: can preserve and extend parent behavior.
- Covariant return types and contravariant parameter types keep overrides substitutable.
- Composition is appropriate when one object collaborates with another instead of being a specialized kind of it.
- The example uses PHP 8.0+ syntax.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [066 - Static properties and methods](../lessons/066-static-properties-and-methods.md) | [Lesson](../lessons/067-inheritance-and-method-overriding.md) | [Q&A](../qnas/067-inheritance-and-method-overriding.md) | [068 - Composition and object collaboration](../lessons/068-composition-and-object-collaboration.md) |
