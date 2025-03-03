# PARTE 2 - INSTALACIÓN DEL NODO MIDNIGTH EN LA RED DE TEST

version 1
updated 2025/03/03

#### Disclaimer

Esta guía es tal cual, no pretende ser un proceso infalible, lo que funcione en mi sistema puede que no funcione en el tuyo. Úsala bajo tu responsabilidad.

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

Cambiar configuración para ejecutar docker sin _sudo_

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

Si hay errores en alguno de ellos revisar los usuarios y contraseñas del fichero _compose.yml_

```bash
container_name: db-sync-postgres
    environment:
      - POSTGRES_PASSWORD=<TU_PASSWORD>
      - POSTGRES_DB=cexplorer

 container_name: db-sync
    environment:
      - POSTGRES_PASSWORD=<TU_PASSWORD>
```

También se puede dar el caso de que falten permisos de lectura/escritura para los directorios del fichero **_.env_**

Comandos docker de interés para la gestión de los contenedores.

```bash
docker container list
docker-compose stop # stop containers
docker-compose start # start containers
docker-compose restart # restart containers
docker-compose down # stop and remove containers
docker-compose stats # display resource usage statistics                                  # (Y|N) Download latest Mithril snapshot
```

`NOTA: Las imágenes de Kupo y Ogmios solo son necesarias para realizar la transacción del registro del pool. Una vez completado el registro se pueden parar.`

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

Alternativa gráfica pgAdmin [Download](https://www.pgadmin.org/download/)

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

Crear un nuevo fichero de configuración para **Midnight testnet** , en _/partner-chains-cli/partner-chains-cli-chain-config.json_

Añadir el siguiente contenido

20/3/225 Actualización a testnet-02

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
    "chain_id": 47,
    "genesis_committee_utxo": "d8774f03b4d44eddca22554fbb24f06bde27f8b7c29c979d79058f76b1e3f604#0",
    "threshold_numerator": 2,
    "threshold_denominator": 3,
    "governance_authority": "93f21ad1bba9ffc51f5c323e28a716c7f2a42c5b51517080b90028a6"
  },
  "cardano_addresses": {
    "committee_candidates_address": "addr_test1wrtrt7v002utktsuhsnm3nlzrrxg94z32zhw4nmtseangrs5l5x9p",
    "d_parameter_policy_id": "51a8d059b9b3d831bad5640ed70c54b2c27f051e6eef3d6faa0be6f1",
    "permissioned_candidates_policy_id": "82b78bff2f2409c778e0faf2a36a7814b886ead316d5110387772f8c"
  },
  "native_token": {
    "asset": {
      "asset_name": "0x",
      "policy_id": "0x00000000000000000000000000000000000000000000000000000000"
    },
    "illiquid_supply_address": "addr_test1aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
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
 --chain-id 47 \
 --threshold-numerator 2 \
 --threshold-denominator 3 \
 --governance-authority 0x93f21ad1bba9ffc51f5c323e28a716c7f2a42c5b51517080b90028a6 \
 --genesis-committee-utxo d8774f03b4d44eddca22554fbb24f06bde27f8b7c29c979d79058f76b1e3f604#0 \
 --registration-utxo c487b26ac1a11fb65d010fff3d347d1ece1dad5e034ba302150c1f0e1c19e1e3#1 \
 --aura-pub-key 0xea89dbf84bd3af102a78ed5ed74027051bf6e503eb026cda7caa6d6bb2524040 \
 --grandpa-pub-key 0x52316d47d422f9a0b96cf4aa6e5783fd44b8b07424375fde0f979772cb5506e9 \
 --sidechain-pub-key 0x027580b1ed7e91a2c6bba6e5fcfd8209c02f64b8070be11f5a2c23de376a645f1f \
 --sidechain-signature bda58a2e0a49a18677719b81084429bd15b264fb53db5bdcb6d85915707c33402ed4496f53fc36518dddbff20254600a498daf249023f4296c06237355c5e6c3
```

/home/<usuario>/cardano-testnet/keys/KES/BP.cold.skey

###### 6.4 Registro 3

Copiar el resultado del registro 2

Ruta del fichero cold.key en el equipo principal no en docker.

/home/sergi/cardano-testnet/keys/midnight/payment.skey

"" _/home//cardano-testnet/keys/KES/BP.cold.skey_"" ->> Error step 3

```bash
./partner-chains-cli register3 \
--chain-id 47 \
--threshold-numerator 2 \
--threshold-denominator 3 \
--governance-authority 0x93f21ad1bba9ffc51f5c323e28a716c7f2a42c5b51517080b90028a6 \
--genesis-committee-utxo d8774f03b4d44eddca22554fbb24f06bde27f8b7c29c979d79058f76b1e3f604#0 \
--registration-utxo c487b26ac1a11fb65d010fff3d347d1ece1dad5e034ba302150c1f0e1c19e1e3#1 \
--aura-pub-key 0xea89dbf84bd3af102a78ed5ed74027051bf6e503eb026cda7caa6d6bb2524040 \
--grandpa-pub-key 0x52316d47d422f9a0b96cf4aa6e5783fd44b8b07424375fde0f979772cb5506e9 \
--sidechain-pub-key 0x027580b1ed7e91a2c6bba6e5fcfd8209c02f64b8070be11f5a2c23de376a645f1f \
--sidechain-signature bda58a2e0a49a18677719b81084429bd15b264fb53db5bdcb6d85915707c33402ed4496f53fc36518dddbff20254600a498daf249023f4296c06237355c5e6c3 \
--spo-public-key 9404e1ef6197d708c86d3159ef4b83f57933da8d437e0deacf9ffe5aa9fadbe5 \
--spo-signature 237076ffd3d234c220a2ad713208aa59b57ab61c410042fbc4801d4970737e5bda78af365488b9d69e1e7da6736ce845a942fe0eb195ce09e0ba7e5352d71d0d
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
    }' https://rpc.testnet-02.midnight.network | jq


"epoch": 859,
```

Buscar la llave publica de la sidechain

```bash
cat partner-chains-public-keys.json | jq .'sidechain_pub_key'

0x027580b1ed7e91a2c6bba6e5fcfd8209c02f64b8070be11f5a2c23de376a645f1f
```

Comprobar que nuestra llave pública está registrada para dos epoch posteriores a la del registro,

"params": [861] = sumar 2 al "epoch": 859

```bash
curl -L -X POST -H "Content-Type: application/json" -d '{
      "jsonrpc": "2.0",
      "method": "sidechain_getAriadneParameters",
      "params": [861],
      "id": 1
    }' https://rpc.testnet-02.midnight.network | jq | grep 0x027580b1ed7e91a2c6bba6e5fcfd8209c02f64b8070be11f5a2c23de376a645f1f
```

Resultado

```bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 32672  100 32554  100   118   107k    397 --:--:-- --:--:-- --:--:--  107k
          "sidechainPubKey": "0x027580b1ed7e91a2c6bba6e5fcfd8209c02f64b8070be11f5a2c23de376a645f1f",
          "crossChainPubKey": "0x027580b1ed7e91a2c6bba6e5fcfd8209c02f64b8070be11f5a2c23de376a645f1f",
```

###### 7 Ejecutar el nodo de midnight

clonar el repositorio [GitHub - midnight-ntwrk/midnight-node-docker](https://github.com/midnight-ntwrk/midnight-node-docker.git)

editar el ficheo **.env**

```bash
# con 127.0.0.1 NO funciona la conexión
# IP local de tu equipo tipo 192.168.xxx.xxx
POSTGRES_HOST="192.168.xxx.xxx"
POSTGRES_PASSWORD=<TU_PASSWORD_POSTGRES>
CFG_PRESET=testnet-02
```

```bash
NODE_KEY=""
# Tiene que ser el valor de la llave secret, en el Registro1 se crea en el directorio
# "./data/chains/partner_chains_template/network/secret_ed25519"
CARDANO_DATA_DIR=/optnode/midnight/data
# directorio donde se almacenaran los datos del nodo
```

Descargar e instalar el contenedor con el nodo

```bash
docker compose -f ./compose.yml up -d
```

Comprobaciones

```bash
docker logs midnight
docker logs midnight -follow
```

editar el ficheo **compose.yml**

Asociar el directorio local para el nodo.

```yml
volumes:
  - /optnode/midnight/data/node:/node
```

Ejecutar los contenedores necesarios

```bash
docker compose -f ./compose-partner-chains.yml up -d
```

Comprobaciones

```bash
docker ps --format "table {{.ID}}\t{{.Status}}\t{{.Names}}"

CONTAINER ID   STATUS                  NAMES
a8ea7138185f   Up 32 hours (healthy)   midnight
f9a32ab2bb5e   Up 2 days               db-sync
963c3ea8f14c   Up 2 days (healthy)     kupo
3ec18101ec08   Up 2 days (healthy)     db-sync-postgres
50eb365fc55d   Up 2 days (healthy)     ogmios
de4d9435f112   Up 2 days               cardano-node
```

###### 8 VARIOS

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
  - /optnode/midnight/data/node:/node
```

para revisarEn testnet-01 estaba incluido en la config de volumes

```yml
 - /optnode/midnight/partner-chain-cli/data/chains/partner_chains_template:/node/chain/chains/testnet
 user: root
 restart: always
```

###### 8 Consultar bloques minados

```bash
docker logs midnight > log.txt 2>&1 && cat -n log.txt | grep -i "Prepared" | wc -l
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
