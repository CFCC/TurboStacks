FROM docker.io/library/fedora:34

# https://fedoramagazine.org/how-to-set-up-a-tftp-server-on-fedora/
RUN dnf install -y tftp tftp-server

CMD ["/usr/sbin/in.tftpd", "-L", "-c", "-p", "-s", "/var/lib/tftpboot"]
