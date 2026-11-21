# lora-sdr-receiver

Receptor LoRa con un **HackRF One** y **GNU Radio**. Escucha en 433,3 MHz, decodifica las tramas LoRa y las imprime por consola, con espectro y cascada para ver la señal en vivo.

Usa el decodificador de [gr-lora_sdr](https://github.com/tapparelj/gr-lora_sdr) (EPFL) y funciona para capturar intercambios completos nodo ↔ gateway: los dos mensajes de un ping-pong se decodifican.

## Configuración

| Parámetro | Valor |
|---|---|
| Frecuencia | 433,3 MHz |
| Sample rate del HackRF | 4 MSps |
| Decimación (pasabajos) | 16, queda en 250 kSps |
| Ancho de banda LoRa | 62,5 kHz |
| Spreading factor | 12 |
| Coding rate | 4/5 |
| CRC | sí, header explícito |
| Sync word | `0x12` |
| Ganancias (RF / IF / BB) | 20 / 20 / 20 dB |

Todo se cambia desde GNU Radio Companion abriendo `lora.grc`, o directamente en las variables de `lora.py`.

## Requisitos

- GNU Radio 3.10
- [gr-osmosdr](https://github.com/osmocom/gr-osmosdr) con soporte para HackRF (`sudo apt install gr-osmosdr hackrf`)
- [gr-lora_sdr](https://github.com/tapparelj/gr-lora_sdr), compilado e instalado:

```sh
git clone https://github.com/tapparelj/gr-lora_sdr
cd gr-lora_sdr && mkdir build && cd build
cmake .. && make -j"$(nproc)" && sudo make install && sudo ldconfig
```

## Uso

Con el HackRF conectado:

```sh
python3 lora.py            # corre el receptor
gnuradio-companion lora.grc  # o abrilo para editarlo
```

Si cambiás el flowgraph en GRC, regenerá `lora.py` con el botón *Generate* (o `grcc lora.grc`).

## Por qué 4 MSps

Con 2 MSps se perdía la respuesta del gateway: el bloque `frame_sync` se queda esperando la info del header del primer mensaje y, con SF12, en ese tiempo no hay suficientes muestras en el buffer para encontrar el preámbulo del segundo. Duplicar el sample rate le da margen. El detalle está en [docs/recepcion-doble-trama.md](docs/recepcion-doble-trama.md).

## Licencia

[GPL-3.0](LICENSE), igual que gr-lora_sdr y GNU Radio.
