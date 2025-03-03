# PARTE 1 - INSTALACIÓN DEL NODO BLOCK PRODUCER EN LA RED DE CARDANO

#### Disclaimer

Esta guía es tal cual, no pretende ser un proceso infalible, lo que  funcione en mi sistema puede que no funcione en el tuyo. Úsala bajo tu responsabilidad.

Fuentes:

- [Handbook | cardano course](https://cardano-course.gitbook.io/cardano-course/handbook)

- cardano-node-wiki: [cardano-node-wiki/docs/getting-started/install.md at main · input-output-hk/cardano-node-wiki · GitHub](https://github.com/input-output-hk/cardano-node-wiki/blob/main/docs/getting-started/install.md)

- URL releases: [Releases · IntersectMBO/cardano-node · GitHub](https://github.com/IntersectMBO/cardano-node/releases)

Guía de instalación:

- **guild-operators** https://cardano-community.github.io/guild-operators/

#### 1 Descagar el instalador.

`# Ubuntu / Debian - sudo apt -y install curl`

```bash
curl -sS -o guild-deploy.sh https://raw.githubusercontent.com/cardano-community/guild-operators/master/scripts/cnode-helper-scripts/guild-deploy.sh

chmod 755 guild-deploy.sh
```

#### 2 Instalar el nodo y las dependencias y las herramientas

```bash
./guild-deploy.sh -b master -n preview -p /optnode/preview -t cnode -s pdlcowx`
. "${HOME}/.bashrc"
```

#### 3 Comprobar instalación

```bash
cardano-cli –version

cardano-node –version
```

#### 4 Cambiar preferencias

Editar el fichero de configuración para indicar ciertos datos necesarios para ejecutar el nodo.

```bash
gedit $CNODE_HOME/scripts/env
```

```tex
CNODEBIN="${HOME}/.local/bin/cardano-node"             # Override automatic detection of cardano-node executable
CCLI="${HOME}/.local/bin/cardano-cli"                  # Override automatic detection of cardano-cli executable
CNCLI="${HOME}/.local/bin/cncli"                       # Override automatic detection of executable (https://github.com/AndrewWestberg/cncli)
CNODE_HOME="/optnode/preview/cnode"                        # Override default CNODE_HOME path (defaults to /opt/cardano/cnode)
CNODE_PORT=6000                                        # Set node port
#CONFIG="${CNODE_HOME}/files/config.json"               # Override automatic detection of node config path
SOCKET="${CNODE_HOME}/sockets/node.socket"            # Override automatic detection of path to socket

POOL_NAME="MIDBP"                                       # Set the pool's name to run node as a core node (the name, NOT the ticker, ie folder name)
```

Activar el uso de Mithril para agilizar la sincronización de la cadena.

```tex
MITHRIL_DOWNLOAD="Y"                                   # (Y|N) Download latest Mithril snapshot
```

#### 5 Ejecutar el nodo

```bash
cd "${CNODE_HOME}"/scripts

./cnode.sh
```

#### 6 revisar

`Listening on [http://127.0.0.1:12798](http://127.0.0.1:12798/)`

Consultar el estado de la descarga de la cadena

```bash
cardano-cli query tip --testnet-magic 2
```

Resultado

```json
{
    "block": 271042,
    "epoch": 65,
    "era": "Babbage",
    "hash": "2f9d1067cfcd5929bc430bb84c42325e740124f7a08383994c200398bff68b71",
    "slot": 5652856,
    "slotInEpoch": 36856,
    "slotsToEpochEnd": 49544,
    "syncProgress": "8.26"
}
```

#### 7 Crear llaves.

-Fuente: https://developers.cardano.org/docs/operate-a-stake-pool/generating-wallet-keys

```bash
mkdir -p $HOME/cardano-testnet/keys
cd $HOME/cardano-testnet/keys
```

Crear llaves con el nombre BP (Block Producer)

Fuente:

- [Discord](https://discord.com/channels/1165826384975908924/1253707025989238826/1322149177198907444)

- [Direcciones de payment y stake | TOPO⚡](https://es-kb.topopool.com/primeros-pasos/direcciones-payment-y-stake)

- https://developers.cardano.org/docs/operate-a-stake-pool/generating-wallet-keys

En un equipo desconectado y en un directorio propio.

1. ##### Crear llaves de pago para el nodo

```bash
cardano-cli conway address key-gen     
--verification-key-file payment.vkey    
--signing-key-file payment.skey
```

```tex
-rw------- 1 sxxxx sxxxx  180 dic 25 11:21 payment.skey
-rw------- 1 sxxxx sxxxx  190 dic 25 11:21 payment.vkey
```

2. ###### Crear llaves para el stake

```bash
cardano-cli conway address key-gen \
    --verification-key-file stake.vkey \
    --signing-key-file stake.skey
```

```tex
-rw------- 1 sxxxx sxxxx  180 dic 25 11:54 stake.skey
-rw------- 1 sxxxx sxxxx  190 dic 25 11:54 stake.vkey
```

3. Creando la dirección de stake

```bash
cardano-cli conway stake-address build --staking-verification-key-file stake.vkey  --out-file stake.addr --testnet-magic 2
```

```tex
-rw------- 1 sxxxx sxxxx   64 dic 29 18:14 stake.addr
stake_test1urduhqhzr4kkwk8wgh9t58leyeuph9qqfkq3cryrwxjpjaq4v6y4c
```

4. Creando la dirección de pago

```bash
cardano-cli conway address build \
    --payment-verification-key-file payment.vkey \
    --stake-verification-key-file stake.vkey \
    --out-file payment.addr \--testnet-magic 2
```

```tex
-rw------- 1 sxxxx sxxxx   63 dic 25 11:56 payment.addr
addr_test1qrqu02xjrexw5x7vynjw7zw3fhv0pwhrj0jqkwgxfn7v0xkmewpwy8tdvavwu3w2hg0ljfncrw2qqnvprsxgxudyr96qm3s5h3
```

Consultar saldo de la billetera

```bash
cardano-cli query utxo --address addr_test1qrqu02xjrexw5x7vynjw7zw3fhv0pwhrj0jqkwgxfn7v0xkmewpwy8tdvavwu3w2hg0ljfncrw2qqnvprsxgxudyr96qm3s5h3 --testnet-magic 2
```

Al crear la billetera está vacía

```tex
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
```

5. ###### Reclamar tADA en el Faucet

https://docs.cardano.org/cardano-testnets/tools/faucet

TxHash: **7e0c0c05bf4603dca047b9c7f78d6d30883ece529e2e520cb21ed2e45da78bde**

```tex
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
7e0c0c05bf4603dca047b9c7f78d6d30883ece529e2e520cb21ed2e45da78bde     0        10000000000 lovelace + TxOutDatumNone
```

6. ###### Crear el certificado del stake
   
   Primero consultar el coste del registro
   
   ```bash
   cardano-cli conway query protocol-parameters --testnet-magic 2 --out-file protocol.json
   
   cat protocol.json | jq .stakeAddressDeposit
   2000000
   ```

```bash
cardano-cli conway stake-address registration-certificate    --stake-verification-key-file stake.vkey --key-reg-deposit-amt 2000000 --out-file stake.cert
```

```tex
rw------- 1 sxxxx sxxxx   194 dic 31 08:43  stake.cert
```

7. ###### Registro de la dirección de stake
   
   Crear una transacción
   
   ```bash
   expr 10000000000 - 179552 - 2000000
   9997820448
   
   cardano-cli conway transaction build-raw --tx-in 7e0c0c05bf4603dca047b9c7f78d6d30883ece529e2e520cb21ed2e45da78bde#0 \
     --tx-out addr_test1qrqu02xjrexw5x7vynjw7zw3fhv0pwhrj0jqkwgxfn7v0xkmewpwy8tdvavwu3w2hg0ljfncrw2qqnvprsxgxudyr96qm3s5h3+9997820448 \
     --fee 179552 \
     --certificate-file stake.cert \
     --out-file tx.raw
   ```
   
   Firmar transacción
   
   ```bash
   cardano-cli conway transaction sign \
     --tx-body-file tx.raw \
     --signing-key-file payment.skey \
     --signing-key-file stake.skey \
     --testnet-magic 2 \
     --out-file tx.signed
   ```
   
   Enviar transacción
   
   ```bash
   cardano-cli conway transaction submit \
     --tx-file tx.signed \
     --testnet-magic 2
   
   Transaction successfully submitted.
   ```

8. ###### Crear llave KES

```bash
cardano-cli conway node key-gen-KES \
    --verification-key-file BP.kes.vkey \
    --signing-key-file BP.kes.skey
```

```tex
-rwx------ 1 sxxxx sxxxx 1327 dic 25 12:04 BP.kes.skey*
-rw------- 1 sxxxx sxxxx  183 dic 25 12:04 BP.kes.vkey
```

9. Crear llaves cold y Kes.counter

```bash
cardano-cli conway node key-gen \
    --cold-verification-key-file BP.cold.vkey \
    --cold-signing-key-file BP.cold.skey \
    --operational-certificate-issue-counter-file BP.counter
```

```tex
-rw------- 1 sxxxx sxxxx  187 dic 31 09:33 BP.cold.skey
-rw------- 1 sxxxx sxxxx  197 dic 31 09:33 BP.cold.vkey
-rw------- 1 sxxxx sxxxx  203 dic 31 09:33 BP.counter
```

10. Consultar datos para el certificado
    
    ```bash
    cardano-cli conway query tip --testnet-magic 2 | jq .slot
    68978879
    cat /optnode/preview/cnode/files/shelley-genesis.json | jq .slotsPerKESPeriod
    129600
    expr 68978879 / 129600
    532
    ```

11. Crear el certificado
    
    ```bash
    cardano-cli conway node issue-op-cert \
        --kes-verification-key-file BP.kes.vkey \
        --cold-signing-key-file BP.cold.skey \
        --operational-certificate-issue-counter BP.counter \
        --kes-period 532 \
        --out-file BP.node.opcert
    ```
    
    ```tex
    -rw------- 1 sxxxx sxxxx  367 dic 31 09:57 BP.node.opcert
    ```

12. Crear llavers VRF

```tex
cardano-cli node key-gen-VRF \
    --verification-key-file BP.vrf.vkey \
    --signing-key-file BP.vrf.skey

chmod 400 BP.vrf.skey
-r-------- 1 sxxxx sxxxx  230 dic 31 09:36 BP.vrf.skey
-rw------- 1 sxxxx sxxxx  176 dic 31 09:36 BP.vrf.vkey
```

Modificar los fichero a Solo Lectura

```bash
chmod -R 400 BP.*.skey
chmod -R 400 BP.*.vkey
chmod -R 400 BP.counter
chmod -R 400 BP.node.opcert
```

13. Revisar el ID del stakepool
    
    ```bash
    cardano-cli conway stake-pool id --cold-verification-key-file BP.cold.vkey --out-file stakepoolid.txt
    
    -rw------- 1 sxxxx sxxxx   56 dic 31 09:43 stakepoolid.txt
    pool1q4h9ed20wv8549dw5yvr3xw5ll5fg0v72s998nyjs6uc6dark6l
    ```
    
    ```bash
    cardano-cli conway query stake-snapshot --stake-pool-id $(cat stakepoolid.txt) --testnet-magic 2 
    {
        "pools": {
            "056e5cb54f730f4a95aea1183899d4ffe8943d9e540a53cc9286b98d": {
                "stakeGo": 0,
                "stakeMark": 0,
                "stakeSet": 0
            }
        },
        "total": {
            "stakeGo": 656721852629195,
            "stakeMark": 661373235028825,
            "stakeSet": 657531712348951
        }
    }
    ```

#### 8 Crear un servicio para ejecuta el nodo

```bash
cd $CNODE_HOME/scripts
./cnode.sh -d
```

Arrancar el servicio

```bash
sudo systemctl start cnode.service
```

Ver el log del servicio

```bash
journalctl --unit=cnode --follow
```

#### 9 Monitorizar con gLiveView

```bash
cd $CNODE_HOME/scripts
./gLiveView.sh
```



#### 10 Convertir el nodo Relay en Block Producer

Fuente: https://developers.cardano.org/docs/operate-a-stake-pool/block-producer-keys/

Copiar las llaves generadas a la ruta del nodo.

```bash
cp  $HOME/cardano-testnet/keys/KES/BP.kes.skey $CNODE_HOME/priv/pool/MIDBP/kes.skey
cp  $HOME/cardano-testnet/keys/KES/BP.vrf.skey $CNODE_HOME/priv/pool/MIDBP/vrf.skey
cp  $HOME/cardano-testnet/keys/BP.node.opcert $CNODE_HOME/priv/pool/MIDBP/node.cert
```

Cambiar nombres en el fichero .env

```bash
gedit $CNODE_HOME/scripts/env 

WALLET_FOLDER="${CNODE_HOME}/priv/wallet"              
POOL_FOLDER="${CNODE_HOME}/priv/pool"                                                                         
POOL_NAME="MIDBP"            

POOL_HOTKEY_SK_FILENAME="kes.skey"
POOL_OPCERT_FILENAME="node.cert"
POOL_VRF_SK_FILENAME="vrf.skey"
```

Reiniciar el servicio y comprobar si se ejecuta el nodo como Block Producer

```bash
sudo systemctl restart cnode.service
```

Se puede comprobar que el tipo de nodo ya es Core

Y que la información del periodo de renovación de las llaves KES se muestra.



#### 11 Registrar el stake

Fuente: https://developers.cardano.org/docs/operate-a-stake-pool/register-stake-address

#### 12 Registrar el pool

Fuente: https://developers.cardano.org/docs/operate-a-stake-pool/register-stake-pool
