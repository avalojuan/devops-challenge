# Imagen base
FROM python:3.7-alpine

WORKDIR /api

# Copiamos la especificacion de paquetes requeridos
COPY requirements.txt .

RUN \
  apk update && apk add --no-cache postgresql-libs && \
  apk add --no-cache --virtual .build-deps python3-dev gcc musl-dev postgresql-dev graphviz-dev  && \
  python3 -m pip install -r requirements.txt --no-cache-dir && \
  apk --purge del .build-deps

# Copiamos los archivos de la api al contenedor
COPY . .

# Exponemos el puerto utilizado por la api
EXPOSE 8000

# Ejecutamos el servidor
CMD [ "python", "manage.py", "runserver", "0.0.0.0:8000"]