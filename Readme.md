# Sistema de Contactos y Actividades

Este proyecto es un **Sistema de Contactos y Actividades** desarrollado en **ASP.NET Core** con una base de datos **MongoDB**. Utiliza **Docker** para el despliegue en contenedores.

## Requisitos Previos

Antes de ejecutar este proyecto, asegúrate de tener instalado lo siguiente:

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [.NET SDK](https://dotnet.microsoft.com/download) (opcional, solo para desarrollo local)

## Estructura del Proyecto
```

El proyecto tiene la siguiente estructura:

Sistema_Contactos/
├── bin/
├── Controllers/
├── Data/
├── Models/
├── obj/
├── Properties/
├── Servicios/
├── Views/
├── wwwroot/
├── appsettings.Develop...
├── appsettings.json
├── Program.cs
├── Sistema_Contactos.c...
├── Sistema_Contactos.c...
├── docker-compose.yml   
├── Dockerfile          
└── Sistema_Contactos.sln

```

debe de estar dockerfile y docker-compose.yml


## Configuración de Docker

### 1. Archivo `Dockerfile`

```Dockerfile

# Usar la imagen oficial de .NET 8 para ejecutar la app
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 18  

# Usar la imagen de SDK para compilar la aplicación
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["Sistema_Contactos.csproj", "./"]
RUN dotnet restore
COPY . .
WORKDIR "/src"
RUN dotnet build -c Release -o /app/build

# Publicar la aplicación
FROM build AS publish
RUN dotnet publish -c Release -o /app/publish

# Crear la imagen final
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Sistema_Contactos.dll"]

```

### 2. Archivo docker-compose.yml

El archivo docker-compose.yml define los servicios (contenedores) de la aplicación y MongoDB.

```yml

services:
  # Contenedor de MongoDB
  mongodb:
    image: mongo:6.0
    container_name: mongodb
    ports:
      - "20253:27017"  # Puerto personalizado en el host : Puerto estándar en el contenedor
    volumes:
      - mongodb_data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: Tito18
      MONGO_INITDB_ROOT_PASSWORD: Juan189
    networks:
      - internal_network

  # Contenedor de la aplicación ASP.NET Core
  app:
    image: sistema-contactos
    container_name: sistema-contactos
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "2003:18"  # Puerto personalizado en el host : Puerto estándar en el contenedor
    depends_on:
      - mongodb
    environment:
      - ASPNETCORE_URLS=http://+:18
      - Db_config__Db_connection=mongodb://Tito18:Juan189@mongodb:27017
      - Db_config__Db_name=Proyecto_Bb_2
      - Db_config__C_Contactos=Contactos
      - Db_config__C_Actividades=Actividades
    networks:
      - internal_network

# Volumen para persistir los datos de MongoDB
volumes:
  mongodb_data:

# Red interna para aislar los contenedores
networks:
  internal_network:
    driver: bridge

```

## Configuración de la Aplicación

**Archivo appsettings.json**

El archivo appsettings.json contiene la configuración de la aplicación, incluyendo la cadena de conexión a MongoDB.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "Db_config": {
    "Db_connection": "mongodb://Tito18:Juan189@mongodb:27017",
    "Db_name": "Proyecto_Bb_2",
    "C_Contactos": "Contactos",
    "C_Actividades": "Actividades"
  }
}
```

## Despliegue con Docker Compose

Antes de emepzar con los codigos nos tenemos que dirigir a la carpeta donde esta nuestro proyecto y lo archivos **Dockerfile** y **docker-compose.yml**.

### 1 Construir y Ejecutar los Contenedores

Para construir las imágenes y ejecutar los contenedores, utiliza el siguiente comando:

```bash
docker-compose up -d --build
```

Este comando:

- Construye las imágenes.

- Crea y arranca los contenedores (mongodb y sistema-contactos).


![alt text](image.png)

![alt text](image-1.png)

### 2 Verificar el Estado de los Contenedores

Para verificar que los contenedores estén en ejecución:

```bash 
docker-compose ps 
```

![alt text](image-2.png)

### 3 Ingresamos a la aplicación 

http://localhost:2003/

usamos el puerto 2003 por que es lo que pusimos en el archivo **docker-compose.yml**

![alt text](image-3.png)

## Conectarse a MongoDB desde el Contenedor
Para conectarse a MongoDB y verificar los datos:

Entra al contenedor de MongoDB:

```bash

docker exec -it mongodb /bin/bash
```

![alt text](image-4.png)

Instala mongosh (si no está instalado):

```bash
apt-get update && apt-get install -y mongodb-mongosh
```
![alt text](image-5.png)

Conéctate a la base de datos:

```bash
mongosh "mongodb://Tito18:Juan189@localhost:27017"
```

![alt text](image-6.png)

Lista las bases de datos:

```bash
show dbs
```

![alt text](image-7.png)

Selecciona la base de datos:

```bash
use Proyecto_Bb_2
```
Lista las colecciones:

```bash
show collections
```
Consulta los documentos en una colección:

```bash
db.Contactos.find()
```
![alt text](image-8.png)

## Conectarse a MongoDB desde MongoDB Compass

**Para conectarse a MongoDB desde MongoDB Compass:**

- Abre MongoDB Compass.

**Ingresa los siguientes datos de conexión:**

- Hostname: localhost

- Port: 20253

**Authentication: Username / Password**

- Username: Tito18

- Password: Juan189

**Authentication Database: admin**

- Haz clic en "Connect".

**Selecciona la base de datos Proyecto_Bb_2.**

Explora las colecciones (Contactos y Actividades).

![alt text](image-9.png)

