docker-basic-squid
==================

A non-caching forward proxy using [Squid](http://www.squid-cache.org/) with Basic authentication. Forked from [mafr/docker-squid](https://github.com/mafr/docker-squid)

Setting up a user
-----------------
Cd into the project directory and use the `htpasswd` program from the [httpd](https://formulae.brew.sh/formula/httpd) package to add users:

    $ htpasswd conf/passwd proxy_user
    New password:
    Re-type new password:
    Adding password for user proxy_user
    $ 

If on **Windows**, you can use an online htpasswd generator such as https://wtools.io/generate-htpasswd-online and copy the resulting output to the `conf/passwd` file in the project directory.

Build the image and run container
---------------------------------
From the project directory build and tag the image:

    $ docker build -t basic-squid .

Use the built image to create a container and run it:

    $ docker run \
       --detach \
       --name=local-squid \
       --publish=3128:3128 \
       --volume=</path/to/store/squid/logs>:/var/log/squid3/ \
       basic-squid

**Note** the volume switch: We map Squid's log directoy to a directory on
the host to get convenient access to the proxy's log files (`cache.log` and `access.log`). The directory
on the host has to be writeable by the container's proxy user!

You could also map the `/etc/squid3/` to the host's filesystem. That way,
you can add new users or play with the configuration without needing to
build a new image. Sending a HUP signal from the host to the Squid process
will cause Squid to re-read its configuration e.g.

    docker kill --signal=HUP <container>

Testing
-------
Proxy user auth:

    curl -I -x localhost:3128 -U proxy_user:password /
    https://google.com

    HTTP/1.1 200 Connection established

No proxy user auth returns 407 response:

    curl -I -x localhost:3128 http://google.com

    HTTP/1.1 407 Proxy Authentication Required
    Server: squid/3.3.8


