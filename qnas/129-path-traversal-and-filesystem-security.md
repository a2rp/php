# 129 - Path traversal and filesystem security: Questions and answers

Use these questions to review how untrusted input can escape an intended directory and how PHP applications can safely map, resolve, and serve files.

## Multiple-choice questions

### 1. What is path traversal?

- A. A query that reads several database tables.
- B. Untrusted input changes a filesystem path so it points outside the intended directory.
- C. A browser follows a link to another page.
- D. PHP checks whether a file exists.

**Answer:** B. **Explanation:** Path traversal can expose or modify files outside the directory the application intended to use.

### 2. What is the safest ordinary design for selecting a private file?

- A. Accept an arbitrary path and remove ../.
- B. Accept a logical identifier and map it to a server-controlled path.
- C. Accept any filename with a .txt extension.
- D. Use basename() and trust the result.

**Answer:** B. **Explanation:** The request selects an approved key, while the server controls the actual storage path.

### 3. What does realpath() return when it successfully resolves an existing path?

- A. A canonical absolute path with dot components and symbolic links resolved.
- B. The original user input unchanged.
- C. The contents of the file.
- D. The user's permissions for the file.

**Answer:** A. **Explanation:** realpath() resolves an existing filesystem path; it returns false if it cannot resolve the target.

## Path construction and containment

### 4. What causes a path traversal vulnerability?

**Answer:** The application builds a path using untrusted input without ensuring the final target stays inside the intended directory. **Explanation:** Parent-directory components, absolute paths, alternate separators, or links can redirect the lookup.

### 5. Why is concatenating $_GET['file'] into a path unsafe?

**Answer:** The request value may contain path syntax that escapes the intended folder. **Explanation:** Input is not a trustworthy filename just because it came through a form or URL parameter.

### 6. Why is a server-side identifier-to-path map safer?

**Answer:** The user chooses only a known identifier, while trusted server code chooses the file location. **Explanation:** The request never becomes an arbitrary path.

### 7. Is an unguessable document ID an authorization check?

**Answer:** No. **Explanation:** IDs can be exposed or shared; the application still needs to check whether the current user may access the document.

### 8. What does realpath() do with dot components and symbolic links?

**Answer:** It resolves dot references and expands symbolic links for an existing path. **Explanation:** The resulting canonical path can be compared with an approved base directory.

### 9. Why must realpath()'s return value be checked?

**Answer:** It returns false when the target does not exist or cannot be resolved. **Explanation:** Passing false into later path logic can create incorrect comparisons or warnings.

### 10. Can realpath() validate a file that has not been created yet?

**Answer:** No. **Explanation:** The target must exist for realpath() to resolve it; validate the trusted parent directory and use a safe creation strategy for new files.

### 11. Why is a plain prefix comparison insufficient for containment?

**Answer:** A sibling path such as /srv/private-files-old begins with the text /srv/private-files. **Explanation:** Compare a canonical base path ending at a path separator or compare path components.

### 12. Why must the candidate target be resolved as well as the base directory?

**Answer:** The candidate may contain traversal components or a symlink that points elsewhere. **Explanation:** Compare the final canonical target against the canonical base, not the raw request string.

### 13. Why should path behavior be tested on the deployment operating system?

**Answer:** Filesystems differ in separators, case handling, drive paths, junctions, and network mount behavior. **Explanation:** A check that works on one platform may not enforce the same boundary on another.

### 14. Does rejecting the literal string ../ catch every traversal form?

**Answer:** No. **Explanation:** Encoded input, backslashes, repeated separators, and platform-specific syntax can express the same escape in another way.

### 15. Should a request value be URL-decoded a second time after validation?

**Answer:** No. **Explanation:** A second decode can turn previously checked text into separators; decode once at the boundary, then validate and use that same representation.

### 16. Why is basename() not a complete defense?

**Answer:** It does not prove the result is an approved file, authorized for the user, or safe from symlink resolution. **Explanation:** It only returns a trailing path component according to its parsing rules.

### 17. Does checking a filename extension enforce directory containment?

**Answer:** No. **Explanation:** A filename can have an expected suffix and still point outside the intended directory.

### 18. Is file_exists() followed by fopen() enough to prevent path attacks?

**Answer:** No. **Explanation:** Existence does not imply authorization or containment, and the target may change between the check and use.

### 19. What is a time-of-check/time-of-use race in filesystem access?

**Answer:** The filesystem changes after the application validates a path but before it opens or modifies the target. **Explanation:** An attacker who can modify relevant directories or symlinks may redirect the later operation.

### 20. What helps reduce symlink and path replacement races?

**Answer:** Keep the directory tree writable only by trusted processes and use APIs that enforce containment during the open operation when available. **Explanation:** A prior string check alone cannot control changes made after the check.

## Includes, wrappers, and runtime restrictions

### 21. Why is include $_GET['page'] dangerous?

**Answer:** The request can select an unintended file for PHP to execute. **Explanation:** Dynamic inclusion can turn a path traversal flaw into code execution or expose sensitive source.

### 22. How should an application select among a fixed set of templates?

**Answer:** Use an allowlist that maps approved keys to server-controlled template paths. **Explanation:** Do not pass the caller's value directly to include or require.

### 23. What is a PHP stream wrapper?

**Answer:** A scheme that lets some stream or filesystem functions access resources using forms such as php:// or phar://. **Explanation:** Whether wrappers are accepted depends on the function, configuration, and installed extensions.

### 24. Should user input be passed directly to file_get_contents() even if the application only intends to read local files?

**Answer:** No. **Explanation:** The function can support wrapper and URL forms; use a server-controlled path and an explicit allowlist.

### 25. Does allow_url_include prevent local path traversal?

**Answer:** No. **Explanation:** That setting concerns URL inclusion; it does not stop a local path from escaping its intended directory.

### 26. Is open_basedir a complete filesystem security boundary?

**Answer:** No. **Explanation:** It can restrict accidental access as an extra safeguard, but it does not enforce per-user authorization and PHP documents it as not comprehensive.

### 27. Can open_basedir replace application authorization?

**Answer:** No. **Explanation:** A PHP process may be permitted to read a directory while the current user should be denied access to a particular file inside it.

## Private files and uploads

### 28. Where should private uploads and generated documents be stored?

**Answer:** Outside the publicly served web root. **Explanation:** Serve them through an authorized application handler rather than allowing a static URL to bypass access checks.

### 29. Is $_FILES['name'] safe as a server-side filename?

**Answer:** No. **Explanation:** The original filename is supplied by the client and can contain misleading or unsafe content; generate a storage name on the server.

### 30. Is the uploaded Content-Type header proof of the file format?

**Answer:** No. **Explanation:** It is untrusted client metadata and must not be the only validation for a file's intended use.

### 31. What does move_uploaded_file() help verify?

**Answer:** It moves a file that PHP recognizes as an uploaded file from its temporary location. **Explanation:** It does not validate content, choose a safe storage name, authorize access, or make the destination private by itself.

### 32. Why should the PHP worker have limited filesystem permissions?

**Answer:** A compromised route or bug then has less ability to read or modify unrelated files. **Explanation:** Least privilege limits the impact of mistakes.

### 33. Why should a writable upload directory not also execute PHP files?

**Answer:** If a user can place executable content in a served directory, a misconfiguration may allow the web server to run it. **Explanation:** Keep private or active uploads outside the document root and disable execution in data storage.

### 34. What should a private download handler do before reading a file?

**Answer:** Authorize the current user, resolve a server-controlled path, verify it is an allowed regular file inside the storage base, and then serve it safely. **Explanation:** Path containment and user permission are separate checks.

## Errors and testing

### 35. Should raw server paths be shown to users when a file lookup fails?

**Answer:** No. **Explanation:** Paths can reveal directory layout and sensitive filenames; return a generic response and keep protected diagnostics in server logs.

### 36. Why might an application return the same 404 for a missing and unauthorized document?

**Answer:** To avoid revealing whether another user's private file exists. **Explanation:** The application must still deny access before returning data or performing a side effect.

### 37. What should a traversal test include besides ../?

**Answer:** Encoded and repeated traversal strings, backslashes, absolute paths, symlinks, wrapper-like values, and platform-specific path syntax. **Explanation:** Attackers test how the full request pipeline and operating system interpret the value.

### 38. What upload cases should be tested?

**Answer:** Misleading extensions, MIME types, original filenames, oversized content, and files that may contain active content. **Explanation:** Also confirm stored files cannot be executed or reached directly when they are private.

### 39. What should a denied file request leave unchanged?

**Answer:** It should disclose no protected contents or raw paths and should not modify, move, or delete a file. **Explanation:** Test the side effects and response, not just the status code.

### 40. What is the main rule for filesystem security?

**Answer:** Keep paths under server control; when user input must select a file, map it through an allowlist, enforce canonical containment, and authorize the user separately. **Explanation:** Filesystem permissions and runtime limits add defense in depth but do not replace those checks.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [128 - Session and cookie security](../lessons/128-session-and-cookie-security.md) | [Lesson](../lessons/129-path-traversal-and-filesystem-security.md) | [Example](../examples/129-path-traversal-and-filesystem-security.md) | [130 - SSRF, URL validation, and redirect safety](../lessons/130-ssrf-url-validation-and-redirect-safety.md) |