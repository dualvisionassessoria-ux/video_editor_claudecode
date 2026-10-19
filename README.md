# Editor de vídeo profissional no Claude Code

Skills para o [Claude Code](https://claude.com/claude-code) editar vídeos como um editor profissional: **corta pausas e erros de fala, tira o ruído da voz, põe música de fundo baixinha, cria legendas palavra a palavra e motion graphics** (cards, HUD, números animados, CTA). Pensado para Reels, Shorts e TikTok (vertical 1080x1920), mas serve para YouTube.

Você grava, pede "edita esse vídeo" e o Claude segue um fluxo fixo de qualidade.

## O que ele faz em todo vídeo

1. **Sem ruído:** DeepFilterNet 3 em todas as falas.
2. **Ritmo rápido e enxuto:** corta pausas acima de 0,24 s e acelera 1,1x (sem mudar o tom).
3. **Sem erros de fala:** remove repetições ("tá, tá"), engasgos e enchimentos (ahn, é...), cortando no vale do áudio.
4. **Música de fundo sempre**, ~19 dB abaixo da voz, com ducking, para nunca competir com a fala.
5. **Legendas dinâmicas** palavra a palavra com destaque na cor da sua marca.
6. **Motion graphics** em Remotion (React): cards, HUD, títulos, CTA final.
7. **Áudio em -14 LUFS** (padrão YouTube/Instagram).
8. **Revisões como "versão 2", "versão 3"**, sem sobrescrever.

## Estrutura

```
.claude/skills/
  edicao-video/              # orquestrador: regras fixas + fluxo + guia de estilo
  decupagem-video/           # transcrição (faster-whisper) + cortes automáticos
  limpeza-audio-musica/      # DeepFilterNet, música com ducking, SFX, loudness
  legendas-dinamicas/        # legendas palavra a palavra (ffmpeg/ASS)
  motion-graphics-remotion/  # template Remotion (cards, HUD, CTA, legendas)
config/marca.exemplo.json    # sua identidade: cor, fontes, CTA
docs/licoes-aprendidas.md    # armadilhas técnicas e soluções
CLAUDE.md                    # instrui o Claude Code a usar as skills
```

## Instalação

Pré-requisitos: [Claude Code](https://claude.com/claude-code), Python 3.10+, Node 20+ (só para motion graphics) e ffmpeg (`apt install ffmpeg`, `brew install ffmpeg` ou `winget install ffmpeg`).

```bash
git clone https://github.com/dualvisionassessoria-ux/video_editor_claudecode
cd editor-de-video-claude-code
bash .claude/skills/edicao-video/scripts/preparar_ambiente.sh      # faster-whisper + modelo (~1,6 GB)
bash .claude/skills/limpeza-audio-musica/scripts/instalar_deepfilternet.sh   # limpeza de ruído
cp config/marca.exemplo.json config/marca.json                     # edite com sua marca
```

Para usar em qualquer pasta, copie `.claude/skills/*` para `~/.claude/skills/` (skills pessoais) ou para o `.claude/skills/` do seu projeto de vídeos. No Windows use WSL ou Git Bash para os `.sh`.

Música: o repositório não traz faixas. Baixe músicas sem direitos autorais (por exemplo em [freepd.com](https://freepd.com), domínio público) e deixe numa pasta `musicas/`.

## Como usar

Abra o Claude Code na pasta do repositório (ou do seu projeto de vídeos) e peça em português:

> Edite o vídeo `videos/depoimento.mp4`: vertical, com legenda e música de fundo. CTA: "COMENTA QUERO".

O Claude vai: ver o vídeo, limpar o áudio, transcrever em modo literal, cortar pausas/erros, aplicar legendas e motion, mixar a música e entregar `depoimento-v1.mp4`. Para ajustes, peça "versão 2: tira o card do segundo 5 e deixa a música ainda mais baixa".

### Rodando as etapas na mão

```bash
S=.claude/skills
python3 $S/limpeza-audio-musica/scripts/limpar_ruido.py video.mp4 trabalho/voz_limpa.wav
python3 $S/decupagem-video/scripts/transcrever.py video.mp4 --literal -o trabalho/tr
python3 $S/decupagem-video/scripts/decupar.py video.mp4 trabalho/tr/video.json -o trabalho/dec \
        --audio trabalho/voz_limpa.wav --vertical --renderizar
python3 $S/legendas-dinamicas/scripts/gerar_legendas_ass.py trabalho/dec/edl.json trabalho/dec/leg.ass --marca config/marca.json
ffmpeg -i trabalho/dec/base.mp4 -vf "ass=trabalho/dec/leg.ass:fontsdir=fontes" -c:a copy trabalho/dec/legendado.mp4
ffmpeg -i trabalho/dec/base.mp4 -vn -ac 1 -ar 48000 trabalho/dec/voz_final.wav
python3 $S/limpeza-audio-musica/scripts/mixar_audio.py trabalho/dec/voz_final.wav musicas/faixa.mp3 trabalho/mix.wav
ffmpeg -i trabalho/dec/legendado.mp4 -i trabalho/mix.wav -map 0:v -map 1:a -c:v copy -c:a aac -b:a 256k video-v1.mp4
```

## O que foi testado

Os scripts de decupagem, legendas ASS, SFX e mixagem foram executados de ponta a ponta em um vídeo sintético (cortes, remoção de repetição/enchimento, vertical 1080x1920, mixagem). O template Remotion passa na checagem de tipos (`npm run tipos`). O `instalar_deepfilternet.sh` e o `limpar_ruido.py` dependem de torch e do modelo baixado da internet e não foram reexecutados ao montar este pacote: se algo falhar, veja `docs/licoes-aprendidas.md`.

## Créditos e licença

O guia de estilo foi destilado da análise de vídeos de referência de criadores de conteúdo sobre edição com IA. Ferramentas: [faster-whisper](https://github.com/SYSTRAN/faster-whisper), [DeepFilterNet](https://github.com/Rikorose/DeepFilterNet), [Remotion](https://www.remotion.dev), [ffmpeg](https://ffmpeg.org). Licença MIT (veja `LICENSE`). Remotion tem licença própria: confira se o seu uso exige licença comercial.
