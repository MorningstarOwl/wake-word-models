# Micro Wake Word Models for Home Assistant Voice PE

Custom [microWakeWord](https://github.com/OHF-Voice/micro-wake-word) models for on-device wake word detection on [Home Assistant](https://www.home-assistant.io/) voice hardware.

These models run directly on the ESP32-S3 chip — no cloud services, no streaming to a server. Wake word detection happens entirely on the device itself.

## Available Models

| Wake Word | Language | Model |
|-----------|----------|-------|
| **Nocturna** | English | [`models/nocturna/`](models/nocturna/) |

## Usage with Home Assistant Voice PE

Add the model to your Voice PE's ESPHome configuration by overriding the `micro_wake_word` section:

```yaml
substitutions:
  name: home-assistant-voice-XXXXXX
  friendly_name: My Voice PE

packages:
  Nabu Casa.Home Assistant Voice PE:
    github://esphome/home-assistant-voice-pe/home-assistant-voice.yaml

esphome:
  name: ${name}
  name_add_mac_suffix: false
  friendly_name: ${friendly_name}

api:
  encryption:
    key: "YOUR_API_KEY"

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

micro_wake_word:
  id: mww
  models:
    - model: https://github.com/MorningstarOwl/wake-word-models/raw/main/models/nocturna/nocturna.json
      id: nocturna
```

Replace the `name`, `friendly_name`, and `api.encryption.key` with the values from your existing Voice PE config.

After flashing, go to **Settings → Devices & Services → ESPHome**, find your Voice PE, and select **Nocturna** from the wake word dropdown.

## Usage with Other ESPHome Devices

Any ESPHome device with microWakeWord support (ESP32-S3 with PSRAM) can use these models. Add the model to your `micro_wake_word` configuration:

```yaml
micro_wake_word:
  models:
    - model: https://github.com/MorningstarOwl/wake-word-models/raw/main/models/nocturna/nocturna.json
```

See the [ESPHome microWakeWord documentation](https://esphome.io/components/micro_wake_word/) for full configuration options.

## Tuning

If the wake word triggers too easily or not reliably enough, you can override the detection parameters in your ESPHome YAML without re-training:

```yaml
micro_wake_word:
  id: mww
  models:
    - model: https://github.com/MorningstarOwl/wake-word-models/raw/main/models/nocturna/nocturna.json
      id: nocturna
      probability_cutoff: 80%       # Lower = more sensitive (default from manifest: 97%)
      sliding_window_size: 8        # Higher = smoother detection, slightly more latency
```

## How These Models Were Trained

Models are trained locally using the [microWakeWord Trainer](https://github.com/TaterTotterson/microWakeWord-Trainer-Nvidia-Docker) Docker container, which automates the full pipeline:

1. Synthetic speech samples are generated using [Piper TTS](https://github.com/rhasspy/piper)
2. Samples are augmented with room simulation, background noise, and speed variation
3. A streaming MixConv neural network is trained with [TensorFlow](https://www.tensorflow.org/)
4. The model is quantized and exported to TFLite for deployment on microcontrollers

Personal voice recordings can optionally be added to improve detection accuracy for specific speakers.

## Related Resources

- [microWakeWord](https://github.com/OHF-Voice/micro-wake-word) — The training framework
- [ESPHome micro_wake_word docs](https://esphome.io/components/micro_wake_word/) — Configuration reference
- [Home Assistant Voice PE](https://github.com/esphome/home-assistant-voice-pe) — Official Voice PE firmware
- [ESPHome wake word models](https://github.com/esphome/micro-wake-word-models) — Official model repository

## License

This project is licensed under the [Apache License 2.0](LICENSE).
