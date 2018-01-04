# Graylog content pack for nginx using JSON logging

This is partially based on the core Graylog nginx content pack at https://github.com/graylog-labs/graylog-contentpack-nginx, and gives the same inputs, streams and a dashboard as that.

It is designed for people using virtual hosts or other sorts of more complex nginx configuration, and will only work with nginx version 1.11.8 onwards (you can remove the `escape=json` from the nginx setup if you want to use an earlier version).

The core advantage of this is that you can add arbitrary fields to the nginx logging and they will just appear magically in nginx, rather than having to delve into complex regex expressions to do things.

This content pack will create two inputs for the nginx `error_log` and `access_log`. Extractors are applied to effectively read the most important data into message fields. You will be able to do searches for all requests of a given remote IP, all requests that were answered with a HTTP 400 or just all requests that were slow.

The pack comes with a default dashboard to build upon and several streams that pre-group your HTTP requests into interesting categories. The additional log information described below (see *Configuring nginx*) will also add timing information to the requests handled by nginx.

![Screenshots](https://s3.amazonaws.com/graylog2public/images/contentpack-nginx-2.png)

![Screenshots](https://s3.amazonaws.com/graylog2public/images/contentpack-nginx-1.png)

### Configuring nginx

You need to run at least nginx version 1.11.8, escaped JSON support.

**Add this to your nginx configuration file and restart the service:**

        log_format graylog2_json escape=json '{ "timestamp": "$time_iso8601", '
                         '"remote_addr": "$remote_addr", '
                         '"remote_user": "$remote_user", '
                         '"body_bytes_sent": $body_bytes_sent, '
                         '"request_time": $request_time, '
                         '"status": $status, '
                         '"request": "$request", '
                         '"request_method": "$request_method", '
                         '"host": "$host",'
                         '"upstream_cache_status": "$upstream_cache_status",'
                         '"upstream_addr": "$upstream_addr",'
                         '"http_x_forwarded_for": "$http_x_forwarded_for",'
                         '"http_referrer": "$http_referer", '
                         '"http_user_agent": "$http_user_agent" }';

    # replace the hostnames with the IP or hostname of your Graylog2 server
    access_log syslog:server=graylog.server.org:12301 graylog2_json;
    error_log syslog:server=graylog.server.org:12302;
