#!/bin/bash
# Script para gravar o áudio de saída do sistema no Linux usando FFmpeg
# Requer: FFmpeg, PulseAudio/PipeWire
# Interface: Terminal com monitor de execução em tempo real

erro_fatal() {
    echo -e "\n[ERRO] $1\n"
    exit 1
}

info_msg() {
    echo -e "\n[$1] $2\n"
}

cleanup() {
    rm -f "$TEMP_LOG" "$TEMP_PID" 2>/dev/null
    # Restaura cursor visível ao sair
    tput cnorm 2>/dev/null || true
}

trap cleanup EXIT

# Verifica dependências
if ! command -v ffmpeg &> /dev/null; then
    erro_fatal "FFmpeg não está instalado.\nInstale com: sudo apt install ffmpeg"
fi

if ! command -v pactl &> /dev/null; then
    erro_fatal "pactl não encontrado. Verifique se PulseAudio ou PipeWire está instalado."
fi

echo "==============================================="
echo "Bem-vindo ao Gravador de Áudio do Sistema Linux"
echo "==============================================="
echo

# Detecta monitores
echo "Detectando fontes de áudio (monitores)..."
MONITORES=$(pactl list short sources | grep -i "monitor" | awk '{print NR " " $2}')

if [ -z "$MONITORES" ]; then
    erro_fatal "Nenhum monitor de áudio encontrado.\nExecute 'pactl list short sources' para verificar manualmente."
fi

echo -e "\nMonitores disponíveis:"
echo "$MONITORES" | while read -r num nome; do
    echo "  [$num] $nome"
done

echo
read -p "Escolha o número do monitor para gravar: " MONITOR_SELECIONADO

if ! [[ "$MONITOR_SELECIONADO" =~ ^[0-9]+$ ]]; then
    erro_fatal "Escolha inválida. Digite um número."
fi

TOTAL=$(echo "$MONITORES" | wc -l)
if [ "$MONITOR_SELECIONADO" -lt 1 ] || [ "$MONITOR_SELECIONADO" -gt "$TOTAL" ]; then
    erro_fatal "Número fora do intervalo. Escolha entre 1 e $TOTAL."
fi

MONITOR=$(echo "$MONITORES" | sed -n "${MONITOR_SELECIONADO}p" | awk '{print $2}')
echo "Monitor selecionado: $MONITOR"

echo
echo "Escolha o formato de gravação:"
echo "  1) WAV (PCM 16-bit, 44.1kHz, Stereo)"
echo "  2) WAV (PCM 24-bit, 48kHz, Stereo)"
echo "  3) MP3 (320kbps)"
echo "  4) FLAC (Lossless)"
read -p "Digite sua opção (1-4): " OPCAO

case $OPCAO in
    1)
        EXTENSAO="wav"
        CODEC_PARAMS="-acodec pcm_s16le -ar 44100 -ac 2"
        DESCRICAO="WAV 16-bit 44.1kHz"
        ;;
    2)
        EXTENSAO="wav"
        CODEC_PARAMS="-acodec pcm_s24le -ar 48000 -ac 2"
        DESCRICAO="WAV 24-bit 48kHz"
        ;;
    3)
        EXTENSAO="mp3"
        CODEC_PARAMS="-acodec libmp3lame -b:a 320k -ar 44100 -ac 2"
        DESCRICAO="MP3 320kbps"
        ;;
    4)
        EXTENSAO="flac"
        CODEC_PARAMS="-acodec flac -compression_level 8 -ar 48000 -ac 2"
        DESCRICAO="FLAC Lossless"
        ;;
    *)
        erro_fatal "Opção inválida! Escolha 1, 2, 3 ou 4."
        ;;
esac

DATA_HORA=$(date +"%Y%m%d-%H%M%S")
ARQUIVO="gravacao-${DATA_HORA}.${EXTENSAO}"
CAMINHO_COMPLETO="${PWD}/${ARQUIVO}"

echo
echo "Configurações:"
echo "  Fonte: $MONITOR"
echo "  Formato: $DESCRICAO"
echo "  Arquivo: $ARQUIVO"
echo
read -p "Iniciar gravação? (s/N): " CONFIRMA
[[ ! "$CONFIRMA" =~ ^[Ss]$ ]] && { info_msg "Cancelado" "Gravação cancelada."; exit 0; }

TEMP_LOG=$(mktemp)
TEMP_PID=$(mktemp)

# Inicia gravação em background
ffmpeg -f pulse -i "$MONITOR" $CODEC_PARAMS "$CAMINHO_COMPLETO" > "$TEMP_LOG" 2>&1 &
FFMPEG_PID=$!
echo $FFMPEG_PID > "$TEMP_PID"

sleep 2

if ! kill -0 $FFMPEG_PID 2>/dev/null; then
    ERRO=$(tail -n 20 "$TEMP_LOG")
    erro_fatal "Falha ao iniciar a gravação!\nErro do FFmpeg:\n$ERRO"
fi

# ================================
# 🔴 MONITOR DE EXECUÇÃO EM TEMPO REAL
# ================================

echo
echo "Gravação iniciada! Pressione Ctrl+C para parar."
echo "-----------------------------------------------"

# Esconde cursor para interface mais limpa
tput civis 2>/dev/null || true

INICIO=$(date +%s)

# Função para formatar segundos em HH:MM:SS
format_time() {
    local total=$1
    local h=$((total / 3600))
    local m=$(( (total % 3600) / 60 ))
    local s=$((total % 60))
    printf "%02d:%02d:%02d" $h $m $s
}

# Loop de monitoramento
while kill -0 $FFMPEG_PID 2>/dev/null; do
    AGORA=$(date +%s)
    DURACAO=$((AGORA - INICIO))
    TEMPO=$(format_time $DURACAO)

    # Tenta obter tamanho do arquivo (pode estar sendo escrito)
    if [ -f "$CAMINHO_COMPLETO" ]; then
        TAMANHO_BYTES=$(stat -c%s "$CAMINHO_COMPLETO" 2>/dev/null || echo 0)
        if [ "$TAMANHO_BYTES" -gt 0 ]; then
            # Converte para KB/MB
            if [ "$TAMANHO_BYTES" -gt 1048576 ]; then
                TAMANHO=$(awk "BEGIN {printf \"%.1f MB\", $TAMANHO_BYTES/1048576}")
            elif [ "$TAMANHO_BYTES" -gt 1024 ]; then
                TAMANHO=$(awk "BEGIN {printf \"%.1f KB\", $TAMANHO_BYTES/1024}")
            else
                TAMANHO="${TAMANHO_BYTES} B"
            fi
        else
            TAMANHO="0 B"
        fi
    else
        TAMANHO="—"
    fi

    # Atualiza a mesma linha
    printf "\r⏳ Tempo: %s | 📁 Arquivo: %s | 📏 Tamanho: %s " "$TEMPO" "$ARQUIVO" "$TAMANHO"
    
    sleep 1
done

# Restaura cursor
tput cnorm 2>/dev/null || true

# Nova linha após o loop
echo
echo
echo "Gravação finalizada."

# Aguarda um pouco para garantir que o arquivo foi fechado
sleep 2

# Verifica resultado
if [ -f "$CAMINHO_COMPLETO" ] && [ -s "$CAMINHO_COMPLETO" ]; then
    TAMANHO_FINAL=$(du -h "$CAMINHO_COMPLETO" | cut -f1)
    
    if command -v ffprobe &> /dev/null; then
        DURACAO_FINAL=$(ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 "$CAMINHO_COMPLETO" 2>/dev/null)
        if [[ -n "$DURACAO_FINAL" && "$DURACAO_FINAL" != "N/A" ]]; then
            DURACAO_HUMANA=$(awk "BEGIN {printf \"%.2f segundos\", $DURACAO_FINAL}")
        else
            DURACAO_HUMANA="Desconhecida"
        fi
        INFO_EXTRA="\nDuração: ${DURACAO_HUMANA}"
    else
        INFO_EXTRA=""
    fi

    echo "===================================="
    echo "✅ Gravação concluída com sucesso!"
    echo "===================================="
    echo "Arquivo: $ARQUIVO"
    echo "Tamanho: $TAMANHO_FINAL$INFO_EXTRA"
    echo "Local: $PWD"
    echo
    echo "Reproduzir: ffplay \"$ARQUIVO\""
else
    if [ -f "$TEMP_LOG" ]; then
        ERRO=$(tail -n 30 "$TEMP_LOG" | head -c 600)
        erro_fatal "Falha na gravação.\nLog do FFmpeg:\n$ERRO"
    else
        erro_fatal "Arquivo não foi criado ou está vazio."
    fi
fi

exit 0
