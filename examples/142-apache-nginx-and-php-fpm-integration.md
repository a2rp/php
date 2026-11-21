# Apache, Nginx, and PHP-FPM integration — Example

This example connects a small PHP front controller to PHP-FPM using a local Unix socket. It shows an Nginx configuration and an Apache alternative, then checks that PHP executes, static content is served, and an unrelated PHP path is rejected.

These Linux configurations are starting points, not universal server files. Adjust the PHP-FPM socket, service names, ownership, modules, and application paths to match the installed system. Do not expose the FastCGI listener publicly.

## Question

A small application lives at **/srv/shop**, and its public directory is **/srv/shop/public**. How can the web server pass the front-controller request to FPM while serving a static asset directly and denying other PHP paths?

## Example files

~~~text
/srv/shop/
    public/
        index.php
        status.txt
        assets/
            note.txt
~~~

Create the front controller at **public/index.php**:

~~~php
<?php

declare(strict_types=1);

header('Content-Type: text/plain; charset=utf-8');

echo 'Request executed by PHP-FPM.' . PHP_EOL;
~~~

Create a static test asset at **public/assets/note.txt**:

~~~text
Static asset served by the web server.
~~~

The front controller returns a stable response so that the HTTP request can be distinguished from static-file handling. In a real application, the front controller usually bootstraps the framework and routes the request.

## PHP-FPM pool socket

The pool must listen on the same socket that the chosen web server uses. A simplified pool fragment could be:

~~~ini
[shop]
user = www-data
group = www-data
listen = /run/php/shop-fpm.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0660

pm = dynamic
pm.max_children = 8
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 4
~~~

The example assumes both services use the **www-data** account or group for the socket connection. Confirm the actual web-server worker user and set the narrowest socket permissions that allow access.

The worker counts are example values only. Size them from measured memory per PHP worker, expected concurrency, host capacity, and database limits. The socket's parent directory must also permit the services to reach the socket.

## Nginx configuration

Use this server block when Nginx is the HTTP server:

~~~nginx
server {
    listen 80;
    server_name shop.example.test;

    root /srv/shop/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /index.php {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/run/php/shop-fpm.sock;
    }

    location ~ [.]php$ {
        return 404;
    }
}
~~~

Nginx serves the existing asset itself. Application routes that do not match a file fall back to the front controller. The exact front-controller location sends only the index script to FPM, while other PHP paths return 404.

Check the package-provided FastCGI parameter file and ensure required request parameters are available without duplicate conflicting values. The FPM process must be able to read the script at the path Nginx passes.

## Apache alternative

Use Apache instead of the Nginx block when Apache is the HTTP server. Enable the required proxy, FastCGI, and rewrite modules for the installed Apache package.

~~~apache
<VirtualHost *:80>
    ServerName shop.example.test
    DocumentRoot /srv/shop/public
    DirectoryIndex index.php

    <Directory /srv/shop/public>
        Require all granted
        Options -Indexes
        AllowOverride None

        RewriteEngine On
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteCond %{REQUEST_FILENAME} !-d
        RewriteRule ^ index.php [QSA,L]
    </Directory>

    <FilesMatch "index[.]php$">
        SetHandler "proxy:unix:/run/php/shop-fpm.sock|fcgi://localhost/"
    </FilesMatch>
</VirtualHost>
~~~

The rewrite rules send non-file application routes to the front controller. The handler passes that script to the FPM socket. Apache and FPM must use the same socket path and compatible filesystem paths. Do not enable both server blocks as if they were one configuration; select the web server actually serving the application.

## Validate and reload

For Nginx, check and reload its configuration. For Apache, use its corresponding syntax check and reload command. PHP-FPM is a separately managed service; reload or restart it when its pool configuration changes.

~~~sh
sudo nginx -t
sudo systemctl reload nginx
sudo systemctl reload php-fpm
~~~

The FPM service unit may include the installed PHP version in its name. Use the actual service name on the host. A successful syntax check does not prove that the socket exists or that the services can connect; inspect service status and logs if requests fail.

## Check the HTTP behavior

Request the front controller:

~~~sh
curl -i http://127.0.0.1/
~~~

Expected body:

~~~text
Request executed by PHP-FPM.
~~~

Request the static asset:

~~~sh
curl -i http://127.0.0.1/assets/note.txt
~~~

Expected body:

~~~text
Static asset served by the web server.
~~~

Request an unrelated PHP path:

~~~sh
curl -i http://127.0.0.1/missing.php
~~~

Expected result: HTTP 404. It should not be passed to PHP-FPM by this front-controller configuration.

## Troubleshooting

- If PHP requests return 502, check FPM service status, the socket path, socket ownership and mode, and the web-server error log.
- If FPM says the script cannot be found, compare the script filename passed by the web server with the path visible inside the FPM process or container.
- If the front controller returns 404 for application routes, check the fallback rule and the Apache rewrite module or Nginx location handling.
- If static files return 403, check read permissions on the public directory and each parent directory.
- If requests queue under load, inspect the FPM listen queue, busy workers, memory use, request durations, and downstream services.
- If an uploaded PHP file can execute, move uploads outside the public directory or explicitly deny script execution in that location.

Check web-server, FPM, and application logs together. Keep status endpoints and detailed diagnostics restricted to trusted operators.

## Expected request flow

~~~text
GET /assets/note.txt
    -> Nginx or Apache finds the static file
    -> web server returns the file without PHP execution

GET /
    -> web server routes to public/index.php
    -> FastCGI sends the script request to PHP-FPM
    -> an FPM worker executes PHP
    -> web server returns the response

GET /missing.php
    -> the front-controller rule rejects the path
    -> web server returns 404
~~~

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [141 - Deployment environments and release workflows](../lessons/141-deployment-environments-and-release-workflows.md) | [Lesson](../lessons/142-apache-nginx-and-php-fpm-integration.md) | [Q&A](../qnas/142-apache-nginx-and-php-fpm-integration.md) | [143 - Production configuration, permissions, and logging](../lessons/143-production-configuration-permissions-and-logging.md) |