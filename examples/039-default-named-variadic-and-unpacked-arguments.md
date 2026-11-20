# 039 - Default, Named, Variadic, and Unpacked Arguments: Example

## Question

How can a notification function support sensible defaults, clear named options, and a flexible list of checklist items?

## Goal

Use a default parameter, named arguments, a variadic parameter, and argument unpacking in one small PHP 8+ command-line program.

## Project structure

~~~text
argument-options/
└── index.php
~~~

## Complete code

Save this as **index.php**:

~~~php
<?php

function printNotice(
    string $recipient,
    string $message,
    string $channel = 'email',
    bool $urgent = false,
): void {
    $priority = $urgent ? 'urgent' : 'normal';
    echo "{$channel} ({$priority}) to {$recipient}: {$message}\n";
}

function printChecklist(string $title, string ...$items): void
{
    echo "{$title}\n";

    foreach ($items as $item) {
        echo "- {$item}\n";
    }
}

function printDimensions(int $width, int $height, int $depth): void
{
    echo "Dimensions: {$width} x {$height} x {$depth}\n";
}

echo "Notifications\n";

printNotice('Asha', 'Build passed');

printNotice(
    'Mina',
    'Review requested',
    channel: 'chat',
    urgent: true,
);

printNotice(
    message: 'Backup complete',
    recipient: 'Sam',
    urgent: true,
);

$noticeArguments = [
    'recipient' => 'Ops',
    'message' => 'Deployment complete',
    'channel' => 'dashboard',
    'urgent' => true,
];
printNotice(...$noticeArguments);

echo "\n";

$releaseTasks = [
    'Run checks',
    'Review changes',
    'Publish notes',
];
printChecklist('Release checklist', ...$releaseTasks);
printChecklist('Quiet checklist');

echo "\n";

$dimensions = [120, 80, 35];
printDimensions(...$dimensions);
printDimensions(...[1920, 1080, 24]);
~~~

## Expected output

Run the file with PHP 8.0 or later:

~~~sh
php index.php
~~~

The output is:

~~~text
Notifications
email (normal) to Asha: Build passed
chat (urgent) to Mina: Review requested
email (urgent) to Sam: Backup complete
dashboard (urgent) to Ops: Deployment complete

Release checklist
- Run checks
- Review changes
- Publish notes
Quiet checklist

Dimensions: 120 x 80 x 35
Dimensions: 1920 x 1080 x 24
~~~

## How it works

### Default arguments

The declaration gives **$channel** the default **'email'** and **$urgent** the default **false**:

~~~php
function printNotice(
    string $recipient,
    string $message,
    string $channel = 'email',
    bool $urgent = false,
): void {
    // ...
}
~~~

In **printNotice('Asha', 'Build passed')**, the caller supplies the two required arguments and omits the optional ones. PHP uses their defaults.

### Named arguments

This call names the options explicitly:

~~~php
printNotice(
    'Mina',
    'Review requested',
    channel: 'chat',
    urgent: true,
);
~~~

The first two positional values map to **$recipient** and **$message**. The named arguments set **$channel** and **$urgent**.

This call uses names for every argument and changes their order:

~~~php
printNotice(
    message: 'Backup complete',
    recipient: 'Sam',
    urgent: true,
);
~~~

The order of named arguments does not change which parameter receives each value. The omitted **$channel** uses its default.

### String-keyed unpacking supplies named arguments

~~~php
$noticeArguments = [
    'recipient' => 'Ops',
    'message' => 'Deployment complete',
    'channel' => 'dashboard',
    'urgent' => true,
];
printNotice(...$noticeArguments);
~~~

In PHP 8+, string keys in the unpacked array are interpreted as parameter names. Each key must match a parameter. This is useful when an associative array already contains named input values.

### Variadic parameters collect values

The **printChecklist()** declaration has one required parameter, **$title**, and then the variadic **$items** parameter:

~~~php
function printChecklist(string $title, string ...$items): void
{
    // ...
}
~~~

The three task arguments in **printChecklist('Release checklist', ...$releaseTasks)** are unpacked at the call site. The function receives the title in **$title** and collects the task strings into **$items**. The **foreach** loop prints each item.

The call **printChecklist('Quiet checklist')** supplies no trailing items, so **$items** is an empty array and the loop prints no task lines.

### Positional unpacking supplies fixed parameters

The dimensions function has three positional parameters. The list **[120, 80, 35]** is unpacked into three arguments, in order:

~~~php
$dimensions = [120, 80, 35];
printDimensions(...$dimensions);
~~~

The same works with an array literal, as in **printDimensions(...[1920, 1080, 24])**.

### The two meanings of the ellipsis

- In **string ...$items**, the declaration collects extra arguments into an array.
- In **printChecklist(...$releaseTasks)**, the call expands an iterable into arguments.

The token looks the same, but its role depends on where it appears.

## Make a change

1. Change the default channel from **email** to **console** and omit the channel in a call.
2. Add another notice using only named arguments in a different order.
3. Add a **$category** parameter to **printNotice()**, then pass it by name.
4. Add two tasks to **$releaseTasks** and observe how the variadic function prints them.
5. Remove all task values and call **printChecklist()** with only its title.
6. Add a fourth dimension parameter and update both unpacked lists to supply four values.

## Common mistakes

- Assuming a default replaces an explicit **null** argument.
- Putting a required parameter after a parameter with a default.
- Misspelling a parameter name in a named argument or associative unpacked array.
- Supplying the same parameter both positionally and by name.
- Putting a positional argument after a named argument.
- Using named argument syntax on PHP versions before 8.0.
- Declaring a variadic parameter before another parameter.
- Expecting unpacking in a call to collect values; collection happens in a variadic declaration.
- Unpacking a list with the wrong number or type of values for fixed parameters.

## Key takeaways

- Omitted arguments use defaults; explicitly supplied values do not.
- Named arguments identify parameters by name and require PHP 8.0 or later.
- Variadic parameters collect zero or more trailing arguments into an array.
- Unpacking expands an array or iterable into function-call arguments.
- String-keyed unpacked arrays supply named arguments in PHP 8+.
- The **...** syntax collects in a declaration and expands in a call.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [038 - Function Parameters and Arguments](../lessons/038-function-parameters-and-arguments.md) | [Read lesson](../lessons/039-default-named-variadic-and-unpacked-arguments.md) | [Open Q&A](../qnas/039-default-named-variadic-and-unpacked-arguments.md) | [040 - Function Return Types](../lessons/040-function-return-types.md) |
