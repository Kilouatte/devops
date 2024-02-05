FROM postgres:14.1-alpine

ENV POSTGRES_DB=oui \
   POSTGRES_USER=oui \
   POSTGRES_PASSWORD=oui

COPY ./CreateScheme.sql /docker-entrypoint-initdb.d/

COPY ./InsertData.sql /docker-entrypoint-initdb.d/