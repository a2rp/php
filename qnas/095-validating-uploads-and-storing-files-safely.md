# 095 - Validating uploads and storing files safely: Q&A

## Multiple-choice questions

### 1. What is the strongest starting point for deciding which file types to accept?

- **A.** Accept every type except files whose names contain “script”
- **B.** Allow only formats required by the feature
- **C.** Accept every file detected as an image
- **D.** Trust the file picker's `accept` attribute

**Answer: B.** Define a narrow allowlist from the product requirement. A denylist misses unexpected formats and naming tricks, while a browser hint is not a server-side control.

### 2. Which values are controlled by the upload client?

- **A.** Only the original filename
- **B.** Only the browser MIME type
- **C.** The original filename, extension, and browser MIME claim
- **D.** None, because PHP creates `$_FILES`

**Answer: C.** Treat submitted names, extensions, and browser-provided MIME values as untrusted metadata. PHP's upload structure does not validate that the content matches those claims.

### 3. What does Fileinfo contribute to an upload check?

- **A.** It determines a content type from the temporary file on the server
- **B.** It grants permission to upload
- **C.** It proves that a file contains no malware
- **D.** It creates a safe public URL

**Answer: A.** Fileinfo can identify a likely MIME type from file content. It is one signal; it is not a complete parser or a guarantee that the file is safe.

### 4. Why compare an allowed extension with a detected content type?

- **A.** To let the browser choose a filename
- **B.** To reject obvious mismatches while applying more validation
- **C.** To avoid checking the PHP upload error
- **D.** To make any file safe to execute

**Answer: B.** The comparison catches some mislabeled files, but both checks are only part of a layered validation policy.

### 5. Is checking the first few bytes of a file enough to prove its format is safe?

- **A.** Yes, signatures cannot be forged
- **B.** Yes, if the extension also matches
- **C.** No, a signature is another signal and can be insufficient on its own
- **D.** No, because files never have signatures

**Answer: C.** Signature checks can help identify obvious mismatches, but they do not replace a format-aware parser and safe handling.

### 6. Why generate a server-side storage name?

- **A.** To preserve the client's folder path
- **B.** To prevent client-controlled names from becoming paths or causing collisions
- **C.** To make malware scanning unnecessary
- **D.** To tell the browser which user is signed in

**Answer: B.** An opaque generated name avoids traversal, unsafe characters, and collisions caused by submitted names. Choose its extension from the validated format.

### 7. Where should private uploads normally be stored?

- **A.** In a public executable scripts directory
- **B.** Outside the web root, or on a separate storage service
- **C.** In a directory named after the original filename
- **D.** In the public directory with a random URL

**Answer: B.** Private storage outside the web root prevents direct URL access. Serve files through an application handler that checks authorization.

### 8. Does `move_uploaded_file()` validate the file's content?

- **A.** Yes, it checks whether the bytes are a safe format
- **B.** Yes, it removes malware before moving
- **C.** No, it verifies the source is a PHP upload and moves it
- **D.** No, it moves any arbitrary local path

**Answer: C.** The function protects the move operation by checking the upload origin. The application still needs content, size, authorization, and destination checks.

### 9. Does `getimagesize()` alone prove that an upload is a valid, harmless image?

- **A.** Yes
- **B.** Yes, if it returns dimensions
- **C.** No; use a suitable decoder and other controls
- **D.** No; PHP cannot read image dimensions

**Answer: C.** The function reports image-related metadata but is not a complete image validator. Decode the expected format with an appropriate maintained library and apply resource limits.

### 10. What can image re-encoding help with?

- **A.** It can normalize output and remove some unrelated metadata or appended data
- **B.** It guarantees the image processor cannot be attacked
- **C.** It eliminates the need for memory and pixel limits
- **D.** It automatically checks user authorization

**Answer: A.** Re-encoding can reduce some risks, but the decoder itself processes untrusted input. Keep it updated and bound its resources.

### 11. Why restrict archive extraction?

- **A.** Archives never contain directories
- **B.** Archives can contain path traversal entries, huge expanded data, or nested files
- **C.** PHP cannot receive archive files
- **D.** Extraction automatically enforces user quotas

**Answer: B.** If the feature must extract archives, validate every entry path and type and limit entry count, nesting, and expanded size.

### 12. Is a random, unguessable download URL a replacement for authorization?

- **A.** Yes, if it has enough characters
- **B.** Yes, if it is not listed in the README
- **C.** No; check authorization for every private download
- **D.** No; private files must never be downloadable

**Answer: C.** URLs can be shared, logged, or leaked. A private download handler must verify that the current user may access the requested record.

### 13. Which response header helps prevent browsers from interpreting a response as a different content type?

- **A.** `X-Content-Type-Options: nosniff`
- **B.** `Content-Length: hidden`
- **C.** `Accept: image/png`
- **D.** `Set-Cookie: nosniff`

**Answer: A.** `nosniff` asks browsers not to MIME-sniff the response. It is a useful serving control, not a substitute for validating and storing files safely.

## Short-answer questions

### 14. Why is an extension allowlist preferable to a denylist?

**Answer:** A denylist tries to anticipate every dangerous or unusual extension and is easy to bypass. An allowlist accepts only the formats needed for the feature and can be compared with server-detected content.

### 15. Why is a MIME type from the browser not authoritative?

**Answer:** The client supplies that value and can choose a false one. Inspect the PHP temporary file on the server, then validate the format according to how the application will process or serve it.

### 16. What is a polyglot file?

**Answer:** It is a file whose bytes can be interpreted as more than one format or contain data meaningful to multiple parsers. Extension, MIME, and signature checks can disagree or be fooled, so validation must consider the intended use and use a suitable parser.

### 17. Why must image processing have pixel and memory limits in addition to a byte limit?

**Answer:** A small compressed image can expand to a very large number of pixels in memory. Bound dimensions and pixel count before decoding, then apply memory and time limits to the processing worker.

### 18. Why store a private file's original name separately from its storage key?

**Answer:** The user-facing name is untrusted metadata and may contain unsafe path or header characters. A generated storage key gives the application a stable internal identifier without using client input as a path.

### 19. What should a download endpoint accept from the client?

**Answer:** It can accept an application-level file record ID, then look up the actual storage key from trusted server-side metadata. It should not accept an arbitrary filesystem path.

### 20. What is the role of a quarantine area?

**Answer:** It keeps new uploads inaccessible while validation, scanning, or review is pending. Only move or mark files as available after the required checks succeed.

## Practical scenarios

### 21. The filename is `avatar.jpg.php`, but Fileinfo reports PHP or generic executable content. What should happen?

**Answer:** Reject it. The extension and detected content do not match the allowed image policy. Do not try to repair the name or store the file under a trusted image extension.

### 22. A PNG has a plausible signature and Fileinfo reports `image/png`, but the image decoder cannot parse it. Should it be stored as an accepted avatar?

**Answer:** No. The decoder failure means the file did not pass the format-specific check. Return a safe validation message and keep the rejected temporary upload from being published or processed further.

### 23. The application stores uploaded PDFs in a public directory and hides the download link from other users. Is access private?

**Answer:** No. A visitor may request the URL directly or receive it from another person. Store private files outside the web root and enforce authorization in the download handler.

### 24. A user changes the file ID in a download URL and receives another user's document. Which check is missing?

**Answer:** The handler is missing an object-level authorization check. It must load the requested file record and verify the signed-in user is permitted to access that specific record on every request.

### 25. A file passed extension, MIME, and signature checks. Can the application skip scanning or parser safeguards?

**Answer:** Not automatically. Those checks do not prove the file is harmless. Use appropriate format parsing, scanning or content disarm for the risk level, and isolate resource-intensive processing where practical.

### 26. A ZIP file is within the compressed-size limit but expands to many gigabytes. Which protection failed?

**Answer:** The application checked only compressed bytes. Archive handling also needs limits on entry count, nesting, expanded size, extraction paths, and processing time; avoid extraction unless the feature requires it.

## Code-reading question

### 27. What security problems remain in this storage code?

~~~php
$destination = $uploadDirectory . '/' . $_FILES['document']['name'];
move_uploaded_file($_FILES['document']['tmp_name'], $destination);
~~~

**Answer:** It uses a client-controlled filename as a path, does not check the upload error or validate the content, does not handle a possible destination collision, and ignores whether the move succeeded. It also does not show authorization, CSRF protection, or whether the directory is safely configured. Use a generated name in a controlled directory only after the required validation and permission checks.

## Key takeaways

- Allowlist the formats required by the feature and compare extensions with server-side content inspection.
- MIME and signature checks are useful signals, not proof that a file is harmless.
- Parse complex formats with maintained tools, re-encode where appropriate, and apply resource limits.
- Generate storage keys and keep private files outside the public web root.
- Serve private files through an authorized handler with safe response headers.
- Limit quotas, bytes, counts, dimensions, archive expansion, processing time, and storage lifetime.
- Use quarantine, scanning, audit logs, and cleanup as additional layers when appropriate.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [094 - Handling file uploads](../lessons/094-handling-file-uploads.md) | [Lesson](../lessons/095-validating-uploads-and-storing-files-safely.md) | [Example](../examples/095-validating-uploads-and-storing-files-safely.md) | [096 - Making HTTP requests with streams and cURL](../lessons/096-making-http-requests-with-streams-and-curl.md) |