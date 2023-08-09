FROM docker.elastic.co/beats/filebeat:7.15.1
COPY filebeat.yml /usr/share/filebeat/filebeat.yml
USER root
RUN chmod go-w /usr/share/filebeat/filebeat.yml && chown root:filebeat /usr/share/filebeat/filebeat.yml
USER filebeat
