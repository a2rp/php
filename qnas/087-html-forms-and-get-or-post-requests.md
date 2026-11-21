# 087 - HTML forms and GET or POST requests: Q&A

## Multiple-choice questions

### 1. Which attribute determines the parameter name PHP receives for a form control?

- **A.** `id`
- **B.** `name`
- **C.** `class`
- **D.** `placeholder`

**Answer: B.** The control's `name` becomes the submitted parameter key. The `id` is useful for connecting a label and control, but it does not name the submitted value.

### 2. Which method is usually appropriate for a search form that retrieves results without changing state?

- **A.** GET
- **B.** POST
- **C.** PUT
- **D.** DELETE

**Answer: A.** GET is appropriate for retrieving a representation, such as search results or a selected page. The query can be bookmarked and shared. The server still validates every parameter.

### 3. Where does a normal URL-encoded POST form send its values?

- **A.** In the URL fragment after `#`
- **B.** In the request body
- **C.** In the response headers
- **D.** In the HTML `id` attributes

**Answer: B.** A normal POST form sends its fields in the request body. PHP parses supported form content types into `$_POST`.

### 4. Does POST encrypt form values or make them private?

- **A.** Yes, by definition.
- **B.** Yes, if the form has `required`.
- **C.** No. HTTPS is needed to protect data in transit, and POST itself is not a secrecy mechanism.
- **D.** No, POST values always appear in the address bar.

**Answer: C.** POST places the form body outside the URL, but it does not encrypt the connection. Use HTTPS and apply appropriate access controls to sensitive data.

### 5. What does PHP normally use for form-encoded POST fields?

- **A.** `$_GET`
- **B.** `$_POST`
- **C.** `$_FILES`
- **D.** `$_SERVER`

**Answer: B.** PHP exposes fields from supported URL-encoded or multipart POST forms in `$_POST`. Uploaded file details are in `$_FILES`.

### 6. What happens to an unchecked checkbox when a standard HTML form is submitted?

- **A.** It is submitted with the value `false`.
- **B.** It is usually omitted from the submitted data.
- **C.** It is submitted as an empty array.
- **D.** It prevents the form from submitting.

**Answer: B.** An unchecked checkbox is generally not a successful control, so its key is absent. The server should handle the missing key deliberately.

### 7. Which statement about `required` and `type="email"` is correct?

- **A.** They make server-side validation unnecessary.
- **B.** They help users in the browser, but a client can bypass them.
- **C.** They encrypt the submitted value.
- **D.** They restrict requests sent by command-line clients.

**Answer: B.** Browser constraints improve user experience. The server must repeat important validation because a request can be constructed without using the form.

## Short-answer questions

### 8. What is the difference between `action` and `method` on a form?

**Answer:** `action` identifies the URL that receives the submission. `method` indicates how the browser sends the form, commonly GET or POST.

**Explanation:** The target script must handle the chosen method and read the values from the corresponding source. The `name` attributes determine the submitted keys.

### 9. Why should a control have a `name` attribute?

**Answer:** A named, enabled, successful control contributes a key-value pair to the submission. Without a `name`, its value is normally not submitted.

**Explanation:** `id` is for document identity and label association; `name` is the form parameter key.

### 10. Does `$_GET` only contain data from GET requests?

**Answer:** No. `$_GET` contains URL query-string values and can be populated even when the HTTP request method is different. Check `$_SERVER['REQUEST_METHOD']` separately if the handler only supports a particular method.

### 11. Which form content types are parsed into `$_POST` by PHP?

**Answer:** PHP parses `application/x-www-form-urlencoded` and `multipart/form-data` POST form submissions into `$_POST`. File metadata is exposed through `$_FILES` when using multipart uploads.

### 12. Where should a PHP script read a JSON request body?

**Answer:** Read the raw body from `php://input` and decode it explicitly. JSON fields are not automatically parsed into `$_POST`.

### 13. Why does using POST not prevent CSRF?

**Answer:** POST specifies where the submitted data is carried; it does not prove who initiated the request or whether the user intended it. State-changing endpoints need appropriate authorization and CSRF protections.

### 14. What is Post/Redirect/Get?

**Answer:** After successfully processing a POST request, the server responds with a redirect to a page loaded using GET. A `303 See Other` is commonly used for this flow.

**Explanation:** The handler should send the redirect before any response body output and call `exit` afterward. This helps avoid accidental resubmission when the visitor refreshes the result page.

## Code and behavior questions

### 15. What key will PHP receive from this control, and what would be submitted if it is checked?

```html
<input type="checkbox" id="security" name="topics[]" value="security">
```

**Answer:** The key is `topics`. If checked, its selected value `security` is included. Because the name uses `[]`, multiple checked controls with the same name can be represented as an array.

**Explanation:** The server must still verify that `$_POST['topics']` is an array and validate each value against an allowlist.

### 16. Identify the problem with this POST handler.

```php
$name = $_POST['name'];
echo '<h1>' . $name . '</h1>';
```

**Answer:** The code assumes the parameter exists, assumes it is a string, and writes it into HTML without escaping. A client can omit it, send an array-shaped value, or supply markup.

**Safer approach:** Check that the key exists, verify `is_string($name)`, validate its rules, and escape it for HTML when rendering. For state-changing operations, also perform authorization and CSRF checks.

### 17. What is wrong with using this URL to submit an order?

```text
/orders.php?create=1&product=42
```

**Answer:** Opening or revisiting a GET URL could create an order, which changes application state. GET should retrieve a representation. Submit an order through an appropriate state-changing request and validate, authorize, and protect the action on the server.

### 18. Why might this redirect fail?

```php
echo '<p>Saved</p>';
header('Location: /thank-you.php', true, 303);
```

**Answer:** The page body was output before the redirect header. PHP headers must be sent before actual output.

**Safer approach:** Process the request, send the `Location` header with the 303 status before rendering anything, and then call `exit`.

### 19. What does this form submit if the checkbox is not selected?

```html
<form method="post">
    <input type="checkbox" name="subscribe" value="yes">
    <button type="submit">Continue</button>
</form>
```

**Answer:** The `subscribe` key is usually absent from `$_POST`. It is not automatically submitted with the string `no` or the boolean `false`.

**Explanation:** The server can interpret a missing key as unchecked, while still handling malformed requests according to its input contract.

### 20. Does using `method="post"` guarantee that the server will only receive form data?

**Answer:** No. A client can send requests directly and choose request headers or body content. The handler should check the method and expected content type when needed, then parse and validate the actual data source.

## Key takeaways

- Use GET for retrieval and POST for data submissions that may change state.
- Form `name` values become parameter keys; omitted controls may not appear at all.
- `$_POST` handles standard form encodings, while JSON bodies need `php://input`.
- Client-side validation helps users but does not replace server-side validation.
- POST is not encryption or CSRF protection; authorization and security checks remain server responsibilities.
- Use Post/Redirect/Get after successful POST processing when appropriate.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [086 - Reading query strings and request data](../lessons/086-reading-query-strings-and-request-data.md) | [Lesson](../lessons/087-html-forms-and-get-or-post-requests.md) | [Example](../examples/087-html-forms-and-get-or-post-requests.md) | [088 - Input validation and normalization](../lessons/088-input-validation-and-normalization.md) |