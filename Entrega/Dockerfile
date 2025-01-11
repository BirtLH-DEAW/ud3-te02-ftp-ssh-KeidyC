# Utilizamos una imagen oficial de Ubuntu
FROM ubuntu:18.04

# Damos información sobre la imagen que estamos creando
LABEL \
    version="1.0" \
    description="Ubuntu + Apache2 + ftp + ssh + git + virtual host" \
    maintainer="kcoellogaray <kcoellogaray@birt.eus>"

# Actualizamos la lista de paquetes e instalamos nano y apache2
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y nano apache2 \
    proftpd openssl ssh git && \
    rm -rf /var/lib/apt/lists/*

# Creamos directorios para los sitios web y configuraciones
RUN mkdir -p /var/www/html/sitioprimero /var/www/html/sitiosegundo && \
    mkdir -p /var/ftp/UD3 

# Configurar usuarios keidy
RUN useradd -d /var/www/html/sitioprimero -s /usr/sbin/nologin keidy && \
    echo "keidy:123" | chpasswd && \
    chown keidy:keidy /var/www/html/sitioprimero && \
    chmod 755 /var/www/html/sitioprimero && \
    #Usuario keidy2
    useradd -d /var/www/html/sitiosegundo -s /bin/bash keidy2 && \
    echo "keidy2:123" | chpasswd && \
    chown keidy2:keidy2 /var/www/html/sitiosegundo && \
    chmod 700 /var/www/html/sitiosegundo

# Copiamos ficheros necesarios al directorio raíz
COPY apache/indexPrimero.html /var/www/html/sitioprimero/index.html 
COPY apache/indexSegundo.html /var/www/html/sitiosegundo/index.html
COPY apache/apache2.conf /etc/apache2/apache2.conf
COPY apache/sitioPrimero.conf /etc/apache2/sites-available/sitioPrimero.conf
COPY apache/sitioSegundo.conf /etc/apache2/sites-available/sitioSegundo.conf
COPY apache/sitioPrimero.key /etc/ssl/private/sitioPrimero.key
COPY apache/sitioPrimero.cer /etc/ssl/certs/sitioPrimero.cer

COPY ftp/proftpd.conf /etc/proftpd/proftpd.conf
COPY ftp/tls.conf /etc/proftpd/tls.conf
COPY ftp/cert.pem /etc/ssl/certs/cert.pem
COPY ftp/key.pem /etc/ssl/private/key.pem

COPY ssh/sshd_config /etc/ssh
COPY ssh/id_rsa /etc

# Clonar repositorio Git en carpeta FTP
RUN eval "$(ssh-agent -s)" \
&& chmod 700 /etc/id_rsa \
&& ssh-add /etc/id_rsa \
&& ssh-keyscan -H github.com >> /etc/ssh/ssh_known_hosts \
&& git clone git@github.com:deaw-birt/UD3-ftp_anonimo.git /srv/ftp/UD3 \
&& rm /etc/id_rsa

# Habilitamos los sitios y el módulo SSL
RUN a2ensite sitioPrimero.conf && \
    a2ensite sitioSegundo.conf && \
    a2enmod ssl

# Exponemos los puertos
EXPOSE 80
EXPOSE 443
EXPOSE 21
EXPOSE 1022
EXPOSE 50000-50030

# Comando por defecto al iniciar el contenedor
CMD ["apachectl", "-D", "FOREGROUND"]
