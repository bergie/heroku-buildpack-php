Apache+PHP build pack
========================

This is a build pack bundling PHP and Apache for Heroku apps.

Configuration
-------------

The config files are bundled with the LP itself:

* conf/httpd.conf
* conf/php.ini


Pre-compiling binaries
----------------------

    # apache
    mkdir /app
    wget http://apache.cyberuse.com//httpd/httpd-2.2.19.tar.gz
    tar xvzf httpd-2.2.19.tar.gz
    cd httpd-2.2.19
    ./configure --prefix=/app/apache --enable-rewrite
    make
    make install
    cd ..
    
    # php
    wget http://us2.php.net/get/php-5.3.6.tar.gz/from/us.php.net/mirror 
    mv mirror php.tar.gz
    tar xzvf php.tar.gz
    cd php-5.3.6/
    ./configure --prefix=/app/php --with-apxs2=/app/apache/bin/apxs --with-mysql --with-pdo-mysql --with-pgsql --with-pdo-pgsql --with-iconv --with-gd --with-curl=/usr/lib --with-config-file-path=/app/php --enable-soap=shared --with-openssl
    make
    make install
    cd ..
    
    # php extensions
    mkdir /app/php/ext
    cp /usr/lib/libmysqlclient.so.15 /app/php/ext/
    
    # pear
    apt-get install php5-dev php-pear
    pear config-set php_dir /app/php
    pecl install apc
    mkdir /app/php/include/php/ext/apc
    cp /usr/lib/php5/20060613/apc.so /app/php/ext/
    cp /usr/include/php5/ext/apc/apc_serializer.h /app/php/include/php/ext/apc/
    
    
    # package
    cd /app
    echo '2.2.19' > apache/VERSION
    tar -zcvf apache.tar.gz apache
    echo '5.3.6' > php/VERSION
    tar -zcvf php.tar.gz php

	
Getting binaries out of the ephemeral filesystem!
-------------------------------------------------

After you compile the binaries and bundle them into your .tar.gz files stored in the /app folder.

WARNING: DO NOT log out of your 'heroku bash' shell-- doing so would return the files you just waited 20 minutes to compile back into the ephemeral-filesystem ether!

Instead, run the following script to implement 's3-bash' in order to whisk those files out to Amazon S3 immediately.

    #!/bin/bash
    #
    #--   UPLOAD YOUR BUILT .TAR.GZ ARCHIVES DIRECT TO S3 VIA BASH   --#
    #--   after you https://github.com/heroku/heroku-buildpack-php   --#
    #
    set -uex

    #-- configure amazon
    AMAZON_KEY_ID="A12345WZ12345KL12345"
    AMAZON_SECRET="a123457U12345+D12345aB12345tE12345RZ123P"
    AMAZON_BUCKET_NAME="lorem-ipsum"

    #-- function to put files
    function put_file() {
        local FILENAME="$1"
        ./s3-put -k "$AMAZON_KEY_ID" -s secret -T "$FILENAME" "/$AMAZON_BUCKET_NAME/$FILENAME"
    }

    #-- go to folder
    cd /app

    #-- download and unpack s3-bash
    curl -O https://s3-bash.googlecode.com/files/s3-bash.0.02.tar.gz
    tar -xf s3-bash.0.02.tar.gz
    chmod a+x s3-*

    #-- store the secret
    printf "$AMAZON_SECRET" > secret

    #-- put the files
    put_file apache-2.2.25.tar.gz
    put_file php-5.3.27.tar.gz
    put_file mcrypt-2.5.8.tar.gz

    #-- the last step is making those three files public on AWS:
    #
    #  When complete, they will be publicly accessible, ala:
    #
    #  http://lorem-ipsum.s3.amazonaws.com/php-5.3.27.tar.gz
    #  http://lorem-ipsum.s3.amazonaws.com/apache-2.2.25.tar.gz
    #  http://lorem-ipsum.s3.amazonaws.com/mcrypt-2.5.8.tar.gz
    #
    #--

	
Hacking
-------

To change this buildpack, fork it on Github. Push up changes to your fork, then create a test app with --buildpack <your-github-url> and push to it.


Meta
----

Created by Pedro Belo.
Many thanks to Keith Rarick for the help with assorted Unix topics :)