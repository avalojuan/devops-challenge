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

# Exponemos el puerto a utilizar con la app
EXPOSE 3000

# Ejecutamos el script descripto en el package.json para iniciar la app
CMD ["npm", "start"]