# 094 - Handling file uploads: Q&A

## Multiple-choice questions

### 1. Which form settings are normally needed to send a file to PHP?

- **A.** `method="get"` and `enctype="text/plain"`
- **B.** `method="post"` and `enctype="multipart/form-data"`
- **C.** `method="dialog"` and `enctype="application/json"`
- **D.** A file input alone; the browser chooses the correct request automatically

**Answer: B.** Browser file uploads normally use `POST` with `multipart/form-data`. The form's `enctype` tells the browser to include the file data in the request.

### 2. Where does PHP expose uploaded file information?

- **A.** `$_COOKIE`
- **B.** `$_FILES`
- **C.** `$_SERVER['DOCUMENT_ROOT']`
- **D.** `$_SESSION`

**Answer: B.** PHP builds the `$_FILES` superglobal for files received in a multipart form request. The input's `name` attribute determines the corresponding key.

### 3. What does `UPLOAD_ERR_OK` mean?

- **A.** The upload completed successfully
- **B.** The user did not select a file
- **C.** PHP could not create a temporary file
- **D.** The uploaded file passed all application security checks

**Answer: A.** It means PHP received the file without a transport-level upload error. The application must still enforce its own content, size, authorization, and storage rules.

### 4. Which upload field contains a browser-provided type claim?

- **A.** `tmp_name`
- **B.** `error`
- **C.** `type`
- **D.** `size`

**Answer: C.** `$_FILES['field']['type']` comes from the request and must not be treated as verified content. Inspect the temporary file on the server instead.

### 5. What is the `accept` attribute on a file input for?

- **A.** Enforcing the server's file-type policy
- **B.** Guiding the browser's file picker
- **C.** Making PHP move the file automatically
- **D.** Replacing CSRF protection

**Answer: B.** `accept` is a user-interface hint. A client can ignore it, so the server must validate every uploaded file.

### 6. What happens to a successful upload's temporary file if the application does not move it?

- **A.** PHP keeps it permanently in the web root
- **B.** PHP removes it when the request ends
- **C.** PHP adds it to `$_SESSION`
- **D.** The browser moves it to the server

**Answer: B.** PHP manages the temporary upload and cleans it up after the request. Move a file during the request if the application needs to retain it.

### 7. Which function moves a file received through PHP's upload mechanism?

- **A.** `copy_upload()`
- **B.** `rename_uploaded_file()`
- **C.** `move_uploaded_file()`
- **D.** `file_put_contents_from_browser()`

**Answer: C.** `move_uploaded_file()` verifies that the source is a valid PHP HTTP upload before moving it. Check its boolean return value because the move can fail.

### 8. Why should an application not use the uploaded `name` directly as a destination path?

- **A.** The browser always sends an empty name
- **B.** It can contain unexpected path characters or collide with another file
- **C.** PHP cannot read string values from `$_FILES`
- **D.** File names are stored only in cookies

**Answer: B.** The name is client-controlled. Use a server-generated filename and keep the original only as carefully escaped display metadata if needed.

### 9. Which configuration setting limits the complete POST request body?

- **A.** `post_max_size`
- **B.** `session.gc_maxlifetime`
- **C.** `default_socket_timeout`
- **D.** `max_execution_time`

**Answer: A.** `post_max_size` limits all data in the POST body, including multipart framing and other form fields. Set it above the per-file `upload_max_filesize` limit.

### 10. Is the hidden `MAX_FILE_SIZE` field a security boundary?

- **A.** Yes; visitors cannot edit hidden fields
- **B.** Yes, but only when the form uses HTTPS
- **C.** No; a client can change or remove it
- **D.** No; PHP never reads it for upload handling

**Answer: C.** The field may help PHP report `UPLOAD_ERR_FORM_SIZE`, but a client controls the request. Enforce the real limit on the server.

### 11. What does `UPLOAD_ERR_NO_FILE` indicate?

- **A.** No file was selected for that input
- **B.** The file exceeded `upload_max_filesize`
- **C.** PHP could not write the temporary upload
- **D.** A PHP extension rejected the file contents

**Answer: A.** It represents no selected file. Treat that as a validation error for a required upload, or as an omitted optional value if the form permits it.

### 12. Can `move_uploaded_file()` overwrite an existing destination file?

- **A.** No, it always refuses
- **B.** Yes
- **C.** Only when the source name is the same
- **D.** Only on Windows

**Answer: B.** The function overwrites an existing target. Generate unique server-side names and handle collisions rather than trusting a user-supplied name.

## Short-answer questions

### 13. Why is `enctype="multipart/form-data"` needed?

**Answer:** It tells the browser to encode the form parts so file bytes can be sent to the server. Without it, PHP does not receive the file in the expected upload structure in `$_FILES`.

### 14. What is the difference between an upload transport check and application validation?

**Answer:** PHP's `error` value reports whether the upload request arrived successfully. It does not prove that the file is an allowed or safe format. The application must still check size, inspect content, apply authorization, and store the file safely.

### 15. Why check that expected `$_FILES` keys exist and have the expected types?

**Answer:** Requests can omit a field, send malformed structures, or use array-shaped input unexpectedly. Defensive shape checks prevent warnings and make the handler reject invalid input before using temporary paths or other values.

### 16. What is different about a multiple-file input named `documents[]`?

**Answer:** PHP returns arrays under `$_FILES['documents']`; each file property has corresponding indexed values. A handler must validate each file entry independently and enforce a total file-count limit.

### 17. Why may `$_FILES` and `$_POST` both be empty after a large upload?

**Answer:** If the complete request exceeds `post_max_size`, PHP may not populate either superglobal. This is different from an ordinary per-file error and should be handled with sensible configuration and a clear application response.

### 18. Why should `post_max_size` be larger than `upload_max_filesize`?

**Answer:** The POST body includes multipart boundaries and other form data in addition to the file. If both limits are equal, a file at the per-file limit can make the complete request exceed the body limit.

### 19. What is the purpose of `is_uploaded_file()`?

**Answer:** It checks whether a path refers to a file uploaded through PHP's HTTP POST mechanism. `move_uploaded_file()` performs its own upload-origin check, so a handler moving the file can rely on that check while still validating the request entry first.

### 20. Why can returning the raw `$_FILES` array to the browser be unsafe?

**Answer:** It can reveal temporary filesystem paths and internal error details. Show a safe, useful message to the user and keep diagnostic information in server-side logs.

## Practical scenarios

### 21. A form has `accept=".pdf"` but the server accepts any file. Is the validation complete?

**Answer:** No. `accept` only guides the file picker. The server must enforce an allowlist and inspect the uploaded content before it accepts or uses the file.

### 22. A handler checks `$_FILES['document']['type'] === 'application/pdf'`. Is that enough?

**Answer:** No. The `type` field is supplied by the client and can be forged. Inspect the temporary file on the server and apply format-specific validation appropriate to the application.

### 23. A visitor submits `../../public/shell.php` as the original filename. What should the application do?

**Answer:** Never append that name to a storage path. Generate an opaque server-side name, store the file in a controlled location, and treat the original name only as untrusted metadata if it must be shown.

### 24. A user can upload a file, but `move_uploaded_file()` returns `false`. What should you check?

**Answer:** Check that the destination directory exists and is writable by the PHP process, that the target path is valid, and that storage permissions and available disk space are correct. Return a safe message instead of exposing the server path.

### 25. A protected profile form uploads a new avatar. What checks still apply besides file validation?

**Answer:** Authenticate the request, authorize the current user to update that profile, validate a CSRF token, and enforce upload limits and content rules. File validation does not grant permission to change another user's profile.

### 26. Why should an upload endpoint check the result of `move_uploaded_file()`?

**Answer:** A valid temporary upload does not guarantee that storage will succeed. The destination may be unwritable, invalid, or unavailable. Only report success after the move returns `true`.

## Code-reading question

### 27. What is wrong with this check?

~~~php
if ($_FILES['document']['error']) {
    echo 'Upload failed';
}
~~~

**Answer:** It treats the integer error code as a generic boolean and does not check whether the field or `error` key exists. A safer starting point is:

~~~php
if (
    !isset($_FILES['document']['error'])
    || !is_int($_FILES['document']['error'])
    || $_FILES['document']['error'] !== UPLOAD_ERR_OK
) {
    // Reject the upload or handle its specific error.
}
~~~

`UPLOAD_ERR_OK` is `0`, so success should be checked explicitly. After that, the application still needs its own validation.

## Key takeaways

- `multipart/form-data` and `POST` are the normal browser upload mechanism.
- `$_FILES` values require defensive checks; browser-provided names and types are not trusted.
- Handle `UPLOAD_ERR_*` values explicitly and distinguish no selection from other failures.
- PHP's temporary upload must be moved during the request if it needs to persist.
- Configure per-file, total-request, file-count, proxy, and storage limits together.
- Server-side content validation, authorization, CSRF protection, safe naming, and controlled storage remain necessary.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [093 - Authentication and authorization](../lessons/093-authentication-and-authorization.md) | [Lesson](../lessons/094-handling-file-uploads.md) | [Example](../examples/094-handling-file-uploads.md) | [095 - Validating uploads and storing files safely](../lessons/095-validating-uploads-and-storing-files-safely.md) |