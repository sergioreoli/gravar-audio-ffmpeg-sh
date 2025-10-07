
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

bash


🚀 Instalação
Baixe o script:
```bash
wget https://raw.githubusercontent.com/seu-usuario/gravador-audio-linux/main/gravar-audio-ffmpeg.sh
```

Torne-o executável:

```bash
1
chmod +x gravar-audio-ffmpeg.sh
```

🔁 Substitua seu-usuario pelo seu nome de usuário no GitHub. 

▶️ Uso
Execute o script:

```bash
./gravar-audio-ffmpeg.sh
```

Passo a passo:
O script lista todos os monitores de áudio disponíveis (ex: alsa_output.pci-0000_00_1b.0.analog-stereo.monitor)
Escolha o número correspondente à saída que deseja gravar
Selecione o formato desejado:
1: WAV 16-bit, 44.1kHz (CD Quality)
2: WAV 24-bit, 48kHz (Estúdio)
3: MP3 320kbps (Compacto e de alta qualidade)
4: FLAC (Sem perdas, compactado)
Confirme a gravação
Gravação inicia automaticamente com monitor em tempo real:


1
⏳ Tempo: 00:02:15 | 📁 Arquivo: gravacao-20250405-143022.wav | 📏 Tamanho: 24.8 MB
Pressione Ctrl+C a qualquer momento para parar

Após a gravação, o arquivo será salvo no diretório atual com informações detalhadas.

🛑 Parando a gravação
Durante a gravação, pressione Ctrl+C no terminal.
O script envia um sinal de interrupção suave (SIGINT) ao ffmpeg, garantindo que o arquivo seja finalizado corretamente (não corrompido).
Evite fechar o terminal bruscamente — isso pode deixar o arquivo inutilizável.

🔍 Solução de Problemas
❌ "Nenhum monitor de áudio encontrado"
Verifique se o áudio está funcionando normalmente.
Execute manualmente:

```bash
pactl list short sources
```

Procure por entradas com .monitor no nome. Se não houver, seu sistema pode não estar configurado para loopback.
Dica: Em alguns ambientes (como WSL ou containers), o monitor de áudio não está disponível. 

❌ "Falha ao iniciar a gravação"
Verifique se outro programa está usando exclusivamente o dispositivo de áudio.
Certifique-se de que o ffmpeg foi compilado com suporte a pulse:

```bash
ffmpeg -formats 2>/dev/null | grep -i pulse
```

❌ Arquivo vazio ou corrompido
Isso geralmente acontece se o processo for morto com SIGKILL (ex: kill -9).
Sempre use Ctrl+C para parar — o script trata isso corretamente.

📄 Licença
Este projeto está licenciado sob a MIT License.

```bash

MIT License

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

```

🙌 Contribuições
Sugestões, correções e melhorias são bem-vindas! Abra uma issue ou envie um pull request.

📦 Autor
Criado por Sergio ReOli

🎧 Dica profissional: Combine com parec ou ffmpeg para gravar áudio do microfone + sistema simultaneamente (mixagem). Consulte a documentação do PulseAudio para criar fontes virtuais! 
sudo apt update
sudo apt install ffmpeg pulseaudio-utils
