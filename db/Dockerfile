FROM postgres:11.6-alpine
# Copie des fichiers de script dans l'entrypoint 
COPY scripts/01-CreateSchemes.sql /docker-entrypoint-initdb.d
COPY scripts/02-InsertData.sql /docker-entrypoint-initdb.d

# Variables d'environnement
ENV POSTGRES_DB=db \
POSTGRES_USER=usr \
POSTGRES_PASSWORD=pwd

