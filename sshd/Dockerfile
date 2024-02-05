FROM alpine:latest
LABEL maintainer="jnmartineau@gmail.com"
RUN apk add --update --no-cache openssh 
RUN sed -i '/PasswordAuthentication/d' /etc/ssh/sshd_config && \
    echo 'PasswordAuthentication yes' >> /etc/ssh/sshd_config
RUN sed -i '/AllowTcpForwarding/d' /etc/ssh/sshd_config && \
    echo 'AllowTcpForwarding yes' >> /etc/ssh/sshd_config
RUN sed -i '/PermitTunnel/d' /etc/ssh/sshd_config && \
    echo 'PermitTunnel yes' >> /etc/ssh/sshd_config
RUN adduser -h /home/pi -s /bin/sh -D pi
RUN echo -n 'pi:.ailsc1e' | chpasswd
ENTRYPOINT ["/entrypoint.sh"]
EXPOSE 22
COPY entrypoint.sh /

