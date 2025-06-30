FROM bref/php-83-console:2

COPY --from=bref/extra-gd-php-83 /opt /opt
 
# Copy the source code in the image
COPY . /var/task

COPY php/conf.d/intl.ini /opt/bref/etc/php/conf.d/ 
 
# Configure the handler file (the entrypoint that receives all HTTP requests)
CMD ["bin/magento"]
