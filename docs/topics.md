# Infranect – MQTT Topics Specification

Este documento define o **padrão oficial de tópicos MQTT** usados no ecossistema **Infranect**.

## Convenções Gerais

### Tópicos

```
<namespace>/<domain>/<mac>/...
```

| Elemento | Descrição |
|--------|-----------|
| `namespace` | Organização ou projeto (`infranect`, `smartIR`) |
| `domain` | Contexto funcional (`energy`, `ir`, `ac`, `temperature`) |
| `mac` | Identificador único do device |

---

# Device Discovery

### 📤 Publicador
- Firmware (ESP)

### 📥 Consumidor
- Backend Django

### 📄 Payload
```json
{
  "device_type": "ir_bridge",
  "mac":"AA:BB:CC:DD:EE:FF",
  "firmware_version": "1.0.0",
  "ip_address": "192.168.0.100",
  "wifi_signal": -74,
  "last_seen": "1734219123",
}
```
# 📺 AC universal AC
Envio de comando AC

**Backend → ESP**
### Tópico

```
infranect/<mac>/ac/send
```

### 📄 Payload
```json
{
  "protocol_id": 10,
  "power": true,
  "temperature": 23,
  "mode": "cool",
  "fan": "auto",
  "swing": true
}
```
--- 
# 📺 Infravermelho (IR)
Envio de comando IR

**Backend → ESP**
### Tópico

```
infranect/<mac>/ir/send
```

### 📄 Payload
```json
{
  "protocol_id": 22,
  "command_hex": 0x20DF,
  "bits": 32
}
```
--- 
# 🌡️ Temperatura

**ESP → Backend**

### Tópico

```
infranect/mac/temperature
```

### 📄 Payload
```json
{
  "value": 24.6,
  "unit": "celsius"
}
```