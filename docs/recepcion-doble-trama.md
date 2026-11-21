# HackRF LoRa Receiver Fix: Dual Frame Decoding

## Problem Solved
El receptor LoRa no capturaba el segundo mensaje (gateway response) en intercambios ping-pong — solo el primero (nodo).

## Root Cause
`frame_sync_impl.cc:867-871` — frame_sync se queda parado en estado `SFO_COMPENSATION` esperando mensaje async `frame_info` de `header_decoder`, con `items_to_consume=0` (no procesa entrada). Con sample rate bajo (2 MSps), no hay suficientes muestras en buffer durante este tiempo muerto (~500ms con SF=12), y el preamble del gateway se pierde.

## Solution
Aumentar sample rate de 2 MSps a 4 MSps:
- `samp_rate: 2000000 → 4000000`
- Ajustar decimación: `decim: 8 → 16` (para mantener `samp_rate_rx = 250000`)

Con el doble de muestras en el buffer, frame_sync tiene margen suficiente para encontrar el segundo preamble antes de procesar completamente el primero.

## Verificación
✅ Ambos mensajes (nodo + gateway) se decodifican consistentemente.

## Parámetros Finales
```
samp_rate: 4000000 (4 MSps)
low_pass_filter decim: 16
samp_rate_rx: 250000 (unchanged)
minoutbuf: 20000
Gains: bb_gain=30, if_gain=30, gain=30 dB
```

HackRF soporta hasta 20 MSps, así que 4 MSps está bien dentro de rango.
