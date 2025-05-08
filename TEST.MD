## SERVER DEPLOYMENT

Deployment completed successfully.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
RPC is available at: http://194.247.186.149/v1/tick-info
Demo App: http://194.247.186.149:8081
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
To connect to the testnet via qubic-cli, use:
_______________________
|                     |
| IP: 194.247.186.149 |
| Port: 31841         |
|_____________________|
Example commands:
./qubic-cli -nodeip 194.247.186.149 -nodeport 31841 -getcurrenttick
```

**Respuesta:**
```
Tick: 21180008
Epoch: 152
Number Of Aligned Votes: 0
Number Of Misaligned Votes: 0
Initial tick: 21180000
```

### Consultar Balance del Contrato

```bash
./qubic-cli -nodeip 194.247.186.149 -nodeport 31841 -getbalance WEVWZOHASCHODGRVRFKZCGUDGHEDWCAZIZXWBUHZEAMNVHKZPOIZKUEHNQSJ
```

**Respuesta:**
```
Identity: WEVWZOHASCHODGRVRFKZCGUDGHEDWCAZIZXWBUHZEAMNVHKZPOIZKUEHNQSJ
Balance: 1000000000
Incoming Amount: 1000000000
Outgoing Amount: 0
Number Of Incoming Transfers: 1
Number Of Outgoing Transfers: 0
Latest Incoming Transfer Tick: 15502487
Latest Outgoing Transfer Tick: 0
Tick: 21180008
Spectum Digest: f63265df35ed06942aea03852d323a19555b920f0ec2aebe0274c83ab4aebff1
```

## Creación de Órdenes

### Descripción del Proceso

Creo una orden con destino a mi wallet EVM `0x090378a9c80c5E1Ced85e56B2128c1e514E75357`, con un monto de 1000. 
Esto llama a la función 1: `REGISTER_USER_PROCEDURE(createOrder, 1)`

### Código de la Función createOrder

```cpp
PUBLIC_PROCEDURE_WITH_LOCALS(createOrder)
{
    // Validate the input
    if (input.amount == 0)
    {
        locals.log = EthBridgeLogger{
            CONTRACT_INDEX,
            EthBridgeError::invalidAmount,
            0,
            input.amount,
            0};
        LOG_INFO(locals.log);
        output.status = 1; // Error
        output.orderId = 0; // Inicializar a 0 en caso de error
        return;
    }

    if (qpi.invocationReward() < state.transactionFee)
    {
        locals.log = EthBridgeLogger{
            CONTRACT_INDEX,
            EthBridgeError::insufficientTransactionFee,
            0,
            input.amount,
            0};
        LOG_INFO(locals.log);
        output.status = 2; // Error
        output.orderId = 0; // Inicializar a 0 en caso de error
        return;
    }

    // Create the order
    locals.newOrder.orderId = state.nextOrderId++;
    locals.newOrder.qubicSender = qpi.invocator();
    locals.newOrder.ethAddress = input.ethAddress;
    locals.newOrder.amount = input.amount;
    locals.newOrder.orderType = 0; // Default order type
    locals.newOrder.status = 0;    // Created
    locals.newOrder.fromQubicToEthereum = input.fromQubicToEthereum;

    // Store the order
    for (uint64 i = 0; i < state.orders.capacity(); ++i)
    {
        if (state.orders.get(i).status == 255)
        { // Empty slot
            state.orders.set(i, locals.newOrder);

            // Log específico para mostrar claramente el orderID
            char orderIdMsg[64];
            sprintf(orderIdMsg, "ORDER_ID: %llu", locals.newOrder.orderId);
            LOG_INFO(orderIdMsg);
            
            locals.log = EthBridgeLogger{
                CONTRACT_INDEX,
                0,
                locals.newOrder.orderId,
                input.amount,
                0};
            LOG_INFO(locals.log);
            output.status = 0; // Success
            output.orderId = locals.newOrder.orderId; // Devolver el ID de la orden
            return;
        }
    }
}
```

### Comando de Ejecución

```bash
./qubic-cli -nodeip 194.247.186.149 -nodeport 31841 -seed xfccearcehvadeokstomhjsyqaudyhkngbmlwvsxsppexhlhdvijyaq -sendcustomtransaction WEVWZOHASCHODGRVRFKZCGUDGHEDWCAZIZXWBUHZEAMNVHKZPOIZKUEHNQSJ 1 100000000 29 090378a9c80c5e1ced85e56b2128c1e514e75357e80300000000000001
```

### Resultado de la Transacción

```
Transaction has been sent!
~~~~~RECEIPT~~~~~
TxHash: nfxqglsolushkbhgqlxbquahtqsbejsszdofdlkryfmhesgujkgvwpvahvvl
From: XABEFABIHWRWBAIJQJPWTIIQBUCBHBVWYYGFFJADQBKWFBORRVXWSCVBUKWL
To: WEVWZOHASCHODGRVRFKZCGUDGHEDWCAZIZXWBUHZEAMNVHKZPOIZKUEHNQSJ
Input type: 1
Amount: 100000000
Tick: 21180037
Extra data size: 29
Extra data: 090378a9c80c5e1ced85e56b2128c1e514e75357e80300000000000001
MoneyFlew: N/A
~~~~~END-RECEIPT~~~~~
```

Para verificar el estado de la transacción:
```bash
./qubic-cli [...] -checktxontick 21180037 nfxqglsolushkbhgqlxbquahtqsbejsszdofdlkryfmhesgujkgvwpvahvvl
```

> **Nota**: La salida debería proporcionar el ID de la orden, pero no lo hace. Como es la primera orden, su ID sería 0.

### Verificación de la Transacción

```bash
./qubic-cli -nodeip 194.247.186.149 -nodeport 31841 -checktxontick 21180037 nfxqglsolushkbhgqlxbquahtqsbejsszdofdlkryfmhesgujkgvwpvahvvl
```

**Respuesta:**
```
Found tx nfxqglsolushkbhgqlxbquahtqsbejsszdofdlkryfmhesgujkgvwpvahvvl on tick 21180037
Received end response message
```

### Análisis del Tick

#### Comandos para Obtener y Analizar Datos del Tick

```bash
# Obtener datos del tick
./qubic-cli -nodeip 194.247.186.149 -nodeport 31841 -gettickdata 21180037 21180037.bin

# Obtener lista de computadores
./qubic-cli -nodeip 194.247.186.149 -nodeport 31841 -getcomputorlist computorlist.bin

# Leer los datos del tick
./qubic-cli -readtickdata 21180037.bin computorlist.bin
```

#### Resultado del Análisis

```
List of transactions on tickData (correct order):
5c64c7660c22492fadfbee2c0b1129f87b8a5a0dd0de873354ec784e39f8e0d9
8b740116cdd711302b028e8f6cb21f3bb6ae9d98a1cbd5cc421fa272a2b49e1c
Computor list is NOT verified
Tickdata: ggiiiuwiptswxbvvxbjrebhtjyfbzhvriwijgoxalevchrsxvvkvmoogpuuh
Tick is VERIFIED (signed by correct computor).
Comp ID: RPZXQWFBIFQBGBQDSISJNNHFOIODRSGOVCECHEMFVEPYHGNBEYZJXZBFTOQF
Epoch: 152
Tick: 21180037
Computor index: 281
Datetime: 8-5-25 12:15:39.0
```

#### Transacción del Sistema

```
~~~~~RECEIPT~~~~~
TxHash: sxfusyanhpisjbxpeugawzamzjfhjbkmpvkbrlrxmbwnjrktqttlmnigulyn
From: RPZXQWFBIFQBGBQDSISJNNHFOIODRSGOVCECHEMFVEPYHGNBEYZJXZBFTOQF
To: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAFXIB
Input type: 1
Amount: 0
Tick: 21180037
Extra data size: 848
Extra data: 0842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084000842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421084210842108421000000
MoneyFlew: N/A
~~~~~END-RECEIPT~~~~~
Transaction is VERIFIED
```

#### Nuestra Transacción

```
~~~~~RECEIPT~~~~~
TxHash: nfxqglsolushkbhgqlxbquahtqsbejsszdofdlkryfmhesgujkgvwpvahvvl
From: XABEFABIHWRWBAIJQJPWTIIQBUCBHBVWYYGFFJADQBKWFBORRVXWSCVBUKWL
To: WEVWZOHASCHODGRVRFKZCGUDGHEDWCAZIZXWBUHZEAMNVHKZPOIZKUEHNQSJ
Input type: 1
Amount: 100000000
Tick: 21180037
Extra data size: 29
Extra data: 090378a9c80c5e1ced85e56b2128c1e514e75357e80300000000000001
MoneyFlew: N/A
~~~~~END-RECEIPT~~~~~
Transaction is VERIFIED
```

## Completar Órdenes

### Problema con el Comando

Debería usar `REGISTER_USER_PROCEDURE(completeOrder, 6)`, pero el cliente de línea de comandos devuelve el error: **unknown input type 6**

### Código de la Función completeOrder

```cpp
PUBLIC_PROCEDURE_WITH_LOCALS(completeOrder)
    locals.invocatorAddress = qpi.invocator();
    locals.isManagerOperating = false;
    CALL(isManager, locals.invocatorAddress, locals.isManagerOperating);

    // Check if the order is handled by a manager
    locals.orderFound = false;
    for (uint64 i = 0; i < state.orders.capacity(); ++i)
    {
        if (state.orders.get(i).orderId == input.orderId)
        {
            locals.order = state.orders.get(i);
            locals.orderFound = true;
            break;
        }
    }
```

### Intento de Ejecución

```bash
./qubic-cli -nodeip 194.247.186.149 -nodeport 31841 -seed xfccearcehvadeokstomhjsyqaudyhkngbmlwvsxsppexhlhdvijyaq -sendcustomtransaction WEVWZOHASCHODGRVRFKZCGUDGHEDWCAZIZXWBUHZEAMNVHKZPOIZKUEHNQSJ 6 0 8 0000000000000000
```

**Error recibido:**
```
unknown input type 6
```

### Balance del Contrato

```bash
./qubic-cli -nodeip 194.247.186.149 -nodeport 31841 -getbalance WEVWZOHASCHODGRVRFKZCGUDGHEDWCAZIZXWBUHZEAMNVHKZPOIZKUEHNQSJ

Identity: WEVWZOHASCHODGRVRFKZCGUDGHEDWCAZIZXWBUHZEAMNVHKZPOIZKUEHNQSJ
Balance: 1200000000
Incoming Amount: 1200000000
Outgoing Amount: 0
Number Of Incoming Transfers: 3
Number Of Outgoing Transfers: 0
Latest Incoming Transfer Tick: 21180215
Latest Outgoing Transfer Tick: 0
Tick: 21182346
Spectum Digest: 1835825deec56a6173f668665b842433fc14e8dc6a78926ef8def32b444a16b0
```
