# Project: build a personal portfolio page — Example

## Goal

Build a small one-page portfolio with PHP-rendered project cards, reusable header and footer templates, safe HTML output, and responsive CSS. The project is static: it does not collect visitor data or require a database.

## Project structure

Create these files:

~~~text
portfolio/
├── public/
│   ├── index.php
│   └── assets/
│       └── styles.css
└── src/
    ├── portfolio-data.php
    └── templates/
        ├── header.php
        └── footer.php
~~~

Keep **src/** outside the public document root. The web server should serve files from **public/**.

## 1. Portfolio data

### File: src/portfolio-data.php

~~~php
<?php
declare(strict_types=1);

return [
    'name' => 'Maya Verma',
    'role' => 'PHP developer',
    'intro' => 'I build clear, reliable web applications with PHP.',
    'about' => 'I enjoy turning practical problems into simple, accessible tools. I focus on readable code, careful validation, and useful experiences.',
    'email' => 'maya@example.test',
    'skills' => [
        'PHP',
        'HTML',
        'CSS',
        'SQL',
    ],
    'projects' => [
        [
            'name' => 'Task tracker',
            'description' => 'A small task manager with clear status and due dates.',
            'url' => 'https://example.com/task-tracker',
        ],
        [
            'name' => 'Reading list',
            'description' => 'A personal list for saving books and tracking reading progress.',
            'url' => 'https://example.com/reading-list',
        ],
        [
            'name' => 'Command-line notes',
            'description' => 'A local command-line tool for keeping short notes.',
            'url' => null,
        ],
    ],
];
~~~

The data array separates portfolio content from page markup. The third project has no public URL, so the page can render its description without showing a broken or misleading link.

## 2. Shared page header

### File: src/templates/header.php

~~~php
<header class="site-header">
    <a class="site-name" href="#top">
        <?= escape($portfolio['name']) ?>
    </a>

    <nav aria-label="Main navigation">
        <ul class="navigation">
            <li><a href="#about">About</a></li>
            <li><a href="#projects">Projects</a></li>
            <li><a href="#skills">Skills</a></li>
            <li><a href="#contact">Contact</a></li>
        </ul>
    </nav>
</header>
~~~

This template uses the **escape** helper defined by the page. Navigation links point to section IDs in the same document.

## 3. Shared page footer

### File: src/templates/footer.php

~~~php
<footer class="site-footer">
    <p>
        &copy; <?= date('Y') ?>
        <?= escape($portfolio['name']) ?>.
        Built with PHP.
    </p>
</footer>
~~~

The year is generated at request time so the footer does not need a manual annual edit.

## 4. Main PHP page

### File: public/index.php

~~~php
<?php
declare(strict_types=1);

$portfolio = require __DIR__ . '/../src/portfolio-data.php';

/**
 * Escape a string for HTML text and quoted attribute output.
 */
function escape(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE,
        'UTF-8'
    );
}

/**
 * Return an HTTPS project URL, or null when it is not acceptable.
 */
function safeProjectUrl(?string $url): ?string
{
    if ($url === null || filter_var($url, FILTER_VALIDATE_URL) === false) {
        return null;
    }

    if (parse_url($url, PHP_URL_SCHEME) !== 'https') {
        return null;
    }

    return $url;
}
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta
        name="description"
        content="<?= escape($portfolio['intro']) ?>"
    >
    <title>
        <?= escape($portfolio['name']) ?> — <?= escape($portfolio['role']) ?>
    </title>
    <link rel="stylesheet" href="assets/styles.css">
</head>
<body>
    <a class="skip-link" href="#main-content">Skip to main content</a>

    <div class="page-shell" id="top">
        <?php require __DIR__ . '/../src/templates/header.php'; ?>

        <main id="main-content">
            <section class="hero" aria-labelledby="intro-heading">
                <p class="eyebrow"><?= escape($portfolio['role']) ?></p>
                <h1 id="intro-heading">
                    Hi, I'm <?= escape($portfolio['name']) ?>.
                </h1>
                <p class="hero-copy"><?= escape($portfolio['intro']) ?></p>
                <a class="button-link" href="#projects">Explore my projects</a>
            </section>

            <section class="content-section" id="about" aria-labelledby="about-heading">
                <div class="section-heading">
                    <p class="eyebrow">A little about me</p>
                    <h2 id="about-heading">About</h2>
                </div>
                <p class="prose"><?= escape($portfolio['about']) ?></p>
            </section>

            <section class="content-section" id="projects" aria-labelledby="projects-heading">
                <div class="section-heading">
                    <p class="eyebrow">Selected work</p>
                    <h2 id="projects-heading">Projects</h2>
                </div>

                <div class="project-grid">
                    <?php foreach ($portfolio['projects'] as $project): ?>
                        <?php $projectUrl = safeProjectUrl($project['url']); ?>
                        <article class="project-card">
                            <h3><?= escape($project['name']) ?></h3>
                            <p><?= escape($project['description']) ?></p>

                            <?php if ($projectUrl !== null): ?>
                                <a href="<?= escape($projectUrl) ?>">
                                    View <?= escape($project['name']) ?>
                                </a>
                            <?php else: ?>
                                <p class="project-note">No public demo available.</p>
                            <?php endif; ?>
                        </article>
                    <?php endforeach; ?>
                </div>
            </section>

            <section class="content-section" id="skills" aria-labelledby="skills-heading">
                <div class="section-heading">
                    <p class="eyebrow">Tools I use</p>
                    <h2 id="skills-heading">Skills</h2>
                </div>

                <ul class="skill-list">
                    <?php foreach ($portfolio['skills'] as $skill): ?>
                        <li><?= escape($skill) ?></li>
                    <?php endforeach; ?>
                </ul>
            </section>

            <section class="contact-section" id="contact" aria-labelledby="contact-heading">
                <p class="eyebrow">Have a project in mind?</p>
                <h2 id="contact-heading">Let's talk.</h2>
                <p>
                    Send me an email if you would like to discuss an idea or
                    collaborate.
                </p>
                <a class="button-link" href="mailto:<?= escape($portfolio['email']) ?>">
                    Email <?= escape($portfolio['name']) ?>
                </a>
            </section>
        </main>

        <?php require __DIR__ . '/../src/templates/footer.php'; ?>
    </div>
</body>
</html>
~~~

### How the page works

- The data file returns one array; the page stores it in **$portfolio**.
- **__DIR__** anchors include paths to the current file instead of relying on the shell's working directory.
- **escape** encodes dynamic strings for HTML text and quoted attributes.
- **safeProjectUrl** accepts only valid HTTPS URLs. A missing or rejected URL becomes **null**, and that project is rendered without a link.
- **foreach** renders each project and skill from the data instead of duplicating the same markup.
- The header and footer are included from fixed source paths. A visitor cannot choose an include path.
- The page has a skip link, a navigation landmark, one main region, labeled sections, and a logical heading order.

HTML escaping and URL policy solve different problems. Escaping project names protects their HTML output; checking the scheme decides whether a link destination is allowed.

## 5. Responsive stylesheet

### File: public/assets/styles.css

~~~css
:root {
    color-scheme: light;
    font-family: system-ui, sans-serif;
    color: #1c2733;
    background: #f4f7fa;
    font-synthesis: none;
    text-rendering: optimizeLegibility;
}

*,
*::before,
*::after {
    box-sizing: border-box;
}

body {
    margin: 0;
    line-height: 1.6;
}

a {
    color: #075e9b;
    text-underline-offset: 0.2em;
}

a:hover {
    color: #043c64;
}

a:focus-visible {
    outline: 3px solid #c04b18;
    outline-offset: 4px;
}

.page-shell {
    width: min(100% - 2rem, 68rem);
    margin-inline: auto;
}

.site-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 1.5rem;
    padding-block: 1.25rem;
    border-bottom: 1px solid #ccd5de;
}

.site-name {
    color: inherit;
    font-weight: 750;
    text-decoration: none;
}

.navigation,
.skill-list {
    display: flex;
    flex-wrap: wrap;
    gap: 0.75rem 1.25rem;
    margin: 0;
    padding: 0;
    list-style: none;
}

.navigation a {
    color: inherit;
}

.skip-link {
    position: absolute;
    top: 0.5rem;
    left: 0.5rem;
    z-index: 1;
    padding: 0.6rem 0.9rem;
    transform: translateY(-160%);
    color: #fff;
    background: #17324d;
}

.skip-link:focus {
    transform: translateY(0);
}

.hero {
    padding-block: clamp(4rem, 12vw, 8rem);
    max-width: 48rem;
}

.eyebrow {
    margin: 0 0 0.5rem;
    color: #526476;
    font-size: 0.85rem;
    font-weight: 700;
    letter-spacing: 0.08em;
    text-transform: uppercase;
}

h1,
h2,
h3,
p {
    margin-top: 0;
}

h1 {
    max-width: 14ch;
    margin-bottom: 1rem;
    font-size: clamp(2.5rem, 8vw, 5rem);
    line-height: 1.05;
}

h2 {
    margin-bottom: 0.5rem;
    font-size: clamp(1.8rem, 5vw, 2.5rem);
    line-height: 1.15;
}

h3 {
    margin-bottom: 0.6rem;
    line-height: 1.25;
}

.hero-copy {
    max-width: 42rem;
    color: #394b5c;
    font-size: 1.2rem;
}

.button-link {
    display: inline-block;
    margin-top: 0.75rem;
    padding: 0.7rem 1rem;
    border: 2px solid #075e9b;
    border-radius: 0.35rem;
    color: #fff;
    background: #075e9b;
    font-weight: 700;
    text-decoration: none;
}

.button-link:hover {
    color: #fff;
    background: #043c64;
}

.content-section,
.contact-section {
    padding-block: 3.5rem;
    border-top: 1px solid #ccd5de;
}

.section-heading {
    margin-bottom: 1.5rem;
}

.prose {
    max-width: 55rem;
    color: #394b5c;
    font-size: 1.1rem;
}

.project-grid {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 1rem;
}

.project-card {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    min-height: 13rem;
    padding: 1.25rem;
    border: 1px solid #ccd5de;
    border-radius: 0.6rem;
    background: #fff;
}

.project-card p {
    color: #394b5c;
}

.project-card a,
.project-note {
    margin-top: auto;
}

.project-note {
    font-size: 0.95rem;
}

.skill-list li {
    padding: 0.45rem 0.8rem;
    border: 1px solid #a8bac9;
    border-radius: 99rem;
    background: #fff;
}

.contact-section {
    padding-bottom: 4rem;
}

.contact-section > p:not(.eyebrow) {
    max-width: 42rem;
    color: #394b5c;
}

.site-footer {
    padding-block: 1.5rem;
    border-top: 1px solid #ccd5de;
    color: #526476;
    font-size: 0.95rem;
}

.site-footer p {
    margin: 0;
}

@media (max-width: 48rem) {
    .site-header {
        align-items: flex-start;
        flex-direction: column;
    }

    .project-grid {
        grid-template-columns: 1fr;
    }

    .project-card {
        min-height: 0;
    }
}
~~~

## Run it locally

From the **portfolio/** project directory, start the PHP development server with **public/** as its document root:

~~~sh
php -S localhost:8000 -t public
~~~

Open the local address in a browser. The built-in server is for local development and controlled demonstrations, not production hosting.

## Expected page content

The browser shows:

- A header with **Maya Verma** and About, Projects, Skills, and Contact navigation.
- An introduction identifying Maya as a PHP developer.
- An About section with the supplied background paragraph.
- Three project cards. The first two have HTTPS links; Command-line notes has no public demo link.
- A Skills list containing PHP, HTML, CSS, and SQL.
- A contact section with an email link.
- A footer with the current year.

## Suggested checks

- Change a project name to include angle brackets and confirm the browser displays them as text.
- Change a project URL to a non-HTTPS scheme and confirm no link is rendered for that card.
- Set a project URL to **null** and confirm its description still appears without a placeholder link.
- Navigate the page using only the keyboard and confirm the skip link and focus indicator are visible.
- Narrow the browser viewport and confirm project cards become a single column.
- Confirm a request for **src/portfolio-data.php** is not publicly served by the web server.

## Important limits

- The example's data is trusted local configuration. If content later comes from an administrator, a database, or a user, validate it at the appropriate boundary and continue escaping it at output.
- The email address is an example. Replace it with a contact method you intentionally want to publish.
- The example has no form, persistence, authentication, or authorization because a static portfolio does not need them.
- The PHP development server is not a production deployment strategy.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [147 - Code review, maintenance, and upgrades](../lessons/147-code-review-maintenance-and-upgrades.md) | [Lesson](../lessons/148-project-build-a-personal-portfolio-page.md) | [Q&A](../qnas/148-project-build-a-personal-portfolio-page.md) | [149 - Project: create a validated contact form](../lessons/149-project-create-a-validated-contact-form.md) |
