# NAME

Plack::Middleware::Deflater - Compress response body with Gzip or Deflate

# SYNOPSIS

    use Plack::Builder;

    builder {
      enable sub {
          my $app = shift;
          sub {
              my $env = shift;
              my $ua = $env->{HTTP_USER_AGENT} || '';
              # Netscape has some problem
              $env->{"psgix.compress-only-text/html"} = 1 if $ua =~ m!^Mozilla/4!;
              # Netscape 4.06-4.08 have some more problems
               $env->{"psgix.no-compress"} = 1 if $ua =~ m!^Mozilla/4\.0[678]!;
              # MSIE (7|8) masquerades as Netscape, but it is fine
              if ( $ua =~ m!\bMSIE (?:7|8)! ) {
                  $env->{"psgix.no-compress"} = 0;
                  $env->{"psgix.compress-only-text/html"} = 0;
              }
              $app->($env);
          }
      };
      enable "Deflater",
          content_type => ['text/css','text/html','text/javascript','application/javascript'],
          vary_user_agent => 1;
      sub { [200,['Content-Type','text/html'],["OK"]] }
    };

# DESCRIPTION

Plack::Middleware::Deflater is a middleware to encode your response
body in gzip or deflate, based on `Accept-Encoding` HTTP request
header. It would save the bandwidth a little bit but should increase
the Plack server load, so ideally you should handle this on the
frontend reverse proxy servers.

This middleware removes `Content-Length` and streams encoded content,
which means the server should support HTTP/1.1 chunked response or
downgrade to HTTP/1.0 and closes the connection.

# CONFIGURATIONS

- content\_type

        content_type => 'text/html',
        content_type => [ 'text/html', 'text/css', 'text/javascript', 'application/javascript', 'application/x-javascript' ]

    Content-Type header to apply deflater. if content-type is not defined, Deflater will try to deflate all contents.

- vary\_user\_agent

        vary_user_agent => 1

    Add "User-Agent" to Vary header.

# ENVIRONMENT VALUE

- psgix.no-compress

    Do not apply deflater

- psgix.compress-only-text/html

    Apply deflater only if content\_type is "text/html"

- plack.skip-deflater

    Skip all Deflater features

## Compare psgix.no-compress with plack.skip-deflater

If no-compress is true, PM::Deflater skips gzip or deflate. But adds Vary: Accept-Encoding and Vary: User-Agent header. skip-deflater forces to skip all PM::Deflater feature, doesn't allow to add Vary header.

# LICENSE

This software is licensed under the same terms as Perl itself.

# AUTHOR 

Tatsuhiko Miyagawa

# SEE ALSO

[Plack](https://metacpan.org/pod/Plack), [http://httpd.apache.org/docs/2.2/en/mod/mod\_deflate.html](http://httpd.apache.org/docs/2.2/en/mod/mod_deflate.html)
