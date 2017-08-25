FROM gliderlabs/alpine:3.4

COPY sensor /bin

WORKDIR bin
ENTRYPOINT ["sensor"]

EXPOSE 9001
ENV SERVICE_9001_NAME sensor
ENV SERVICE_9001_CHECK_HTTP /health_check
