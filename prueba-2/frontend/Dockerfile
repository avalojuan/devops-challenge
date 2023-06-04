# Se utiliza node 14 por problemas de dependencias 
# con versiones superiores de npm y React.
FROM node:14-alpine as builder

WORKDIR '/app'

# Copiamos primero el package.json para utilizar el cache
# del build en caso de no haber cambios de dependencias.
COPY ./package.json ./
RUN npm install

# Se copian los archivos de la app
COPY . .

# Se realiza el build utilizando el script del package.json
RUN npm run build

# Utilizamos la imagen de nginx para servir los archivos estaticos creados
# en la fase anterior
FROM nginx

# Exponemos el puerto 3000 que utiliza nuestra app
EXPOSE 3000

# Copiamos el archivo de configuracion de nginx
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf

# Copiamos los archivos estaticos creado en la fase anterior
# al directorio que especifica la documentacion de nginx
COPY --from=builder /app/assets/bundles /usr/share/nginx/html
