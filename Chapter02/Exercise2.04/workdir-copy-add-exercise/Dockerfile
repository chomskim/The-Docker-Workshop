# WORKDIR, COPY and ADD example
FROM ubuntu:latest 
RUN apt-get update && apt-get install apache2 -y 
WORKDIR /var/www/html/
COPY index.html .
ADD https://seeklogo.com/images/M/moby-logo-EB8A3B8E0C-seeklogo.com.png?v=637792398800000000 ./logo.png
CMD ["ls"]
