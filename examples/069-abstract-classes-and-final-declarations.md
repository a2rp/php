# Abstract classes and final declarations - Example

## Goal

Create two document exporters that share a safe export workflow but render different formats.

- The abstract base validates input and controls the export sequence.
- Each concrete exporter supplies its extension and rendering format.
- A final result class represents the completed export.
- The base export method is final so subclasses cannot skip validation or change the sequence.

The main example requires PHP 8.0 or later.

## Complete PHP example

~~~php
<?php
declare(strict_types=1);

final class ExportReceipt
{
    public function __construct(
        private string $fileName,
        private string $content
    ) {
    }

    public function summary(): string
    {
        return 'Created ' . $this->fileName;
    }

    public function content(): string
    {
        return $this->content;
    }
}

abstract class DocumentExporter
{
    final public function export(string $title, array $sections): ExportReceipt
    {
        $title = trim($title);

        if ($title === '') {
            throw new InvalidArgumentException('The document title cannot be empty.');
        }

        foreach ($sections as $section) {
            if (!is_string($section) || trim($section) === '') {
                throw new InvalidArgumentException(
                    'Every section must be a non-empty string.'
                );
            }
        }

        $fileName = $this->makeFileName($title);
        $content = $this->renderDocument($title, $sections);

        return new ExportReceipt($fileName, $content);
    }

    private function makeFileName(string $title): string
    {
        $slug = strtolower(str_replace(' ', '-', $title));

        return $slug . '.' . $this->extension();
    }

    abstract protected function extension(): string;

    /**
     * @param array<int, string> $sections
     */
    abstract protected function renderDocument(
        string $title,
        array $sections
    ): string;
}

class MarkdownExporter extends DocumentExporter
{
    protected function extension(): string
    {
        return 'md';
    }

    protected function renderDocument(string $title, array $sections): string
    {
        $lines = ['# ' . $title];

        foreach ($sections as $section) {
            $lines[] = '- ' . trim($section);
        }

        return implode(PHP_EOL, $lines);
    }
}

class PlainTextExporter extends DocumentExporter
{
    protected function extension(): string
    {
        return 'txt';
    }

    protected function renderDocument(string $title, array $sections): string
    {
        $lines = [strtoupper($title), str_repeat('=', strlen($title))];

        foreach ($sections as $section) {
            $lines[] = '* ' . trim($section);
        }

        return implode(PHP_EOL, $lines);
    }
}

$exporter = new MarkdownExporter();

$receipt = $exporter->export(
    'Release notes',
    [
        'Improved search',
        'Fixed login',
    ]
);

echo $receipt->summary() . PHP_EOL;
echo $receipt->content();
~~~

## Expected output

~~~text
Created release-notes.md
# Release notes
- Improved search
- Fixed login
~~~

## How the design works

The object is assembled by choosing a concrete exporter:

~~~php
$exporter = new MarkdownExporter();
~~~

The concrete class is usable because it implements both abstract operations:

- <code>extension()</code> supplies the output file extension.
- <code>renderDocument()</code> creates format-specific content.

The inherited final <code>export()</code> method owns the steps every exporter must follow:

1. Trim and validate the document title.
2. Validate each section.
3. Build the output filename.
4. Ask the child class to render the document.
5. Return a final <code>ExportReceipt</code>.

The child can customize the two required rendering details, but cannot override <code>export()</code> and bypass validation. The final result class can be instantiated and used, but another class cannot extend it.

## Choose another concrete implementation

The same workflow can use the plain-text exporter:

~~~php
$exporter = new PlainTextExporter();

$receipt = $exporter->export(
    'Release notes',
    [
        'Improved search',
        'Fixed login',
    ]
);

echo $receipt->summary() . PHP_EOL;
echo $receipt->content();
~~~

Expected output:

~~~text
Created release-notes.txt
RELEASE NOTES
=============
* Improved search
* Fixed login
~~~

The caller chooses which concrete exporter to instantiate. Both classes follow the abstract base class's requirements.

## What cannot be done

The abstract class is incomplete, so this is invalid:

~~~php
$exporter = new DocumentExporter();
~~~

A child that omits either abstract method is also incomplete and cannot be instantiated as a concrete class.

This is invalid because <code>export()</code> is final:

~~~php
class UnsafeExporter extends MarkdownExporter
{
    public function export(string $title, array $sections): ExportReceipt
    {
        // A child cannot replace the final workflow.
    }
}
~~~

This is invalid because <code>ExportReceipt</code> is final:

~~~php
class CustomReceipt extends ExportReceipt
{
}
~~~

## Version-specific declarations

The main example targets PHP 8.0. These newer declarations need a newer runtime.

### Final class constants - PHP 8.1+

~~~php
class ExportDefaults
{
    final public const FORMAT_VERSION = '1';
}
~~~

### Abstract and final properties - PHP 8.4+

Abstract properties use property hooks to require accessors:

~~~php
abstract class LabeledExport
{
    abstract public string $label {
        get;
    }
}

class MarkdownExportLabel extends LabeledExport
{
    public string $label = 'Markdown';
}
~~~

A final property prevents a child from redeclaring the property:

~~~php
class ExportMetadata
{
    final protected string $category = 'document';
}
~~~

These features are version-gated. Use abstract methods instead of abstract properties when the project supports PHP versions before 8.4.

## Practice

1. Add a <code>JsonExporter</code> that extends <code>DocumentExporter</code>.
2. Return the <code>json</code> extension and render the title and sections as JSON.
3. Try overriding <code>export()</code> in <code>JsonExporter</code>. Explain why PHP rejects it.
4. Try removing one abstract method implementation. Why can that class no longer be instantiated?
5. Decide whether <code>ExportReceipt</code> should remain final if application code needs custom receipt subclasses.

## Key takeaways

- Abstract classes combine shared behavior with requirements for concrete subclasses.
- Child classes implement every inherited abstract method with a compatible signature.
- A final template method can preserve a workflow while abstract methods provide extension points.
- A final class cannot be extended, but can still be instantiated.
- Final constants require PHP 8.1; abstract and final properties require PHP 8.4.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [068 - Composition and object collaboration](../lessons/068-composition-and-object-collaboration.md) | [Lesson](../lessons/069-abstract-classes-and-final-declarations.md) | [Q&A](../qnas/069-abstract-classes-and-final-declarations.md) | [070 - Interfaces and contracts](../lessons/070-interfaces-and-contracts.md) |
