# Project: build a personal portfolio page

A personal portfolio introduces you and presents selected work in one clear, accessible place. In this project, PHP will assemble a page from structured data and reusable templates. The browser receives ordinary HTML and CSS; PHP runs on the server to produce that response.

The project practices several ideas from earlier chapters: PHP files and includes, arrays, functions, output escaping, HTML structure, CSS, and safe separation between public files and application code.

## Project goals

Build a responsive portfolio page that includes:

- A short introduction and a clear page title.
- A navigation menu with links to sections on the same page.
- An about section with concise background information.
- A projects section generated from PHP data.
- A skills section that is easy to scan.
- Contact links such as email or a professional profile.
- A shared page header and footer.
- Escaped dynamic text and validated link destinations.
- A layout that remains readable on narrow and wide screens.

Keep the first version static. A contact form, database, login, and content-management system are not needed for this project. Add them only when there is a real requirement and a plan to protect and maintain them.

## Plan before coding

### Identify the audience and purpose

Decide who should use the portfolio and what they should understand first. A recruiter, client, collaborator, and teacher may look for different details. Pick a primary audience and make the most useful content easy to find.

Write down:

- Your name or display name.
- A one-sentence description of your work.
- A small set of relevant skills.
- A few projects with honest summaries and working destinations.
- The contact method you are comfortable publishing.

Do not publish personal information merely because there is space for it. A portfolio is public content; include only what supports its purpose.

### Sketch the page sections

A simple one-page layout might have:

1. A header with your name and section navigation.
2. A main introduction with a clear heading and short description.
3. An about section.
4. A projects section.
5. A skills section.
6. A contact section.
7. A footer with a small amount of secondary information.

Give each section a useful heading. Use real links for navigation and actions so they work with keyboards, screen readers, and browser features.

## Organize the project

Keep web-accessible files separate from application code:

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

The **public/** directory is the document root. The browser should not be able to request source files or private configuration from **src/**. In a production deployment, configure the web server's document root to point at **public/**.

### Keep content in structured data

Store repeated project details in an array rather than duplicating the same markup for every project. Each project can contain a title, short description, and destination URL. The template loops over the list and renders one card per item.

This makes content easier to update and helps keep all project cards consistent. It does not make the data safe automatically: escape text when rendering it, and check that links use an allowed destination scheme.

### Reuse shared templates

A header and footer include keep repeated page structure in one place. Keep includes focused and predictable. Avoid turning a small static page into a complicated framework.

A PHP include executes code in the current request. Include only application files you control, using a path built from the current file location. Do not let a request parameter choose an include path.

## Render data safely

Dynamic values must be encoded for the context where they are inserted. For text and quoted HTML attributes, a common HTML-escaping helper is:

~~~php
function escape(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE,
        'UTF-8'
    );
}
~~~

Use the helper when rendering names, descriptions, titles, and attribute values. This prevents text such as angle brackets from being interpreted as markup.

Escaping is context-specific. HTML text escaping does not validate a URL, make JavaScript safe, or sanitize CSS. For a project link, allow only expected schemes such as HTTPS, or construct a URL from trusted local data. Do not put arbitrary strings into a link destination.

When a project does not have a public link, render its name and description without an anchor instead of inventing a placeholder destination.

## Build accessible HTML

Use elements for their meaning:

- **header** contains introductory or navigational content for the page.
- **nav** groups navigation links.
- **main** contains the page's primary content.
- **section** groups a related topic and normally has a heading.
- **article** can represent a project card that makes sense as an item on its own.
- **footer** contains secondary information about the page or site.

Use one clear page-level heading, then headings that describe the sections and their content. Heading levels represent structure; do not choose them only for their visual size.

Every interactive element must be usable by keyboard. Links should describe their destination, not rely on “click here.” If you add an image, provide alternative text that communicates its purpose. A decorative image should not create redundant spoken content.

## Make the layout responsive

Start with a readable narrow-screen layout and expand it for wider screens. Use flexible widths, spacing, and a small number of breakpoints. Keep text lines comfortable to read, and make links and controls easy to reach.

A stylesheet can provide:

- A consistent color and spacing system.
- Clear visual hierarchy between the introduction, sections, and project cards.
- Responsive columns that collapse to one column on small screens.
- Visible keyboard focus.
- Sufficient contrast between text and its background.

Do not communicate status or meaning with color alone. Check the page at more than one viewport width and with keyboard navigation.

## Run the project locally

From the project directory, start PHP's built-in server with **public/** as the document root:

~~~sh
php -S localhost:8000 -t public
~~~

Then open the local address in a browser. The built-in PHP server is intended for development and controlled demonstrations, not as a public production server. For deployment, use a properly configured PHP-capable web server.

If the page is blank or shows an error:

1. Check the terminal where PHP is running.
2. Confirm that the document root points to **public/**.
3. Confirm include paths resolve from the PHP file's location.
4. Check PHP syntax and error messages in the development environment.
5. Verify the project data contains the fields the template expects.

Do not display detailed errors to public visitors in production.

## Review the finished page

Check the page from a visitor's point of view:

- Can a visitor tell who you are and what you do within a few seconds?
- Are project names and descriptions concise and accurate?
- Do all links go to the expected destinations?
- Does the page work without a mouse?
- Are headings and sections understandable when read in order?
- Does the layout work on a phone-sized screen?
- Is dynamic text escaped and are link schemes controlled?
- Is anything private or unnecessary being exposed?
- Can someone maintain the project data without changing the page structure?

## Common mistakes

- Putting the entire page, data, and styling in one difficult-to-maintain file.
- Repeating project-card markup instead of rendering a data list.
- Printing dynamic values directly into HTML without escaping.
- Treating HTML escaping as URL validation.
- Linking to a placeholder such as **#** when there is no real destination.
- Using a heading level only to make text look larger.
- Relying on color alone to distinguish links or information.
- Putting private source files inside the public document root.
- Using the PHP development server as a public production host.
- Adding a contact form without validation, spam protection, and a safe mail-handling design.
- Publishing an email address or other personal detail without considering the exposure.

## Completion checklist

| Area | Complete when |
| --- | --- |
| Content | Introduction, about, projects, skills, and contact information have a clear purpose |
| PHP structure | Repeated content is represented as data and shared layout uses focused includes |
| Output safety | Dynamic text is escaped and link destinations are restricted to expected schemes |
| Semantics | Page landmarks, headings, links, and project items use appropriate HTML elements |
| Accessibility | Keyboard navigation, focus visibility, link names, and image alternatives are considered |
| Responsive design | Content remains readable and usable at narrow and wide viewport sizes |
| Privacy | Only intentional public information appears |
| Local setup | The PHP development server runs with **public/** as its document root |
| Deployment | A PHP-capable production server serves **public/** and does not expose source files |

## Key takeaways

- Start with the audience and content before writing markup.
- Keep repeated content in data and shared layout in focused templates.
- Escape dynamic HTML text and validate URL schemes separately.
- Use semantic HTML and design for keyboard and narrow-screen use.
- Keep application source outside the public document root.
- Use PHP's built-in server for local development, then deploy to a production web server.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [147 - Code review, maintenance, and upgrades](147-code-review-maintenance-and-upgrades.md) | [Q&A](../qnas/148-project-build-a-personal-portfolio-page.md) | [Example](../examples/148-project-build-a-personal-portfolio-page.md) | [149 - Project: create a validated contact form](149-project-create-a-validated-contact-form.md) |
