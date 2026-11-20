# 046 - Generators and yield: Example

## Question

How can a generator produce a keyed report one record at a time, delegate to another generator, and return a final count?

## Goal

Create a small price-report program that:

- Computes discounted prices as the caller requests them
- Yields each result with its product code as the key
- Delegates from a report generator to a calculation generator with **yield from**
- Returns the number of processed products and reads it with **getReturn()**
- Uses **send()** to provide input to a suspended generator

## Project structure

~~~text
generator-practice/
└── index.php
~~~

## Complete code

Save this as **index.php**:

~~~php
<?php

function discountedPrices(array $prices, float $rate): Generator
{
    $count = 0;

    foreach ($prices as $code => $price) {
        echo "Calculating {$code}\n";

        $count++;

        yield $code => $price * (1 - $rate);
    }

    return $count;
}

function discountReport(array $prices, float $rate): Generator
{
    $count = yield from discountedPrices($prices, $rate);

    return $count;
}

function askForName(): Generator
{
    $name = yield 'What is your name?';

    yield "Hello, {$name}!";
}

$prices = [
    'book' => 20.00,
    'pen' => 4.00,
    'bag' => 30.00,
];

$report = discountReport($prices, 0.10);

echo "Report generator created.\n";

foreach ($report as $code => $discountedPrice) {
    echo $code, ': $', number_format($discountedPrice, 2), "\n";
}

echo 'Items processed: ', $report->getReturn(), "\n";

$dialogue = askForName();

echo $dialogue->current(), "\n";

$dialogue->send('Mina');

echo $dialogue->current(), "\n";
~~~

## Expected output

~~~text
Report generator created.
Calculating book
book: $18.00
Calculating pen
pen: $3.60
Calculating bag
bag: $27.00
Items processed: 3
What is your name?
Hello, Mina!
~~~

## Explanation

### Create a generator that calculates one result at a time

~~~php
function discountedPrices(array $prices, float $rate): Generator
{
    $count = 0;

    foreach ($prices as $code => $price) {
        echo "Calculating {$code}\n";

        $count++;

        yield $code => $price * (1 - $rate);
    }

    return $count;
}
~~~

The function contains **yield**, so calling it returns a **Generator** object. It does not calculate every price immediately. The calculation for a product runs when iteration requests that product.

The **yield $code => $value** form provides each discounted price with its product code as a key. The generator also keeps **$count** between pauses and returns it after the loop finishes.

### Delegate from the report generator

~~~php
function discountReport(array $prices, float $rate): Generator
{
    $count = yield from discountedPrices($prices, $rate);

    return $count;
}
~~~

**yield from** passes the inner generator's key/value pairs through the report generator. Once the inner generator finishes, the **yield from** expression evaluates to the inner generator's return value, which is stored in **$count** and returned by the outer generator.

### Observe when the work happens

~~~php
$report = discountReport($prices, 0.10);

echo "Report generator created.\n";

foreach ($report as $code => $discountedPrice) {
    echo $code, ': $', number_format($discountedPrice, 2), "\n";
}
~~~

Creating **$report** does not print any **Calculating ...** line. The first line is printed when **foreach** starts iterating. Each next calculation happens as the loop requests another result.

The source prices are already stored in an array in this example. The generator avoids building a second array containing all discounted results before they are consumed.

### Read the final return value

~~~php
echo 'Items processed: ', $report->getReturn(), "\n";
~~~

The generator returns the count after the last yielded price. **getReturn()** is called after **foreach** has exhausted the generator. The return value is separate from the yielded product prices.

### Send a value into a generator

~~~php
function askForName(): Generator
{
    $name = yield 'What is your name?';

    yield "Hello, {$name}!";
}
~~~

The first **yield** provides a prompt and suspends the generator. **send('Mina')** supplies **'Mina'** as the result of the suspended **yield** expression and resumes the function. The next **yield** provides the greeting.

## Make a change

1. Add another product and confirm the count increases.
2. Change the discount rate from **0.10** to **0.20** and recalculate the expected prices.
3. Remove the **$code =>** part of **yield** and observe the automatic integer keys.
4. Call **$report->getReturn()** before iteration and observe that the generator has not finished.
5. Add another **yield** to **askForName()** and continue iterating after the greeting.
6. Change the name sent into the generator.

## Common mistakes

- Expecting **discountReport()** to calculate all prices when the generator object is created.
- Forgetting that **yield** pauses execution and later resumes it.
- Expecting **getReturn()** to include the returned count as a yielded item.
- Calling **getReturn()** before the generator is exhausted.
- Forgetting that **yield from** preserves keys from the inner generator.
- Assuming a generator creates no memory use at all; it only avoids eagerly collecting the generated output into an array.
- Calling **send()** without understanding that it supplies the current suspended **yield** expression's value.
- Treating a generator like an array with random index access.

## Key takeaways

- A function containing **yield** returns a **Generator** object.
- Iteration runs the function as needed and resumes it after each yield.
- **yield $key => $value** can provide meaningful keys to the consumer.
- **yield from** delegates values from an inner generator and evaluates to its final return value.
- **getReturn()** retrieves the final value after iteration completes.
- **send()** resumes a generator and provides a value to its suspended **yield** expression.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [045 - Passing and Returning Values by Reference](../lessons/045-passing-and-returning-values-by-reference.md) | [Read lesson](../lessons/046-generators-and-yield.md) | [Open Q&A](../qnas/046-generators-and-yield.md) | [047 - Iterators, Traversable, and Iterable Values](../lessons/047-iterators-traversable-and-iterable-values.md) |
