# Design: Mezclar y Unificar elitepass-monitor con Zabbix (Monitoreo Nativo Trapper)

## Arquitectura de la Fusión

El bot `elitepass-monitor` asume el rol del agente de monitoreo, convirtiéndose en el recolector principal para ambos canales de salida (Telegram y Zabbix Server):

```
┌─────────────────────────────────────────────────────────────────┐
│                    elitepass-monitor (NodeJS)                   │
│                                                                 │
│   1. Recopila Métricas (CPU, RAM, Disco, Docker, PM2)           │
│   2. Procesa alertas locales                                    │
└─────────────────────────────────────────────────────────────────┘
          │                                           │
          │ Protocolo TCP Trapper (ZBXD\x01)          │ Mensajes HTML + Botones
          ▼                                           ▼
┌──────────────────┐                        ┌──────────────────┐
│  Zabbix Server   │                        │  Telegram Bot    │
│  (10051 Trapper) │                        │  API (Telegram)  │
└──────────────────┘                        └──────────────────┘
```

---

## Protocolo Zabbix Trapper en Node.js (TypeScript)

El bot enviará las métricas directamente al puerto de Trapper (`10051`) de Zabbix Server utilizando el formato binario nativo de Zabbix:

### Formato de Paquete Binario
1. **Cabecera:** `ZBXD\x01` (5 bytes: ASCII 'Z','B','X','D' y versión 0x01).
2. **Longitud:** 8 bytes (entero de 64 bits en Little Endian) con el tamaño del JSON de datos.
3. **Cuerpo:** JSON string codificado en UTF-8 conteniendo las métricas.

### Implementación del Emisor en TypeScript (`src/zabbix-sender.ts`)
```typescript
import net from 'net';

export interface ZabbixMetric {
  host: string;
  key: string;
  value: string | number;
}

export async function sendToZabbix(serverIp: string, port: number, metrics: ZabbixMetric[]): Promise<string> {
  return new Promise((resolve, reject) => {
    const payload = JSON.stringify({
      request: 'sender data',
      data: metrics.map(m => ({
        host: m.host,
        key: m.key,
        value: String(m.value)
      }))
    });

    const payloadBuffer = Buffer.from(payload, 'utf-8');
    const header = Buffer.from([0x5A, 0x42, 0x58, 0x44, 0x01]); // "ZBXD\x01"
    
    // Crear buffer de longitud de 8 bytes (64-bit Little Endian)
    const lengthBuffer = Buffer.alloc(8);
    lengthBuffer.writeUInt32LE(payloadBuffer.length, 0); 

    const requestBuffer = Buffer.concat([header, lengthBuffer, payloadBuffer]);

    const client = new net.Socket();
    let response = '';

    client.connect(port, serverIp, () => {
      client.write(requestBuffer);
    });

    client.on('data', (data) => {
      response += data.toString('utf-8');
    });

    client.on('close', () => {
      resolve(response);
    });

    client.on('error', (err) => {
      reject(err);
    });
  });
}
```

---

## Puntos de Integración en el Cron Loop (`src/index.ts`)

En el ciclo del cron de Health Check actual (cada 5 minutos), el bot no solo evaluará el estado interno para alertas inmediatas, sino que también despachará la telemetría recolectada hacia Zabbix Server:

```typescript
// En index.ts:
cron.schedule(healthInterval, async () => {
  console.log('🔍 Health check...');
  // 1. Evalúa y envía alertas a Telegram (como lo hace ahora)
  await monitor.sendHealthAlert();
  
  // 2. Extrae las métricas recopiladas y las envía a Zabbix Server
  if (process.env.ZABBIX_SERVER_IP) {
     await monitor.pushMetricsToZabbix();
  }
});
```

---

## Configuración del Servidor Zabbix
En la interfaz web de Zabbix Server, se creará el Host `VM00-Reservas-v2` y se añadirán items de tipo **Zabbix Trapper** con claves correspondientes (ej. `system.cpu.usage`, `system.ram.usage`, `pm2.apps.offline`, etc.). Esto permite a Zabbix recibir los datos históricos de forma pasiva sin necesidad de consultar activamente a la VM.
