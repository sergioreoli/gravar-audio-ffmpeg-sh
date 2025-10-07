# 🎙️ Gravador de Áudio do Sistema Linux

> **Grave o áudio de saída do seu sistema (o que você ouve nos alto-falantes) diretamente pelo terminal — sem interface gráfica, sem complicações.**

Este script em Bash usa `ffmpeg` e o sistema de áudio do Linux (PulseAudio ou PipeWire) para capturar o **monitor de áudio** (loopback) e salvar em formatos de alta qualidade como WAV, MP3 ou FLAC — tudo via linha de comando.

Ideal para:  
- Criar tutoriais com áudio do sistema  
- Arquivar streams de áudio  
- Gravar gameplay com som interno  
- Usar em servidores ou ambientes sem interface gráfica (SSH)

---

## ✨ Recursos

- ✅ Detecta automaticamente fontes de áudio do tipo *monitor*  
- ✅ Suporte a múltiplos formatos: WAV (16/24-bit), MP3 (320kbps), FLAC (lossless)  
- ✅ Nome automático com data e hora (`gravacao-20250405-143022.wav`)  
- ✅ **Monitor de execução em tempo real** no terminal (tempo decorrido + tamanho do arquivo)  
- ✅ Parada suave com `Ctrl+C` (finaliza corretamente o arquivo)  
- ✅ Funciona em qualquer terminal (inclusive via SSH)  
- ✅ Sem dependência de `whiptail`, `zenity` ou GUI

---

## ⚙️ Pré-requisitos

Seu sistema deve ter:

- **Linux** (com PulseAudio **ou** PipeWire)
- **`ffmpeg`** (com suporte a `pulse` e codecs: `pcm`, `libmp3lame`, `flac`)
- **`pactl`** (faz parte do `pulseaudio-utils` ou `pipewire-pulse`)

> 💡 A maioria das distribuições modernas (Ubuntu, Fedora, Debian, Arch, etc.) já incluem esses componentes.

### Instalação das dependências (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install ffmpeg pulseaudio-utils
