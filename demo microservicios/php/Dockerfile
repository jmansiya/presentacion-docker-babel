FROM josemansilla/apache2:1.0
MAINTAINER José Mansilla García-Gil "jose.mansilla@babel.es"

RUN apt-get update && \
      apt-get install -y php && \
      apt-get clean && \
      rm -rf /var/lib/apt/lists/*

EXPOSE 80
ADD ["index.php","/var/www/html/"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]