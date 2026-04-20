FROM ubuntu:24.04

# Evita interações durante a instalação de pacotes
ENV DEBIAN_FRONTEND=noninteractive

RUN apt update -y && apt install -y \
    ufw \
    openssh-server \
    systemd \
    systemctl \
    sed \
    iptables \
    && rm -rf /var/lib/apt/lists/*

RUN echo 'root:123' | chpasswd

RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

RUN update-alternatives --set iptables /usr/sbin/iptables-legacy && \
    update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy

EXPOSE 22

CMD ["/sbin/init"]
