FROM vlg-bionic
#RUN sudo apt-get update
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get -y install haproxy
COPY --chown=root:root haproxy.cfg /etc/haproxy/
RUN mkdir /run/haproxy
EXPOSE 80

CMD ["haproxy", "-f", "/etc/haproxy/haproxy.cfg"]
#CMD ["echo", "blahblah"]


