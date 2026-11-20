# 002 - How PHP Fits into a Web Application

PHP is often used on a web server to create the response a browser asks for. To understand PHP in a larger application, follow a request from the browser to the server and back again.

This chapter explains the main parts of that journey. It also separates PHP's role from the roles of HTML, CSS, JavaScript, the web server, and a database.

## What you will learn

- What a web application is
- What the browser, web server, PHP runtime, and application each do
- How an HTTP request becomes a response
- How static and dynamic responses differ
- Where databases and browser-side code fit
- Why request data needs careful handling

## What is a web application?

A **website** is a set of pages or other resources available over the web. A **web application** usually responds to requests with behavior that depends on a route, user input, stored data, or application rules. The terms overlap: a site can have both informational pages and interactive application features.

A web application is made from cooperating parts. PHP is one possible server-side language in that system. PHP does not replace the browser, the web server, HTML, CSS, JavaScript, or a database. Each has a different job.

## The main parts

### Browser

The browser is the visitor's client. It sends requests, receives responses, and displays HTML. It can also load CSS, images, fonts, and JavaScript files. JavaScript can add behavior that runs in the browser.

### Web server

A web server receives HTTP requests and sends HTTP responses. It can return a static file, such as an image or stylesheet, or route a request to application code. Common server software includes Apache and Nginx. PHP also provides a small development server for local practice.

### PHP runtime

The PHP runtime executes PHP source code. A web server is configured to pass PHP requests to a PHP runtime, then use the result when forming the response. The details of this connection depend on the server setup.

### PHP application

The application contains the code and rules for the site's behavior. It may decide which page or action a URL represents, check input, read or change data, and prepare a response. Larger applications often use a framework to organize this work, but PHP applications can also be written without one.

### Database and other services

An application may read or save information in a database, or communicate with another service. These are optional parts of an application. A simple page can be generated without a database, and PHP can return a response without calling an external service.

## Follow one request

Imagine a visitor opens a greeting page. A simplified request-response journey looks like this:

1. The browser requests a URL from the web server.
2. The request contains details such as a method, a path, headers, and sometimes a body or query string.
3. The web server decides how to handle the path. It may return a static file or pass the request into the PHP application.
4. PHP runs the application logic. It may use request data and, when needed, read or update stored information.
5. The application produces a response. The response has a status, headers, and a body. The body might contain HTML, JSON, or plain text.
6. The web server sends the response to the browser.
7. The browser interprets the response. For an HTML page, it displays the document and may request additional resources such as CSS, images, and JavaScript.

A simplified diagram:

~~~text
Visitor
  | opens a page
  v
Browser -- HTTP request --> Web server
                              |
                       PHP application
                       /      |       \
                  PHP logic  Database  Other service
                       \      |       /
Browser <-- HTTP response -- Web server
~~~

The diagram leaves out deployment and network details so the main responsibilities are easier to see.

## A request and response example

A browser might send a request like this:

~~~http
GET /greeting?name=Asha HTTP/1.1
Host: example.test
Accept: text/html
~~~

This is an illustrative request. The path is **/greeting** and the query string contains a value named **name**. PHP can make query-string values available to the script, but request input is supplied from outside the application and must be treated as untrusted. Later chapters cover reading and validating request data safely.

The application might produce a response like this:

~~~http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<!doctype html>
<html lang="en">
<body>
    <h1>Hello, Asha!</h1>
</body>
</html>
~~~

The response status says the request succeeded. The Content-Type header tells the browser what kind of body follows. The body contains the HTML to display. In an actual application, the greeting value should be validated and escaped for its output context before it is placed in HTML.

## Static and dynamic responses

A **static resource** is returned without running application logic for each requested value. An image file is a common example. A stylesheet can also be served as a static resource.

A **dynamic response** is produced by executing application logic. PHP might use a route, input, or saved information to choose the content. Two requests can therefore receive different responses even when they reach the same PHP application.

Static and dynamic content can exist in the same application. The web server may serve an image directly while sending a page request to PHP.

## Where HTML, CSS, and JavaScript fit

These technologies work together but do different jobs:

- **PHP** commonly runs on the server and can create a response.
- **HTML** describes the structure and meaning of page content.
- **CSS** controls how the page is presented.
- **JavaScript** can add behavior in the browser. It can also run in server environments, but this course's web-page examples focus on browser-side JavaScript.

PHP can generate HTML that includes links to CSS and JavaScript files. The browser receives the generated HTML and then loads the resources it references. A page can also return JSON for a browser script or another client to use.

## A database is optional

Many applications store information in a database, but PHP does not require one. A simple PHP page can return a fixed message. When an application does use a database, PHP code can request data and use the result to create a response.

Database access introduces additional concerns, including credentials, permissions, and safe queries. Those topics belong to later chapters. Do not assume that every PHP request must contact a database.

## Important boundaries

- PHP normally runs on the server in a PHP web application; the browser receives its output.
- The web server handles HTTP connections and selects how a request is served.
- The PHP application decides what the request means and what response to produce.
- The browser displays the response and runs browser-side code such as JavaScript.
- A database or framework may be part of an application, but neither is required for all PHP programs.
- Query strings and form data are user input. They are not trustworthy just because PHP can read them.
- A URL path does not have to match a physical file name. An application can map routes to code.

## Common misunderstandings

| Misunderstanding | Better mental model |
| --- | --- |
| PHP is the web server. | PHP is a language and runtime; a web server receives HTTP requests and works with the runtime. |
| The browser runs PHP. | PHP normally executes before the response reaches the browser. |
| Every page request needs a database. | The application may use a database, but static or simple responses do not require one. |
| Every URL is a file on disk. | A web server or application can map a route to code without a matching file. |
| Anything in a query string is safe to print. | Request values come from outside the application and need validation and context-appropriate output handling. |

## Try it yourself

1. Name the four main parts in a basic PHP web request: client, web server, PHP runtime, and application.
2. Put the request-response steps in order, from the browser requesting a page to the browser displaying it.
3. Explain one difference between a static image response and a dynamic PHP response.
4. In the sample request, identify the path and the query-string value.
5. Name two kinds of response body PHP can produce.
6. Explain why a PHP application can work without a database.
7. Describe which technology usually structures a page, which styles it, and which can add browser-side behavior.

## Key points

- A PHP web application is one part of a request-response system.
- The browser requests a resource, the server routes the request, PHP may execute application logic, and a response returns to the browser.
- Responses have a status, headers, and a body.
- PHP can create HTML, JSON, or other output.
- Web servers can serve static resources and route dynamic requests to PHP.
- Databases and frameworks are useful in many applications, but are optional.
- All request data must be treated as untrusted until the application handles it appropriately.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [001 - Introduction to PHP](001-introduction-to-php.md) | [Open Q&A](../qnas/002-how-php-fits-into-a-web-application.md) | [Open example](../examples/002-how-php-fits-into-a-web-application.md) | [003 - PHP releases, versions, and compatibility](003-php-releases-versions-and-compatibility.md) |
