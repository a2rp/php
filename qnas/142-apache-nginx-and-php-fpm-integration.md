# Apache, Nginx, and PHP-FPM integration — Q&A

These questions review the request path, server configuration, FastCGI parameters, FPM listeners, worker capacity, permissions, and troubleshooting. Each answer follows directly after its question.

## Multiple-choice questions

### 1. What is the role of PHP-FPM in a common PHP web stack?

- A. It accepts browser HTTP requests and serves static files
- B. It manages PHP worker processes and executes scripts received over FastCGI
- C. It replaces the database server
- D. It compiles the application during every request

**Answer: B.**

**Explanation:** Apache or Nginx handles HTTP and forwards permitted PHP requests. FPM manages PHP workers that execute those scripts.

### 2. What protocol commonly connects Apache or Nginx to PHP-FPM?

- A. SMTP
- B. FastCGI
- C. WebSocket
- D. DNS

**Answer: B.**

**Explanation:** FastCGI carries script and request parameters between the web server and the PHP process manager. It is not the browser-facing HTTP protocol.

### 3. Where should the web server's document root normally point?

- A. The repository root
- B. The application source directory
- C. The application's public directory
- D. The vendor directory

**Answer: C.**

**Explanation:** The public directory contains files intended to be reachable by URL. Source, private configuration, dependencies, and storage should remain outside the public web root.

### 4. Which FastCGI parameter tells PHP which script file to execute?

- A. SCRIPT_FILENAME
- B. REQUEST_METHOD
- C. CONTENT_TYPE
- D. HTTP_HOST

**Answer: A.**

**Explanation:** The web server must pass the actual filesystem path to the script. The path must be readable at the same location from the FPM process's filesystem view.

### 5. Why can a front-controller Nginx configuration reject other PHP paths?

- A. To stop PHP-FPM from accepting any requests
- B. To reduce accidental execution of unrelated or uploaded PHP files
- C. To disable static files
- D. To bypass FastCGI

**Answer: B.**

**Explanation:** Many applications need only their public index script to execute. Restricting PHP execution to that entry point reduces risk from other files in the public directory.

### 6. Which statement about a Unix-domain socket is correct?

- A. It is always reachable from the public internet
- B. Its filesystem ownership and permissions can control which local processes connect
- C. It is an HTTP URL
- D. It removes the need to run PHP-FPM

**Answer: B.**

**Explanation:** A local web server and FPM can communicate through a Unix socket. The socket path, owner, group, and mode must allow the correct processes to connect.
### 7. What is a safe default for an FPM TCP listener?

- A. Bind it publicly so any client can submit FastCGI requests
- B. Keep it on a private interface or network and allow only trusted clients
- C. Disable every firewall
- D. Use it as a public health endpoint

**Answer: B.**

**Explanation:** A FastCGI client can send parameters that influence script execution. FPM must not be reachable by untrusted networks.

### 8. What is likely when all FPM workers are busy and new requests arrive?

- A. Requests may wait in a queue or time out
- B. PHP-FPM automatically adds unlimited memory
- C. Nginx starts running PHP directly
- D. The database changes its schema

**Answer: A.**

**Explanation:** FPM worker capacity limits concurrent PHP request handling. The web server and FPM listener may queue requests, but excessive waiting can lead to timeouts.

### 9. Which metrics help show whether FPM worker capacity is exhausted?

- A. The number of CSS files in the project
- B. Busy and idle worker counts and the FPM listen queue
- C. The number of Git branches
- D. The browser's screen resolution

**Answer: B.**

**Explanation:** Worker state and queue length help show whether PHP requests are waiting for available FPM children. Also inspect memory, latency, slow requests, and downstream services.

### 10. Which statement about Apache's FastCGI proxy module is correct?

- A. It necessarily starts and supervises PHP-FPM
- B. It forwards requests to a FastCGI backend, while FPM must be managed separately
- C. It is the same thing as PHP-FPM
- D. It serves only images and stylesheets

**Answer: B.**

**Explanation:** The Apache proxy module handles forwarding. A service manager, container platform, or other process supervisor must manage the FPM service.

## Short-answer questions

### 11. Describe the path of a PHP web request from the browser to the response.

**Answer:** The browser sends HTTP to Apache or Nginx. The web server applies routing and access rules, serves static content or forwards an allowed PHP script over FastCGI, FPM executes it, and the web server returns the response.

**Explanation:** Knowing this boundary helps identify whether a failure belongs to HTTP routing, FastCGI connectivity, the FPM pool, or application code.

### 12. Why should source code, Composer metadata, and private storage stay outside the document root?

**Answer:** They are not intended to be downloaded directly by users and may contain code, configuration, or data that should remain private.

**Explanation:** A public-only document root reduces exposure if a routing or server rule is misconfigured. Keep downloadable files limited to content deliberately made public.

### 13. Why must the script path be visible to both Nginx and PHP-FPM?

**Answer:** Nginx sends the script path to FPM, and FPM must be able to read and execute that path.

**Explanation:** If containers mount the application at different locations, an apparently valid Nginx path may not exist inside the FPM container. Align mounts and the FastCGI script filename.

### 14. What should be checked when a POST request reaches PHP without its expected body or method information?

**Answer:** Check that the web server passes the request method, content type, content length, and request body through FastCGI.

**Explanation:** FastCGI parameter includes can vary by package. A configuration that works for a simple GET may omit metadata needed for POST handling.

### 15. What is the difference between static, dynamic, and on-demand FPM process management?

**Answer:** Static keeps a fixed worker count, dynamic adjusts the count within configured limits, and on-demand starts workers when requests arrive and stops idle workers after a timeout.

**Explanation:** Each mode has different memory, startup, and capacity trade-offs. Choose and tune it from workload measurements rather than copying arbitrary values.

### 16. Why should an FPM status page or slow log be restricted?

**Answer:** They reveal operational details such as worker activity, request state, or traces that should not be public.

**Explanation:** Limit access to administrators or internal monitoring. Operational visibility is useful, but public exposure can leak details about application behavior.

### 17. What should a server configuration syntax check prove, and what does it not prove?

**Answer:** It should confirm that the configuration parses. It does not prove the FPM socket exists, permissions are correct, PHP executes, or application routes work.

**Explanation:** After validation and reload, test representative static, front-controller, POST, and missing-script requests and inspect service logs.
## Scenarios and debugging

### 18. Nginx returns 502 Bad Gateway for every PHP request, but static files load. What should you inspect first?

**Answer:** Check whether PHP-FPM is running, whether the configured socket or TCP address is correct, whether Nginx can connect to it, and whether the FPM listener is accepting connections.

**Explanation:** Static files do not require FPM. A 502 for PHP usually points to an upstream connection, listener, or permission problem.

### 19. Nginx and FPM both run in containers. The application exists at different paths in each container, and FPM reports that the script cannot be found. What is the likely issue?

**Answer:** The SCRIPT_FILENAME sent by Nginx points to a path that does not exist inside the FPM container.

**Explanation:** Both services need compatible application mounts, or the FastCGI path must be translated correctly. Verify the path from within the FPM container.

### 20. A user uploads a file named profile.php into a public uploads directory. Why should the server avoid executing it?

**Answer:** Uploaded content is untrusted and could contain executable PHP code. The server should store private uploads outside the document root or explicitly prevent PHP execution in public upload locations.

**Explanation:** A generic PHP handler that executes every existing file beneath the web root can turn an upload feature into code execution.

### 21. A local FPM socket exists, but Nginx still receives permission-denied errors. What should be checked?

**Answer:** Check the socket owner, group, mode, the FPM pool's listener settings, and the user or group of the Nginx worker.

**Explanation:** The socket must allow the web-server process to connect without granting access to unrelated users. Do not fix this by making the entire application world-writable.

### 22. Requests slow down under load, and the FPM listen queue grows. What should the operator consider before increasing the worker limit?

**Answer:** Measure memory per worker and check database and downstream-service capacity, request duration, slow logs, and host memory.

**Explanation:** More workers can raise concurrency, but they also consume memory and can overload dependencies. Tune and load-test against the actual bottleneck.

### 23. The server configuration test succeeds, but every framework route returns 404. What should be checked?

**Answer:** Check the document root, index or front-controller rule, application rewrite configuration, and whether the web server forwards the fallback route correctly.

**Explanation:** Syntax validation does not verify routing behavior. Test a real application route and inspect the web-server access and error logs.

### 24. An administrator plans to expose the FPM TCP port to all network interfaces to make remote Nginx connect. What safer change should they make?

**Answer:** Put the services on a private network, bind FPM to a private interface, and restrict clients with network controls and FPM listener settings.

**Explanation:** A public FastCGI listener lets untrusted clients send execution parameters. FPM should be reachable only by the intended web-server service.

## Practice exercise

### 25. Review a PHP deployment using Nginx and FPM. The app's public directory is /srv/shop/public, FPM listens on a Unix socket, and the app uses one front controller. Name five checks before releasing the configuration.

**Answer:**

1. Confirm that Nginx's document root is /srv/shop/public.
2. Confirm that the front-controller fallback sends requests to the intended index script.
3. Confirm that SCRIPT_FILENAME resolves to a real path readable by FPM.
4. Confirm that the configured socket path matches the FPM pool and that socket permissions allow the Nginx worker to connect.
5. Confirm that other PHP paths are denied or narrowly allowed, and that uploads cannot execute as PHP.
6. Run the Nginx syntax test, reload gracefully, and check static, application, POST, and missing-script requests.
7. Inspect Nginx, FPM, and application logs and verify worker capacity under expected load.

**Explanation:** These checks cover path mapping, routing, execution scope, process connectivity, and end-to-end behavior. A passing syntax test alone is not enough.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [141 - Deployment environments and release workflows](../lessons/141-deployment-environments-and-release-workflows.md) | [Lesson](../lessons/142-apache-nginx-and-php-fpm-integration.md) | [Example](../examples/142-apache-nginx-and-php-fpm-integration.md) | [143 - Production configuration, permissions, and logging](../lessons/143-production-configuration-permissions-and-logging.md) |