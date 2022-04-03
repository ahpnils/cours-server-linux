FROM debian
MAINTAINER YourName <admin@example.com>

RUN apt update
RUN apt -y install tzdata
RUN apt -y install apache2
RUN echo "COUCOU !!!" > /var/www/html/index.html

EXPOSE 80
CMD ["/usr/sbin/apachectl", "-D", "FOREGROUND"]
