FROM postgres:14.1-alpine

COPY CreateSchema.sql /docker-entrypoint-initdb.d/
COPY InsertData.sql /docker-entrypoint-initdb.d/