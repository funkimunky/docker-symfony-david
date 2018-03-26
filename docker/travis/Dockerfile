FROM ubuntu:16.04
USER root
RUN apt update
RUN apt install -y net-tools iptables
RUN sysctl net.ipv4.ip_forward=1
RUN iptables -t nat -I PREROUTING --src 0/0 --dst $(netstat -rn | grep 0.0.0.0 | awk '{print $2}' | grep -v "0.0.0.0")
RUN iptables -t nat -A POSTROUTING -j MASQUERADE