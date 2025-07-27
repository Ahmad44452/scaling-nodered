FROM nodered/node-red

USER root

WORKDIR /usr/src/node-red

COPY nodered-settings.js /data/settings.js

RUN npm install --prefix /data node-red-mongo-storage-plugin-with-sync
