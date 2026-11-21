# 111 - Project structure, namespaces, and application layers: Q&A

## Multiple-choice questions

### 1. What is the main purpose of a project's directory structure?

- **A.** Make responsibilities and related code easier to find
- **B.** Make every PHP project use the same framework
- **C.** Replace PHP namespaces
- **D.** Guarantee that an application has no bugs

**Answer: A.** A useful layout helps people locate code and understand its responsibilities. **Explanation:** A folder tree cannot guarantee correctness or replace the language's namespace and autoloading mechanisms.

### 2. Does PHP require every web application to use the same directory layout?

- **A.** Yes, every project must have all common framework directories
- **B.** No, the layout should fit the project's size and responsibilities
- **C.** Yes, but only if Composer is installed
- **D.** No, PHP applications cannot use directories

**Answer: B.** PHP does not mandate one application folder layout. **Explanation:** A small script and a large application may need different amounts of structure.

### 3. For a web application, which directory is commonly configured as the document root?

- **A.** `public/`
- **B.** `src/`
- **C.** `config/`
- **D.** `vendor/`

**Answer: A.** `public/` is a common narrow document root for web applications. **Explanation:** Internal source, configuration, dependencies, and secrets should remain outside the directly served directory.

### 4. What does a PHP namespace do?

- **A.** Organizes symbol names and helps avoid naming collisions
- **B.** Automatically loads every PHP file in the matching directory
- **C.** Restricts file access at the operating-system level
- **D.** Creates a database schema

**Answer: A.** A namespace qualifies names for classes and other supported symbols. **Explanation:** Autoloading is a separate mechanism; a namespace does not load a file or secure a directory.

### 5. What does a Composer PSR-4 mapping connect?

- **A.** A namespace prefix to a base directory
- **B.** An HTTP route to a database table
- **C.** A PHP version to a code formatter
- **D.** A password to an environment variable

**Answer: A.** PSR-4 maps a namespace prefix to a directory and maps remaining name segments to paths. **Explanation:** A class such as `App\Domain\Order\Order` can map to `src/Domain/Order/Order.php` when `App\` maps to `src/`.

### 6. Which layer usually coordinates a use case such as placing an order?

- **A.** Application layer
- **B.** Web server layer
- **C.** Template layer only
- **D.** Generated dependency directory

**Answer: A.** The application layer coordinates the steps of a use case. **Explanation:** It can call domain behavior and request persistence through a suitable boundary.

### 7. Where should a business rule that must remain independent of HTTP and SQL usually live?

- **A.** Domain responsibility
- **B.** Public asset directory
- **C.** Composer lock file
- **D.** Web server rewrite configuration

**Answer: A.** Domain code is a suitable home for business concepts and rules that should not depend on delivery or storage details. **Explanation:** The exact domain design depends on the problem; a small application may need only a few simple objects.

### 8. Which is a typical infrastructure responsibility?

- **A.** Adapt application needs to PDO or an external mail provider
- **B.** Decide what every business rule means
- **C.** Render every page directly from a database driver
- **D.** Define PHP's namespace syntax

**Answer: A.** Infrastructure connects the application to concrete technical tools. **Explanation:** An adapter can translate an application contract into database, filesystem, or provider-specific operations.

### 9. What is a typical responsibility of presentation code?

- **A.** Translate boundary input and application results into HTTP or CLI input/output
- **B.** Own every persistence detail
- **C.** Store Composer packages
- **D.** Replace all domain rules with template conditions

**Answer: A.** Presentation code handles how an application receives and returns information at its boundary. **Explanation:** It can translate HTTP requests or command-line input without owning reusable business rules.

### 10. When is an interface or repository contract most useful?

- **A.** When it protects a meaningful boundary or supports real substitution
- **B.** Whenever a class exists, without exception
- **C.** Only to increase the number of files
- **D.** When it hides every dependency from developers

**Answer: A.** A contract is useful when it clarifies a dependency, enables a genuine alternative, or supports focused tests. **Explanation:** An interface that only renames one call can add indirection without value.

### 11. Where is a natural place to select and connect concrete implementations?

- **A.** The application bootstrap or composition root
- **B.** Inside every domain rule
- **C.** In a publicly served image file
- **D.** In a generated log file

**Answer: A.** The bootstrap can construct concrete services and pass dependencies to the code that needs them. **Explanation:** This keeps low-level construction details from being hidden inside business logic.

### 12. Which Composer project files are commonly committed while `vendor/` is ignored?

- **A.** `composer.json` and `composer.lock`
- **B.** Only `vendor/`
- **C.** Only generated cache files
- **D.** Neither manifest nor lock file

**Answer: A.** The manifest describes dependencies and the lock file records the selected versions for repeatable installation. **Explanation:** `vendor/` is generated by installing dependencies and is commonly excluded from version control.

## Short-answer questions

### 13. Give two benefits of a clear project layout.

**Answer:** It makes related code easier to locate and makes responsibilities easier to understand. **Explanation:** Clear ownership also helps teams review changes and keep unrelated concerns apart.

### 14. Why should `public/` be the web document root instead of the project root?

**Answer:** It limits directly served files to the intended entry point and public assets. **Explanation:** Source code, configuration, dependencies, tests, and local secrets should not be exposed as public files.

### 15. What does a namespace declaration not do by itself?

**Answer:** It does not load the corresponding file or provide filesystem security. **Explanation:** Use an autoloader to locate classes and server/filesystem configuration to control file access.

### 16. With `App\\` mapped to `src/`, where would `App\\Billing\\Invoice` normally be stored under a PSR-4 layout?

**Answer:** `src/Billing/Invoice.php`. **Explanation:** The namespace prefix maps to the base directory; the remaining namespace part maps to directories, and the class name maps to the PHP file.

### 17. Why should namespace, directory, and class filename casing be consistent?

**Answer:** Autoloading depends on matching names to paths, and filesystems can be case-sensitive. **Explanation:** A class that appears to load on one machine may fail on another if path casing differs.

### 18. What is the application layer responsible for?

**Answer:** Coordinating use cases by connecting input, domain behavior, and required operations such as persistence. **Explanation:** It should express meaningful actions rather than become an undifferentiated container for all application code.

### 19. What kind of code belongs in the domain layer?

**Answer:** Business concepts and rules that should not depend on HTTP, templates, SQL, or a framework. **Explanation:** The concrete forms may be entities, value objects, or simpler functions, depending on the problem.

### 20. What does an infrastructure adapter do?

**Answer:** It translates an application's needs to a concrete technical service, such as a database or mail provider. **Explanation:** The application can use the adapter through a boundary that describes the operation it needs.

### 21. What is the composition root?

**Answer:** The startup boundary where concrete dependencies are created and connected to application services. **Explanation:** Keeping construction there makes it easier to see which implementation the running application uses.

### 22. When can a direct dependency be clearer than an interface?

**Answer:** When the project has one straightforward implementation and no meaningful substitution or boundary need. **Explanation:** An interface is not automatically beneficial merely because a class exists.

### 23. What is the difference between a layer-first and a feature-first layout?

**Answer:** Layer-first groups code by technical role, while feature-first groups related responsibilities by application capability. **Explanation:** Either can work; choose the approach that makes cohesive changes easiest to navigate.

### 24. Why should real credentials stay out of committed configuration files?

**Answer:** A repository can be copied, published, or accessed by people who should not receive those secrets. **Explanation:** Read sensitive values from an appropriate local or deployment environment and provide only harmless example values in checked-in templates.

## Practical and scenario questions

### 25. A developer has configured `src/` as the web server document root. What should be changed?

**Answer:** Configure the document root to the application's `public/` directory and place the front controller and public assets there. **Explanation:** This reduces the chance of exposing internal source and configuration files.

### 26. `App\\Catalog\\Product` is declared in `src/Catalog/product.php`, but the class fails to autoload on a case-sensitive server. What should be checked?

**Answer:** Make the filename and each namespace-derived directory match the class and namespace casing, for example `src/Catalog/Product.php`. **Explanation:** PSR-4 path resolution relies on those names matching the filesystem paths.

### 27. A class is correctly namespaced, but PHP reports that it cannot find the class. What should you inspect?

**Answer:** Check that the Composer PSR-4 prefix and base directory match the namespace and file path, and that the generated autoloader is loaded. **Explanation:** A namespace names a symbol; it does not automatically locate the file.

### 28. A script works only when launched from the project root because it opens `config/app.php` relative to the current directory. How can it be made more reliable?

**Answer:** Build the path from a known file location, such as `dirname(__DIR__)`, or use an explicit configured project path. **Explanation:** The process working directory may differ from the directory containing the script.

### 29. A price rule is implemented only in an HTML controller, but the same rule is needed by a CLI import. What change would improve reuse?

**Answer:** Move the reusable rule to a domain or application responsibility and let both entry points call it. **Explanation:** Presentation code should translate inputs and outputs, not be the only home for shared business behavior.

### 30. A small site has one database implementation and a repository interface that merely forwards every method without creating a useful seam. What should the team consider?

**Answer:** Keep the design direct until the interface provides a real boundary, alternative, or testing benefit. **Explanation:** Unnecessary abstraction makes navigation harder without making the dependency meaningfully more flexible.

### 31. A domain object constructs a PDO connection in its constructor. Why is this a concern, and what is a better arrangement?

**Answer:** It couples a business concept to database setup; construct the PDO-backed adapter at startup and pass a suitable application dependency to the code that needs persistence. **Explanation:** This keeps storage details out of domain rules and makes dependencies visible.

### 32. A team is debating layer-first versus feature-first folders. What should determine the choice?

**Answer:** Choose the organization that helps the team find related code and make cohesive changes as the project grows. **Explanation:** Neither layout is universally correct, and a modest hybrid can also work.

### 33. A project contains a large `Helpers.php` file with unrelated formatting, billing, and database functions. What is a useful next step?

**Answer:** Identify each function's responsibility and move cohesive behavior into clearly named modules or classes incrementally. **Explanation:** Generic helper files make ownership and change impact difficult to understand.

### 34. A `.env` file contains a real API key and sits under `public/`. What should be done?

**Answer:** Move the secret outside the document root, exclude the local secret file from Git, and rotate the exposed key if it may have been disclosed. **Explanation:** Both repository access and web-server exposure can reveal credentials.

### 35. A developer wants to copy a large framework's complete directory tree into a tiny practice application. What should guide the decision?

**Answer:** Start with the structure needed to make current responsibilities clear and add directories as real complexity appears. **Explanation:** Structure should help the project, not create unnecessary ceremony.

### 36. An existing application needs clearer boundaries, but a full rewrite would be risky. How can the team proceed?

**Answer:** Move one cohesive responsibility at a time, update namespaces and autoload mappings, and keep each change small and reviewable. **Explanation:** Incremental restructuring reduces risk and makes regressions easier to identify.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [110 - PHP-FIG standards and PSR recommendations](../lessons/110-php-fig-standards-and-psr-recommendations.md) | [Lesson](../lessons/111-project-structure-namespaces-and-application-layers.md) | [Example](../examples/111-project-structure-namespaces-and-application-layers.md) | [112 - Routing, controllers, and middleware](../lessons/112-routing-controllers-and-middleware.md) |
