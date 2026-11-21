# Apache, Nginx, and PHP-FPM integration

A PHP web application commonly uses Apache or Nginx to handle HTTP and PHP-FPM to run PHP scripts. The web server serves static files, applies routing and access rules, and passes approved PHP requests to FPM through FastCGI.

Understanding which process handles each step makes configuration and troubleshooting easier. Verify operating-system paths, socket names, and server directives before deploying.

## Learning objectives

After this lesson, you should be able to:

- trace a request through the web server, FastCGI, and PHP-FPM;
- configure a public document root and PHP front controller;
- distinguish a Unix socket from a TCP listener;
- recognize script-path and socket-permission problems;
- reason about FPM worker limits and monitoring;
- validate and reload server configuration safely.
## Request flow

A typical request works like this:

1. The browser sends an HTTP request to Apache or Nginx.
2. The web server handles TLS, static files, URL routing, and access controls.
3. For a permitted PHP request, it sends FastCGI parameters and a script path to PHP-FPM.
4. An FPM worker runs the script with its configured PHP runtime.
5. PHP returns a response to the web server over FastCGI.
6. The web server sends the HTTP response to the browser.

FastCGI is a protocol between the web server and PHP process. It is not HTTP and should not be exposed as a public endpoint.

PHP-FPM manages pools of PHP workers. It does not replace the web server: it does not accept ordinary browser HTTP traffic or serve application static assets. Apache's FastCGI proxy forwards requests; a service manager or container platform manages the FPM process separately.

## Use a public document root

Point the web server at the application's public directory, not the repository root. Only files intended to be reachable by URL should be in that directory.

~~~text
/srv/example-app/
    src/
    vendor/
    config/
    storage/
    public/
        index.php
        assets/
~~~

Here the web root is **/srv/example-app/public**. Source files, Composer metadata, private configuration, and storage stay outside it. Frameworks commonly route application URLs through the public index file.

Keep private uploads outside the document root. If files must be publicly downloadable, serve only intended files and ensure uploaded content cannot be executed as PHP.

## Nginx with PHP-FPM

This simplified server block serves existing static files and sends application routes to one front-controller script. It assumes the FPM Unix socket is **/run/php/php-fpm.sock** and the Nginx worker can connect to it.

~~~nginx
server {
    listen 80;
    server_name example.test;

    root /srv/example-app/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /index.php {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/run/php/php-fpm.sock;
    }

    location ~ [.]php$ {
        return 404;
    }
}
~~~

The root location serves an existing static file or directory and otherwise routes the request to the front controller. The exact match for the front controller passes that script to FastCGI. The final rule rejects other PHP paths, which is useful for a front-controller app and helps prevent accidental execution of unrelated scripts in the public directory.

The **SCRIPT_FILENAME** parameter must name a real script path that the FPM process can read. Nginx and FPM must see the application at compatible filesystem paths. Different container mount paths are a common cause of “Primary script unknown” errors.

The **fastcgi_params** file varies by package. Check that required parameters are present and avoid conflicting definitions. PHP needs the script filename and query string; normal POST requests also need request method and content metadata. If an app intentionally runs multiple scripts, permit only required paths and verify each file exists before forwarding it.
## Apache with PHP-FPM

Apache can proxy PHP requests to FPM using its proxy and FastCGI modules. This fragment uses a Unix socket and a public document root.

~~~apache
<VirtualHost *:80>
    ServerName example.test
    DocumentRoot /srv/example-app/public
    DirectoryIndex index.php

    <Directory /srv/example-app/public>
        Require all granted
        Options -Indexes
        AllowOverride None
    </Directory>

    <FilesMatch "index[.]php$">
        SetHandler "proxy:unix:/run/php/php-fpm.sock|fcgi://localhost/"
    </FilesMatch>
</VirtualHost>
~~~

The Apache installation must have the required proxy modules enabled, and the socket path must match the FPM pool listener. With **AllowOverride None**, put rewrite rules in the virtual-host configuration rather than a per-directory override file. Apache supports other PHP handlers and integration modes; do not mix directives from different modes without checking their setup.

## FPM listeners and access

FPM can listen on a Unix-domain socket or a TCP address.

- A **Unix socket** is local to a host or shared filesystem namespace. Its owner, group, and mode control which processes can connect.
- A **TCP listener** can connect separate containers or hosts. Restrict it to a private network and allow only trusted clients.

For a local web server and FPM, a Unix socket is often a good choice. The web-server worker needs permission to connect, but unrelated users should not. If TCP is needed, bind to a private interface and configure an allowlist as appropriate. Never expose a FastCGI listener to the public internet: a client that can connect can send parameters that influence script execution.

In containers, keep FPM on the private application network and do not publish its port publicly. Confirm that the web server and FPM see the application files at the script path passed over FastCGI.

## FPM pools, capacity, and monitoring

An FPM pool defines a set of worker processes and runtime settings. Pools can use different users, groups, listeners, and environment settings. Separate pools can help with operational isolation, but a pool alone is not a complete security boundary.

FPM supports three process-management modes:

- **Static** keeps a configured number of workers running.
- **Dynamic** adjusts worker count within configured limits.
- **On-demand** starts workers when requests arrive and stops idle workers after a timeout.

Choose based on measured memory, request volume, startup time, and latency. Do not copy another server's worker counts. Estimate memory per worker under realistic requests, reserve capacity for the operating system and other services, then load-test.

Too few workers can leave requests waiting when all children are busy. Too many can exhaust memory or overload the database. Monitor busy and idle workers, listen queue, slow requests, memory, response times, and application errors. FPM status and slow-log endpoints are operational data; restrict access to administrators or internal monitoring.

A setting that recycles workers after a number of requests can limit the impact of gradual memory growth, but it does not replace fixing a leak.

## Filesystem permissions

The web-server and FPM processes may run as different users. Check that:

- the web server can read public assets and connect to the FPM socket;
- FPM can read application code and required configuration;
- only necessary storage, cache, or upload directories are writable by FPM;
- private configuration and data are not readable by unrelated users.

Do not make the whole application tree world-writable to fix a permission error. Correct the owner, group, socket mode, or directory permissions for the actual processes.
## Validate and reload

Before applying a configuration change, use the syntax checker for the installed server. Common Linux commands are:

~~~sh
nginx -t
apachectl configtest
~~~

A syntax check does not prove the socket exists, permissions are correct, or the application works. Check service status and logs after a graceful reload. Then request a static asset, the front controller, a POST route, and an intentionally missing PHP path.

Use a graceful reload where supported so active requests can finish. Confirm that new workers use the intended configuration. PHP-FPM is supervised separately from Apache or Nginx, so confirm that its service is running too.

## Common failure symptoms

- **502 Bad Gateway:** FPM is unavailable, the socket path is wrong, or socket permissions prevent a connection.
- **Primary script unknown:** the FastCGI script path does not match the path visible to FPM, or the file is missing.
- **403 Forbidden:** the web-server user cannot read a directory or a rule denies access.
- **Application routes return 404:** the front-controller fallback or rewrite rule is missing or incorrect.
- **Static files work but PHP does not:** inspect the FastCGI handler, required parameters, listener, and both services' logs.
- **Requests queue or time out under load:** inspect FPM worker capacity, slow requests, memory, and downstream services.

Web-server logs often show proxy and socket failures. FPM and application logs can show PHP startup or runtime failures. Do not display stack traces or configuration details to public users.

## Key takeaways

- Apache or Nginx handles HTTP; PHP-FPM runs PHP workers; FastCGI connects them.
- Keep the document root limited to public files and route PHP execution narrowly.
- Make the script filename parameter, FPM listener, and filesystem paths agree.
- Restrict FPM to trusted local or private-network clients.
- Set permissions and worker capacity from measured service needs.
- Validate configuration, reload gracefully, and check both web-server and FPM logs.
- A passing syntax check still needs end-to-end request verification.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [141 - Deployment environments and release workflows](141-deployment-environments-and-release-workflows.md) | [Q&A](../qnas/142-apache-nginx-and-php-fpm-integration.md) | [Example](../examples/142-apache-nginx-and-php-fpm-integration.md) | [143 - Production configuration, permissions, and logging](143-production-configuration-permissions-and-logging.md) |