# On utilise l'image postgres:14.1-alpine
FROM postgres:14.1-alpine

# On copie les fichiers SQL dans le répertoire d'initialisation de la base de données
COPY ./CreateScheme.sql /docker-entrypoint-initdb.d/

COPY ./InsertData.sql /docker-entrypoint-initdb.d/