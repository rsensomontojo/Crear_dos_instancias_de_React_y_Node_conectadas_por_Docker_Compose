# Crear dos instancias de React y Node conectadas por Docker Compose

Este documento explica cómo configurar y ejecutar dos servicios de React y dos de Node.js utilizando Docker Compose, con configuraciones específicas para entornos de desarrollo y producción.

Si te perdiste *como Conectar React con Node usando Docker_Compose*, ir a la [documentación](https://github.com/rsensomontojo/Documentacion_Conectar_React_con_Node_usando_Docker_Compose).


---

## Paso 1: Crear el archivo `docker-compose.yaml`

Este archivo define los servicios que se ejecutarán en contenedores Docker. Crea el archivo `docker-compose.yaml` con el siguiente contenido:

```yaml
version: '3'

services:
  node-api-1:
    build:
      context: ./node/next-productivity-API
      dockerfile: Dockerfile
    container_name: node-api-1
    ports:
      - "5001:5001"
    networks:
      - productivity-network
    environment:
      - NODE_ENV=development
    volumes:
      - ./node/next-productivity-API:/app
      - /app/node_modules

  node-api-2:
    build:
      context: ./node/next-productivity-API
      dockerfile: dockerfile-prod
    container_name: node-api-2
    ports:
      - "5002:5002"
    networks:
      - productivity-network
    environment:
      - NODE_ENV=production
    volumes:
      - ./node/next-productivity-API:/app
      - /app/node_modules

  react-app-1:
    build:
      context: ./react/next-productivity-app
      dockerfile: Dockerfile
    container_name: react-app-1
    ports:
      - "3001:3000"
    networks:
      - productivity-network
    depends_on:
      - node-api-1
    environment:
      - NODE_ENV=development
    volumes:
      - ./react/next-productivity-app:/app
      - /app/node_modules

  react-app-2:
    build:
      context: ./react/next-productivity-app
      dockerfile: dockerfile-prod
    container_name: react-app-2
    ports:
      - "3002:3000"
    networks:
      - productivity-network
    depends_on:
      - node-api-2
    environment:
      - NODE_ENV=production
    volumes:
      - ./react/next-productivity-app:/app
      - /app/node_modules

networks:
  productivity-network:
    driver: bridge
```

### Explicación del `docker-compose.yaml`:

1. **Servicios de Node (node-api-1 y node-api-2):**
   - **`node-api-1`:** Servicio en modo desarrollo, disponible en el puerto 5001.
   - **`node-api-2`:** Servicio en modo producción, disponible en el puerto 5002.
   - Ambos servicios comparten el directorio `./node/next-productivity-API`.

2. **Servicios de React (react-app-1 y react-app-2):**
   - **`react-app-1`:** Aplicación React en modo desarrollo, disponible en el puerto 3001.
   - **`react-app-2`:** Aplicación React en modo producción, disponible en el puerto 3002.
   - Los servicios dependen de las instancias de Node correspondientes.

3. **Redes:**
   - Se crea una red llamada `productivity-network` para permitir la comunicación entre los contenedores.

4. **Volúmenes:**
   - Los volúmenes sincronizan los archivos locales con los contenedores, permitiendo desarrollo.

---

## Paso 2: Configurar los Dockerfiles

### Para Node.js (backend)

#### Dockerfile (modo desarrollo):
Ubicado en `node/next-productivity-API/Dockerfile`:

```dockerfile
FROM node:16

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm install

COPY . .

EXPOSE 5001

CMD ["npm", "run", "dev"]
```

#### Dockerfile (modo producción):
Ubicado en `node/next-productivity-API/dockerfile-prod`:

```dockerfile
FROM node:16

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm install --production

COPY . .

EXPOSE 5002

CMD ["npm", "run", "start"]
```

### Para React (frontend)

#### Dockerfile (modo desarrollo):
Ubicado en `react/next-productivity-app/Dockerfile`:

```dockerfile
FROM node:16

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm install

COPY . .

EXPOSE 3001

CMD ["npm", "start"]
```

#### Dockerfile (modo producción):
Ubicado en `react/next-productivity-app/dockerfile-prod`:

```dockerfile
FROM node:16

WORKDIR /app

COPY package.json package-lock.json ./

RUN npm install --production

COPY . .

RUN npm run build

EXPOSE 3002

CMD ["npm", "run", "start"]
```

---

## Paso 3: Levantar los contenedores

Una vez que los archivos estén configurados, ejecuta el siguiente comando desde la carpeta principal:

```bash
docker-compose up --build
```

Esto construirá las imágenes de Docker y levantará los contenedores.

### Acceder a los servicios:

- React 1: [http://localhost:3001](http://localhost:3001)
- React 2: [http://localhost:3002](http://localhost:3002)
- Node 1: [http://localhost:5001](http://localhost:5001)
- Node 2: [http://localhost:5002](http://localhost:5002)

---
## Solución de problemas comunes

### Error: `UnixHTTPConnectionPool` o `Read timed out`

Si encuentras este error:

```bash
ERROR: for react-app-2  UnixHTTPConnectionPool(host='localhost', port=None): Read timed out. (read timeout=60)
ERROR: An HTTP request took too long to complete. Retry with --verbose to obtain debug information.
```

Es posible que las solicitudes HTTP a los contenedores estén tardando más de 60 segundos. Para solucionarlo, puedes aumentar el tiempo de espera predeterminado de Docker Compose.

#### Soluciones:

1. **Aumentar el tiempo de espera al ejecutar el comando:**
   ```bash
   COMPOSE_HTTP_TIMEOUT=200 docker-compose up --build
   ```

2. **Exportar la variable de entorno globalmente:**
   ```bash
   export COMPOSE_HTTP_TIMEOUT=200
   docker-compose up --build
   ```

3. **Optimizar los Dockerfiles:**
   - Usa capas de caché eficientemente. Por ejemplo, copia primero `package.json` y `package-lock.json` antes del resto de los archivos para minimizar el tiempo de instalación de dependencias si no han cambiado.

4. **Asignar más recursos a Docker:**
   - En Docker Desktop, aumenta los recursos asignados (CPU, RAM) desde la configuración.

---

¡Y listo! Ahora tienes configurados dos servicios de React y Node conectados mediante Docker Compose.
