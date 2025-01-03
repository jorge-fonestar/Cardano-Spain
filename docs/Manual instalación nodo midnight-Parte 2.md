# PARTE 2 - INSTALACIÓN DEL NODO MIDNIGTH EN LA RED DE TEST

#### Disclaimer

Esta guía es tal cual, no pretende ser un proceso infalible, lo que  funcione en mi sistema puede que no funcione en el tuyo. Úsala bajo tu responsabilidad.

Fuentes:

- https://docs.midnight.network/validate/run-a-validator?utm_campaign=SPO%20Onboarding&utm_source=midnight-block-producer-workshop&utm_medium=YouTube

###### 1 Descagar el repositorio de partner-chain-docker.

```bash
git clone -b main https://github.com/midnight-ntwrk/partner-chain-deps-docker.git
cd partner-chain-deps-docker
```

#### 2 Cambiar usuario y password para POSTGRES

```bash
gedit compose.yml 

    environment:
      - NETWORK=preview
      - POSTGRES_HOST=postgres
      - POSTGRES_PORT=5432
      - POSTGRES_DB=cexplorer
      - POSTGRES_USER=<TU_USUARIO>
      - POSTGRES_PASSWORD=<TU_PASSWORD>
```

#### 3 Descargar dependencias con Docker

Tienes que tener un usuario de Docker Hub.

Debes iniciar sesión con tu usuario de docker. Esto guarda tus credenciales en un fichero local.

```bash
docker login -u <Tu_Usuario>
```

Cambiar configuración para ejecutar docker sin *sudo*

Fuente: [Rootless mode | Docker Docs](https://docs.docker.com/engine/security/rootless/)

Descargar y arrancar los contenedores por primera vez.

```bash
cd partner-chain-deps-docker
docker compose up -d
```

Comprobar los contenedores descargados

```bash
docker container list

docker logs db-sync
docker logs ogmios
docker logs kupo
docker logs postgres
docker logs cardano-node
```

Comprobar sincronización del nodo. Hay que esperar hasta que se complete la operación (1.00000).

```bash
curl -s localhost:1337/health | jq '.networkSynchronization'
1.00000
```

Si hay errores en alguno de ellos revisar los usuarios y contraseñas del fichero *compose.yml*

```bash
container_name: db-sync-postgres
    environment:
      - POSTGRES_PASSWORD=<TU_PASSWORD>
      - POSTGRES_DB=cexplorer

 container_name: db-sync
    environment:
      - POSTGRES_PASSWORD=<TU_PASSWORD>
```

También se puede dar el caso de que falten permisos de lectura/escritura para los directorios  del fichero ***.env***

Comandos docker de interés para la gestión de los contenedores.

```bash
docker container list
docker-compose stop # stop containers
docker-compose start # start containers
docker-compose restart # restart containers
docker-compose down # stop and remove containers
docker-compose stats # display resource usage statistics                                  # (Y|N) Download latest Mithril snapshot
```

#### 4 Consultar postgres

Instalar psql

```bash
sudo apt install postgresql-client-common
```

Login en psql

```tex
psql -h localhost -U postgres -d cexplorer -p 5432

sudo docker exec -it db-sync-postgres psql -U postgres -d cexplorer
```

Alternativa gráfica pgAdmin  [Download](https://www.pgadmin.org/download/)

#### 5 Instalar Partner-Chain-Cli

Fuente: https://docs.midnight.network/validate/run-a-validator/step-3#3a-install-partner-chains-cli-tool

Descargar e instalar siguiendo las instrucciones.

https://github.com/input-output-hk/partner-chains/releases:

```bash
 # Download the zip file
wget https://github.com/input-output-hk/partner-chains/releases/download/v1.1.0/linux_x86_64.zip

# Create the directory and move the zip file into it
mkdir -p partner-chains-cli
mv linux_x86_64.zip partner-chains-cli/

# Change into the directory
cd partner-chains-cli

# Unzip the file, this will handle the case where the zip might contain another zip or just files
unzip linux_x86_64.zip

# If there's another layer of zip files, you can use this to unzip all .zip files found
find . -name "*.zip" -exec unzip {} \; -exec rm {} \;

# Return to the previous directory if needed
cd - 
```

Crear un nuevo fichero de configuración para **Midnight testnet** , en */partner-chains-cli/partner-chains-cli-chain-config.json*

Añadir el siguiente contenido

```json
{
 "cardano": {
 "network": 2,
 "security_parameter": 432,
 "active_slots_coeff": 0.05,
 "first_epoch_number": 0,
 "first_slot_number": 0,
 "epoch_duration_millis": 86400000,
 "first_epoch_timestamp_millis": 1666656000000
 },
 "chain_parameters": {
 "chain_id": 23,
 "genesis_committee_utxo": "f44d20261bd3e079cc76b4d9b32b3330fea793b465c490766df71be90e577d8a#0",
 "threshold_numerator": 2,
 "threshold_denominator": 3,
 "governance_authority": "93f21ad1bba9ffc51f5c323e28a716c7f2a42c5b51517080b90028a6"
 },
 "cardano_addresses": {
 "committee_candidates_address": "addr_test1wp9pehc6t5xem0ccsf7dhktw4hu749dfm83fxx6p8f4jzpqyh330x",
 "d_parameter_policy_id": "f7e7b40ef803905a8567323f9d94fac536fe1cd3d8efbde5d249c5f7",
 "permissioned_candidates_policy_id": "3e0a39f32961debeb7c5db0e5deb98833d70835fc7ec40a3185c4ae5"
 }
}
```

###### 6.1 Generate Partner-chain keys

```bash
./partner-chains-cli generate-keys --help
./partner-chains-cli generate-keys 
```

Guardar el resultado en un fichero .txt en lugar seguro.

Endpoints públicos:

[https://ogmios.preview.midnight.network](https://ogmios.preview.midnight.network)

[https://kupo.preview.midnight.network](https://lkupogmios.preview.midnight.network)



Copiar llaves del nodo al volumen cardano-node

```bash
docker cp /home/<usuario>/cardano-testnet/keys/midnight/. cardano-node:/tmp/

docker exec -it cardano-node ls /tmp/
```

#### 6.2 Registro1

Ejecutar el proceso respondiendo a las datos solicitados.

Asegurar que el **payment.addr** tiene fondos y que el cardano-node está sincronizado con la red. Añadir los datos cuando el script los solicite.

```bash
./partner-chains-cli register1

> cardano cli executable docker exec cardano-node cardano-cli
> path to the cardano node socket file /home/<usuario>/ipc/node.socket
> path to the payment verification file /tmp/payment.vkey
```

###### 6.3 Registro2

Copiar el resultado de register 1 y guardar en sitio seguro.

```bash
./partner-chains-cli register2 \
 --chain-id 23 \
 --threshold-numerator 2 \
 --threshold-denominator 3 \
 --governance-authority 0x93f21ad1bbaxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx \
 --genesis-committee-utxo f44d20261bdxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx#0 \
 --registration-utxo 0866ea8d38182585xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx#0 \
 --aura-pub-key 0xea57181f5e672dc0811xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx \
 --grandpa-pub-key 0xa7545af1501dd0caxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx \
 --sidechain-pub-key 0x0289b3b90aae06xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx \
 --sidechain-signature c140a25d9e5690xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

/home/<usuario>/cardano-testnet/keys/KES/BP.cold.skey

###### 6.4 Registro 3

Copiar el resultado del registro 2

Ruta del fichero cold.key en el equipo principal no en docker. */home//cardano-testnet/keys/KES/BP.cold.skey*

```bash
./partner-chains-cli register3 \
--chain-id 23 \
--threshold-numerator 2 \
--threshold-denominator 3 \
--governance-authority 0x93f21ad1bbaxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx \
--genesis-committee-utxo f44d20261bdxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx#0 \
--registration-utxo 0866ea8d38182585xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx#0 \
--aura-pub-key 0xea57181f5e672dc0811xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx \
--grandpa-pub-key 0xa7545af1501dd0caxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx \
--sidechain-pub-key 0x0289b3b90aae06xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx \
--sidechain-signature c140a25d9e5690xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx \
--spo-public-key 9404e1ef6197d708c86xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx \
--spo-signature c3597af24123140a69dbxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Completar los datos de conexión con Kupo y Ogmios en el volumen docker cuando el script los solicite.

```bash
> Kupo protocol (http/https) http
> Kupo hostname localhost
> Kupo port 1442
> Ogmios protocol (http/https) http
> Ogmios hostname localhost
> Ogmios port 1337
```

###### 6.5 Comprobar el registro

Buscar el número de epoch actual.

```bash
curl -L -X POST -H "Content-Type: application/json" -d '{
      "jsonrpc": "2.0",
      "method": "sidechain_getStatus",
      "params": [],
      "id": 1
    }' https://rpc.testnet.midnight.network | jq


"epoch": 799,
```

Buscar la llave publica de la sidechain

```bash
cat partner-chains-public-keys.json | jq .'sidechain_pub_key'

0x0289b3b90aae065129f4f49860237d7c84e6b3c03650dfcbed67aff694c51d0653
```

Comprobar que nuestra llave pública está registrada para dos epoch posteriores a la del registro,

"params": [801] = sumar 2 al  "epoch": 799

```bash
curl -L -X POST -H "Content-Type: application/json" -d '{
      "jsonrpc": "2.0",
      "method": "sidechain_getAriadneParameters",
      "params": [801],
      "id": 1
    }' https://rpc.testnet.midnight.network | jq | grep 0x0289b3b90aae065129f4f49860237d7c84e6b3c03650dfcbed67aff694c51d0653
```

Resultado

```bash
 % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  118k  100  118k  100   118   284k    283 --:--:-- --:--:-- --:--:--  284k
          "sidechainPubKey": "0x0289b3b90aae065129f4f49860237d7c84e6b3c03650dfcbed67aff694c51d0653",
          "crossChainPubKey": "0x0289b3b90aae065129f4f49860237d7c84e6b3c03650dfcbed67aff694c51d0653",
```

###### 7 Ejecutar el nodo de midnight

clonar el repositorio [GitHub - midnight-ntwrk/midnight-node-docker](https://github.com/midnight-ntwrk/midnight-node-docker.git)

editar el ficheo **.env**

```bash
# con 127.0.0.1 NO funciona la conexión
# IP local de tu equipo tipo 192.168.xxx.xxx
POSTGRES_HOST="192.168.xxx.xxx"
POSTGRES_PASSWORD=<TU_PASSWORD_POSTGRES>
```

```bash
NODE_KEY=""
# Tiene que ser el valor de la llave secret, en el Registro1 se crea en el directorio
# "./data/chains/partner_chains_template/network/secret_ed25519"
```

Descargar e instalar el contenedor con el nodo

```bash
docker compose up -d
```

Comprobaciones

```bash
docker logs midnight-node-docker_midnight-data-testnetdocker logs midnight-node-docker-midnight-node-testnet-1
```

Acceder al shell de un volumen

```bash
docker run -it --rm -v midnight-node-docker_midnight-data-testnet:/node busybox sh
```

Copiar ficheros a un volume que no arranca

###### Opción 1

Copiar las carpetas y los ficheros con la Keys generadas en el proceso Register1

```bash
# ejecutar el volumen con un contenedor temporal busybox y con nombre helper
docker run -v midnight-node-docker_midnight-data-testnet:/node --name helper busybox true

# copiar los ficheros
docker cp . helper:/node

# borrar el contendor temporal busybox
docker rm helper
```

Es necesario mover las carpetas y los ficheros a la carpeta /node/chain/chains/testnet

```bash
/node # tree
.
└── chain
    └── chains
        └── testnet
            ├── keystore
            │   ├── 617572xxxxxxxxxxxxxxxxx
            │   ├── 637263xxxxxxxxxxxxxxxxx
            │   └── 677261xxxxxxxxxxxxxxxxx
            └── network
                └── secret_ed25519
```

###### Opción 2

Editar el fichero **compose.yml**

Añadir la ruta de origen y la de destino del directorio con las keys

```yml
healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9944/health"]
      interval: 10s #added line
      timeout: 10s #added line
      retries: 3 #added line

volumes:
      - midnight-data-testnet:/node
      - /optnode/midnight/partner-chain-cli/data/chains/partner_chains_template:/node/chain/chains/testnet
    user: root
    restart: always
```

###### 8 Consultar bloques minados

```bash
docker logs midnight-node-docker-midnight-node-testnet-1 > log.txt 2>&1 && cat -n log.txt | grep -i "Prepared" | wc -l
0
```

  

###### 9 Varios

Acceder al shell de un contendor

```bash
docker exec -t -i midnight-node-docker-midnight-node-testnet-1 /bin/bash
```



Probar la conexión a postgres

```bash
psql -h localhost -U postgres -d cexplorer -p 5432
psql -h 127.0.0.1 -U postgres -d cexplorer -p 5432
```



Listar los ID de los contenedores en ejecución

```bash
docker ps --format \
"table {{.ID}}\t{{.Status}}\t{{.Names}}"

e43281a00cd3   Up 29 seconds (health: starting)   midnight-node-docker-midnight-node-testnet-1
5221c8da0d75   Up 2 hours                         db-sync
4649aa462409   Up 2 hours (healthy)               db-sync-postgres

```



Consultar la IP interna de los contenedores

```bash
# Comprobar la IP de un contenedor
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' e43281a00cd3
172.19.0.2
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 4649aa462409
172.18.0.3
```
