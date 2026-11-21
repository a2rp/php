# 094 - Handling file uploads

## Learning goals

By the end of this lesson, you will be able to:

- Build an HTML form that sends a file to PHP.
- Read PHP's `$_FILES` structure without assuming client input is valid.
- Handle the upload error codes and distinguish a missing file from a failed upload.
- Explain PHP's temporary-file lifecycle and move a received file during the request.
- Configure request and file-size limits consistently.
- Describe the security checks required before accepting and storing an uploaded file.

## 1. How a browser sends a file

A file upload normally uses an HTTP `POST` request with the form encoding `multipart/form-data`. The file input's `name` becomes the key PHP uses in `$_FILES`.

```html
<form action="/upload.php" method="post" enctype="multipart/form-data">
    <label for="document">Choose a plain-text document</label>
    <input
        id="document"
        name="document"
        type="file"
        accept=".txt,text/plain"
        required
    >
    <button type="submit">Upload</button>
</form>
```

Without `enctype="multipart/form-data"`, the browser does not send the file contents in the format PHP expects. A file input's `accept` attribute only guides the browser's file picker. A user or custom client can ignore it, so PHP must enforce its own rules.

For an authenticated form, include the application's CSRF protection as well. A file upload is still a state-changing request.

## 2. What PHP puts in `$_FILES`

For the field named `document`, PHP provides an entry similar to this:

```php
$_FILES['document'] = [
    'name' => 'notes.txt',
    'full_path' => 'notes.txt',
    'type' => 'text/plain',
    'tmp_name' => '/temporary/php/path',
    'error' => UPLOAD_ERR_OK,
    'size' => 248,
];
```

The fields have different levels of trust:

| Field | Meaning | How to treat it |
| --- | --- | --- |
| `name` | Name supplied by the client | Untrusted display text; never use it as a destination path |
| `full_path` | Client-provided relative path where supported | Untrusted; do not recreate the client directory structure |
| `type` | Type claimed by the browser | Untrusted; inspect the temporary file on the server |
| `tmp_name` | Temporary path created by PHP | Use only after validating the upload entry and error |
| `error` | PHP's upload result code | Check it before reading or moving the file |
| `size` | Size PHP recorded for the received file | Enforce an application limit; do not rely on a form field |

A request may omit the expected field or contain a malformed structure. Check that `$_FILES['document']` is an array, that expected keys exist, and that scalar values have the types you expect. Do not assume every request came from your own HTML form.

### Several files

A multiple-file input can use `name="documents[]"`. PHP then places arrays of names, temporary paths, sizes, and error codes under that field. The values at the same numeric index belong to the same file.

Validate each entry independently. Do not blindly loop over the client-submitted shape, and apply both a per-file limit and a reasonable count limit.

## 3. Check the upload error first

The `error` value tells you whether PHP received the file successfully:

| Constant | Meaning |
| --- | --- |
| `UPLOAD_ERR_OK` | The upload completed |
| `UPLOAD_ERR_INI_SIZE` | The file exceeded `upload_max_filesize` in PHP configuration |
| `UPLOAD_ERR_FORM_SIZE` | The file exceeded the form's `MAX_FILE_SIZE` value |
| `UPLOAD_ERR_PARTIAL` | Only part of the file arrived |
| `UPLOAD_ERR_NO_FILE` | No file was selected |
| `UPLOAD_ERR_NO_TMP_DIR` | PHP could not find its temporary directory |
| `UPLOAD_ERR_CANT_WRITE` | PHP could not write the temporary file |
| `UPLOAD_ERR_EXTENSION` | A PHP extension stopped the upload |

`UPLOAD_ERR_OK` is the only success value. Do not test the error as a generic true or false value: success is the integer `0`.

The form's hidden `MAX_FILE_SIZE` field may let PHP report `UPLOAD_ERR_FORM_SIZE`, but clients can change or remove it. It is not an upload security boundary. The server's configured limit and your own server-side size check are authoritative.

`UPLOAD_ERR_NO_FILE` is different from a failed transfer. Decide whether no selection is allowed by the form's purpose. For a required upload, report that the user must select a file; for an optional upload, continue without a file.

## 4. PHP's temporary-file lifecycle

After a successful request, `$_FILES['document']['tmp_name']` points to a temporary file managed by PHP. It is not a permanent upload location. Process or move it during the request; PHP removes an unmoved temporary upload when the request ends.

`is_uploaded_file()` can confirm that a path came from PHP's HTTP upload mechanism. `move_uploaded_file()` checks this condition itself and moves a valid PHP upload to the destination you choose.

```php
if (!is_uploaded_file($temporaryPath)) {
    throw new RuntimeException('The temporary file is not a valid upload.');
}

if (!move_uploaded_file($temporaryPath, $destinationPath)) {
    throw new RuntimeException('The uploaded file could not be stored.');
}
```

Check the return value. `move_uploaded_file()` can fail because of permissions, an invalid destination, or another filesystem problem. It overwrites an existing destination file, so choose a server-generated unique name and avoid collisions. Never concatenate the client-supplied filename into the destination path.

## 5. Apply server-side limits

Uploads use several limits that work together:

- `upload_max_filesize` limits each uploaded file.
- `post_max_size` limits the entire POST body, so it must be larger than the file limit to allow for form data and multipart boundaries.
- `max_file_uploads` limits how many files PHP accepts in one request.
- `upload_tmp_dir` chooses PHP's temporary directory and must be writable by the PHP process.
- `max_input_time` limits how long PHP can receive request input.
- The web server, reverse proxy, or hosting platform may impose another request-body limit.

Choose limits that fit the application and configure the layers consistently. If `post_max_size` is exceeded, PHP may leave both `$_POST` and `$_FILES` empty, so the application cannot handle it like an ordinary per-file upload error. Show a clear, safe response and keep server-side logs useful without exposing filesystem paths.

Do not increase limits without considering memory, storage, processing time, concurrent uploads, and available disk space.

## 6. Validate before storing or using the file

An upload is untrusted input. Before accepting it:

1. Check the HTTP method, expected field, structure, and PHP upload error.
2. Enforce a server-side size limit and any file-count limit.
3. Use a narrow allowlist of file types the feature actually needs.
4. Inspect content on the server. Do not trust the original filename, extension, browser `type`, or hidden form fields.
5. Generate a random server-side filename. Keep the original name only as escaped display metadata if the product needs it.
6. Store uploads outside the public document root when possible. If they must be web-accessible, prevent execution and serve them through controlled application logic.
7. Require authentication, authorization, and CSRF protection where the feature changes a user's data.
8. Consider malware scanning, quotas, and safe download headers for the application's risk level.

MIME detection is one signal, not proof that a file is harmless. Image parsing, archive extraction, office documents, and other complex formats need format-specific safeguards. The next chapter focuses on validating uploads and storing files safely.

## 7. Return useful errors without leaking internals

Tell the user what they can do next: select a file, choose an accepted type, or try a smaller file. Log useful diagnostic details on the server when needed, but do not expose temporary paths, server directory names, stack traces, or raw request data in the response.

A detailed list of error codes helps the application decide what happened. The public message can remain simple, for example, “The upload could not be completed. Please try again.”

## Key takeaways

- Use `POST` and `multipart/form-data` for normal browser file uploads.
- Read the expected entry from `$_FILES` defensively and check `error` before processing.
- PHP stores successful uploads temporarily; move a file during the request if it must persist.
- Enforce size, count, and content rules on the server; browser hints and hidden fields are not security controls.
- Use server-generated filenames and controlled storage, and never trust the client's filename or MIME claim.
- Authentication, authorization, and CSRF protection still apply to upload endpoints.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [093 - Authentication and authorization](093-authentication-and-authorization.md) | [Q&A](../qnas/094-handling-file-uploads.md) | [Example](../examples/094-handling-file-uploads.md) | [095 - Validating uploads and storing files safely](095-validating-uploads-and-storing-files-safely.md) |