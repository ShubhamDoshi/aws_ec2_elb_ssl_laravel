
I am using both HTTP and HTTPS listeners on my Elastic Load Balancing (ELB) load balancer. 
The ELB is offloading SSL, and the backend is listening only on a single HTTP port (HTTPS to HTTP). 
I want all traffic coming to my web server on port 80 to be redirected to HTTPS port 443, 
but I don’t want to change my backend listener to port 443. When I redirect traffic, 
my website stops working, and I receive this error message: ERR_TOO_MANY_REDIRECTS. How do I resolve this?
This error is commonly caused by the following:

    The rewrite rule on the web server for directing HTTP requests to HTTPS causes requests to 
    use port 443 for HTTPS traffic on the load balancer.
    The load balancer still sends the requests to the backend web server on port 80.
    The backend web server redirects these requests to port 443 on the load balancer.

This causes an infinite loop of redirection between the load balancer and the backend web server, and the requests are never served.
Resolution

Using the X-Forwarded-Proto header of the HTTP request, change your web server’s rewrite rule to apply only if the client protocol is HTTP. Ignore the rewrite rule for all other protocols used by the client.

This way, if clients use HTTP to access your website, they are redirected to an HTTPS URL, and if clients use HTTPS, they are served directly by the web server.

Apache <br>
<VirtualHost *:80><br>
<br>
RewriteEngine On <br>
RewriteCond %{HTTP:X-Forwarded-Proto} =http<br>
RewriteRule .* https://%{HTTP:Host}%{REQUEST_URI} [L,R=permanent]<br>
<br>
</VirtualHost><br>

.htaccess<br>

RewriteEngine On<br>
RewriteCond %{HTTP:X-Forwarded-Proto} =http<br>
RewriteRule .* https://%{HTTP:Host}%{REQUEST_URI} [L,R=permanent]<br>
