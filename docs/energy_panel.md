# Energy Panel – MQTT Specification

## Visão Geral
Este documento define o **contrato oficial de comunicação MQTT** para o projeto **Energy Panel**, responsável por medir corrente, tensão e potência elétrica (ex: usando SCT013) e publicar os dados para um servidor central.

A ESP32 **não toma decisões de negócio** (ligado/desligado). Ela apenas mede, detecta mudanças físicas e publica telemetria.

> **ESP mede → MQTT transporta → Servidor decide**

---

## Arquitetura Geral

```
+-------------------+        MQTT        +------------------+
| ESP32 Energy Node |  ───────────────▶  |      Servidor    |
| (SCT013)          |                    |      (Backend)   |
+-------------------+                   +-------------------+
```

- Um ESP32 pode possuir **múltiplos canais** (sensores)
- Cada canal representa um ar-condicionado
- Cada canal publica em **tópicos independentes**

---

## Identificação do Dispositivo

- **MAC**: identificador único


## Tópicos MQTT

Todos os tópicos seguem o padrão:

```
infranect/energy/<mac>/...
```

---

## Discovery (Autodescrição)

Publicado **uma única vez** ao conectar no MQTT.

### Tópico

```
infranect/energy/<mac>/discovery
```

### Payload

```json
{
  "device": "energy_panel",
  "channels": 4,
  "sensor": "SCT013-100A",
  "fw": "1.0.0"
}
```

### Regras
- `retain = true`
- Não reenviar continuamente

---

## Telemetria por Canal

Cada canal possui **seu próprio tópico**.

### Tópico

```
infranect/energy/<mac>/channels/<channel_id>/telemetry
```

Exemplo:

```
infranect/energy/panel01/channels/1/telemetry
```

### Payload

```json
{
  "current": 2.31,
  "voltage": 220.1,
  "power": 508.2,
  "ts": 1734219123
}
```

### Campos

| Campo | Tipo | Descrição |
|------|------|----------|
| current | float | Corrente (A) |
| voltage | float | Tensão (V) |
| power | float | Potência (W) |
| ts | int | Timestamp epoch (opcional) |

---

## Estratégia de Envio

### Envio Periódico (Heartbeat)

- Intervalo recomendado: **30s ou 60s**
- Publica o estado atual mesmo sem mudanças

### Envio por Evento (Mudança Física)

Publicar imediatamente quando:

```
|current_atual - current_anterior| > LIMIAR
```

Exemplo de limiar:

```c
#define CURRENT_DELTA 0.3 // Amperes
```

---

## Qualidade de Serviço (QoS)

| Tipo | QoS | Retain |
|----|----|----|
| Telemetria | 0 | false |
| Status | 1 | true |
| Discovery | 1 | true |

---

## O que a ESP **NÃO** deve fazer

❌ Decidir se o ar-condicionado está ligado
❌ Inferir estado lógico
❌ Correlacionar múltiplos canais
❌ Executar regras de negócio

---

## Exemplo de Fluxo Completo

```
1. ESP IR envia comando
2. Servidor registra 
3. ESP Energy publica corrente
4. Servidor detecta aumento > limiar
5. Servidor confirma sucesso
```

---

## Escalabilidade

Este modelo suporta:
- Múltiplos painéis
- Dezenas de canais por painel
- Sem alterar firmware.

---

## Versão do Documento

- **v1.0.0**
- Projeto: Energy Panel / SmartIR
- Autor: Marcelo Alves

