# Use Apache as base image
FROM httpd:alpine

# Copy the landing page into the container
COPY index.html /usr/local/apache2/htdocs/index.html
COPY httpd.conf /usr/local/apache2/conf/httpd.conf

# Expose port 80
EXPOSE 80