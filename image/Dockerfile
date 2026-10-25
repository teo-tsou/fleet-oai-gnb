# The OAI gNB with the USRP B2xx firmware baked in. OAI's entrypoint downloads it from the internet at every start
# (USE_B2XX), which an air-gapped site cannot do. UHD's own downloader fetches it at BUILD time and checks it against
# the manifest UHD ships (per-file SHA-256). Base pinned by digest: oaisoftwarealliance/oai-gnb:2026.w43.
FROM docker.io/oaisoftwarealliance/oai-gnb@sha256:b7554483225068fa9c6a03032507b00fc2c663786c436fe61ea89b956fdb4850
RUN mkdir -p /usr/local/share/uhd/images \
 && /opt/oai-gnb/bin/uhd_images_downloader.py -t b2xx -i /usr/local/share/uhd/images \
 && ls -l /usr/local/share/uhd/images
ENV UHD_IMAGES_DIR=/usr/local/share/uhd/images
LABEL org.opencontainers.image.source="https://github.com/teo-tsou/fleet-oai-gnb" \
      org.opencontainers.image.description="OAI gNB 2026.w43 + USRP B2xx firmware (IntEdge)"
