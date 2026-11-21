# 095 - Validating uploads and storing files safely

## Learning goals

By the end of this lesson, you will be able to:

- Apply layered checks to an untrusted uploaded file.
- Use an allowlist for the file types and sizes a feature needs.
- Explain why filenames, extensions, MIME claims, and signatures are each insufficient alone.
- Choose storage that cannot execute uploaded content or expose it by an unintended URL.
- Build a controlled download path with authorization and safe response headers.
- Plan resource limits, malware scanning, cleanup, logging, and failure handling.

## 1. Treat every upload as untrusted input

An upload may be malformed, mislabeled, excessively large, or crafted to exploit a parser. It may also contain active content that becomes dangerous when rendered or served to another user.

A successful PHP upload only means that PHP received a temporary file. It does not prove that the content is acceptable or safe. The upload feature must apply its own policy before processing, storing, or serving the file.

Start with the product requirement: which users need to upload which kinds of files, for which purpose, and who may later access them? Keep the allowed formats and limits as narrow as the feature permits.

## 2. Use layered validation

Apply checks in a deliberate order:

1. **Authenticate and authorize:** Confirm the user is signed in and allowed to upload for this account, record, or workspace.
2. **Protect the request:** Use CSRF protection for cookie-authenticated forms and accept only the intended HTTP method.
3. **Check the PHP upload structure:** Verify the expected field exists and has the shape and error code the handler expects.
4. **Enforce resource limits:** Check per-file bytes, number of files, user quota, request size, dimensions, and processing budgets where relevant.
5. **Apply a narrow extension allowlist:** Compare the decoded filename extension to formats the feature needs. Do not use a denylist of dangerous extensions as the primary control.
6. **Inspect server-side content:** Determine the content type from the temporary file with a server-side parser such as Fileinfo. Do not trust the browser's type value or request Content-Type.
7. **Validate the format for its purpose:** Parse or decode the content with an appropriate, maintained library. For image features, consider decoding and re-encoding to a known format.
8. **Create a server-side name and controlled storage location:** Never use the original filename as a path.
9. **Record metadata and check every operation:** Persist only verified metadata, handle storage/database failures, and remove orphaned temporary or stored files when appropriate.

No single test proves a file is safe. Use the checks together, based on the file's intended use and the application's threat model.

## 3. Allowlist extensions and compare content

An extension is useful as part of a file policy, but it does not prove what the file contains. A name such as `photo.jpg.php`, uppercase variants, path separators, or unusual Unicode characters can confuse simplistic checks. If the business needs the original name, validate it for display and store it as metadata; never use it as the storage path.

A practical allowlist maps accepted extensions to expected detected MIME types:

~~~php
$allowedTypes = [
    'jpg' => 'image/jpeg',
    'jpeg' => 'image/jpeg',
    'png' => 'image/png',
];

$extension = strtolower(pathinfo($originalName, PATHINFO_EXTENSION));
$expectedMime = $allowedTypes[$extension] ?? null;

$fileInfo = new finfo(FILEINFO_MIME_TYPE);
$detectedMime = $fileInfo->file($temporaryPath);

if ($expectedMime === null || $detectedMime !== $expectedMime) {
    throw new RuntimeException('Unsupported file type.');
}
~~~

The original name and its extension remain untrusted input. This check rejects obvious mismatches, but MIME detection is not a full parser and can be fooled by polyglot or malformed files. Validate the content according to how the application will use it.

A signature check can provide another signal, but a signature by itself is not enough. Do not accept a file just because a few leading bytes resemble a known format.

## 4. Parse and normalize complex formats

A validator should answer whether the file can be parsed as the format the feature expects. For an image, a maintained decoder should actually decode the image. If the application only needs a raster image, re-encoding it to a restricted output format can strip unrelated metadata and appended data.

The PHP function `getimagesize()` can report image dimensions and a possible type, but it is not a general image-validity or safety check. Do not use it by itself as proof that a file is a valid, harmless image.

Parsing still has risk: image libraries, document parsers, and archive tools process attacker-controlled bytes. Keep dependencies updated and apply memory, CPU, pixel-count, archive-entry, and decompressed-size limits. Run high-risk processing in an isolated worker or sandbox where practical.

Do not extract ZIP or other archives unless the feature requires it. If extraction is required, prevent path traversal, reject links and special files, limit entry counts and expanded size, and ensure each target remains inside the intended extraction directory.

## 5. Generate names and store files outside the web root

The client chooses the original name. It may contain traversal sequences, collisions, misleading extensions, or characters that behave differently across platforms. Do not build a path from that value.

Instead:

- Generate an opaque name on the server, for example from `random_bytes()`.
- Choose the output extension from the validated format, not the submitted name.
- Store the original name separately only if the product needs it.
- Store uploads outside the public document root, or on a separate storage service.
- Restrict write permissions to the application process and avoid executable permissions.
- Prevent other users or local processes from writing into the upload directory.

`move_uploaded_file()` verifies that its source is a PHP upload, but it does not validate the file's content or make the destination safe. It overwrites an existing target, so use server-generated names and manage collisions.

If a file must be public, publish it deliberately through a controlled retrieval flow. A random URL is not a substitute for authorization.

## 6. Serve downloads through an authorized handler

A download endpoint should accept an application-level record ID, look up the stored file using trusted server-side metadata, and check that the current user may access it. Never accept an arbitrary filesystem path from a query parameter.

For a private download:

1. Authenticate the request.
2. Load the file record by its application ID.
3. Check the user's authorization for that record.
4. Resolve a server-controlled storage key, not a client-supplied path.
5. Set a verified content type, a safe `Content-Disposition` filename, and `X-Content-Type-Options: nosniff`.
6. Prefer `Content-Disposition: attachment` for formats that should not be rendered inline.
7. Log access and failures according to the application's privacy policy.

The browser should not receive the internal storage path. Escape or construct any download filename using a strict policy; never copy a stored untrusted name directly into a response header.

## 7. Limit storage and processing costs

Uploads can exhaust disk space, memory, CPU, and network capacity even when the file type is allowed. Apply a maximum per-file size, count limit, per-user quota, and total storage policy. Keep PHP, the web server, reverse proxy, and application body limits consistent.

For image features, bound width, height, and total pixel count before expensive processing, then enforce worker memory and time limits. For archives and documents, consider expanded size, entry count, nested archives, parser limits, and conversion time.

Temporary files and rejected files should not accumulate indefinitely. Define cleanup for abandoned uploads, failed processing, expired records, and user deletion. Make database metadata and storage operations recoverable if one succeeds while the other fails.

## 8. Scan, quarantine, and audit where needed

For higher-risk formats or public sharing, use malware scanning or content disarm and reconstruction (CDR) when appropriate. Treat a scanner as an additional layer, not proof of safety. A newly received file can remain in quarantine until validation and scanning succeed.

Keep audit events useful and privacy-conscious. Log the application user ID, outcome, generated storage ID, detected type, and relevant limits without logging file contents, secrets, or unnecessary personal data. Avoid exposing internal parser messages and storage paths to visitors.

## 9. Common approaches that are not enough

### Checking only the filename extension

Extensions can be changed or combined with another extension. Compare a narrow extension allowlist with server-detected content and format-specific validation.

### Trusting the browser's MIME type

The request's content type is client-controlled. Inspect the temporary file on the server and validate the format.

### Using `getimagesize()` as the only image check

It can report dimensions for data that is not a valid safe image. Use a real decoder for the expected format and consider re-encoding, while still limiting the parser's resources.

### Storing uploads under the public directory

A server may execute or render a file in an unexpected way, or a URL may expose it without application authorization. Store private files outside the web root and use a guarded download handler.

### Using a random URL as access control

URLs can be shared, logged, guessed, or leaked. Check permission on every private download request.

## Key takeaways

- Design the allowed file policy around the feature and apply layered server-side checks.
- Validate both the extension policy and server-detected content; then parse complex formats for their intended use.
- `getimagesize()` alone does not validate an image.
- Generate server-side storage names and keep private uploads outside the web root.
- Use an authorized download handler instead of trusting a URL or client-provided path.
- Bound bytes, counts, quotas, dimensions, parser resources, and storage lifetime.
- Use scanning, quarantine, logging, and recovery as additional controls where the risk requires them.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [094 - Handling file uploads](094-handling-file-uploads.md) | [Q&A](../qnas/095-validating-uploads-and-storing-files-safely.md) | [Example](../examples/095-validating-uploads-and-storing-files-safely.md) | [096 - Making HTTP requests with streams and cURL](096-making-http-requests-with-streams-and-curl.md) |