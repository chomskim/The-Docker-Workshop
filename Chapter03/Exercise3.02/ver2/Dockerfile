FROM alpine

RUN apk update && apk add wget curl

RUN wget -O test.txt https://github.com/PacktWorkshops/The-Docker-Workshop/raw/master/Chapter03/Exercise3.02/100MB.bin

CMD -p mkdir /var/www/html/

WORKDIR /var/www/html/

ADD Dockerfile.tar.gz /var/www/html/

RUN cat Dockerfile
