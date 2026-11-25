# Podcast diário de notícias com IA (custo zero)

Todo dia de manhã, um episódio de ~30 minutos para cada pessoa, com os temas dela.
Roda sozinho na nuvem (GitHub Actions), não precisa de PC ligado.

```
feeds RSS -> texto das matérias -> Gemini escreve o roteiro -> edge-tts gera a voz
          -> MP3 + feed.xml no GitHub Pages -> app de podcast baixa sozinho
```

| Peça | O que usa | Custo |
|---|---|---|
| Agendamento e execução | GitHub Actions (repositório público) | R$ 0 |
| Roteiro | Gemini API, plano gratuito | R$ 0 |
| Voz | edge-tts (vozes pt-BR da Microsoft) | R$ 0 |
| Hospedagem | GitHub Pages | R$ 0 |
| Ouvir | Apple Podcasts (iPhone) / AntennaPod (Android) | R$ 0 |

## Passo a passo (uma vez só)

1. **Crie um repositório público no GitHub** e envie todos os arquivos desta pasta.
   (Pages em repositório privado exige plano pago, por isso público.)
2. **Pegue uma chave do Gemini**: https://aistudio.google.com/apikey (grátis).
3. **Guarde a chave como segredo**: no repositório, *Settings > Secrets and variables > Actions >
   New repository secret*. Nome: `GEMINI_API_KEY`. Valor: a chave.
4. **O `slug`** de `config/leo.yaml` e `config/lu.yaml` já é um código aleatório. Ele vira parte
   do endereço do feed, que fica "não listado"; só troque se quiser outro.
   Ajuste os temas da Lu em `config/lu.yaml`.
5. **Rode a primeira vez na mão**: aba *Actions > Podcast diário > Run workflow*.
   Leva uns 10 a 15 minutos. Ao final, o branch `gh-pages` é criado.
6. **Ative o Pages**: *Settings > Pages > Build and deployment > Deploy from a branch >
   `gh-pages` / `(root)` > Save*. Em 1 a 2 minutos o site está no ar.
7. **Endereço do feed de cada um**:
   `https://SEU-USUARIO.github.io/NOME-DO-REPOSITORIO/SLUG/feed.xml`

Depois disso o episódio é gerado todo dia às 04:30 (Brasília), e fica pronto antes de você acordar.

## Ouvir no celular

**Android (Leo)** - AntennaPod (grátis, sem anúncios): toque em **+** > *Adicionar podcast por
endereço RSS* e cole o link do feed. Nas configurações do podcast, ative o download automático.
Funciona no Android Auto.

**iPhone (Lu)** - Apple Podcasts: aba *Biblioteca* > menu **...** > *Seguir um programa por
URL* (o texto exato pode variar conforme a versão do iOS) e cole o link. Nas configurações do
programa, deixe o download automático de novos episódios ligado. Funciona no CarPlay.

## Personalizar

Tudo está em `config/<pessoa>.yaml`. Não precisa mexer no código.

- **Adicionar um tema**: copie um bloco de `temas:` e troque `nome`, `instrucoes` e `fontes`.
- **Mais ou menos tempo para um tema**: mude o `peso` (é proporcional).
- **Filtrar assunto**: `palavras_chave` (sobem na fila), `somente_com_palavra_chave: true`
  (descarta o resto) e `excluir` (ignora matérias com esses termos).
- **Voz e velocidade**: `voz` e `velocidade`. Veja as vozes com `edge-tts --list-voices`.
- **Adicionar uma pessoa**: crie `config/nome.yaml` (copie um existente). O workflow pega sozinho.

Para saber se uma fonte RSS funciona: `python gerar_podcast.py --perfil leo --checar-feeds`.
Fonte que falha é apenas ignorada no dia; troque ou remova depois.

## Testar no computador

```bash
pip install -r requirements.txt      # e tenha o ffmpeg instalado
```

Linux/macOS:
```bash
export GEMINI_API_KEY="sua-chave"
```
Windows (PowerShell):
```powershell
$env:GEMINI_API_KEY="sua-chave"
```

```bash
python gerar_podcast.py --perfil leo --checar-feeds             # testa as fontes
python gerar_podcast.py --perfil leo --so-roteiro               # só texto (salvo em rascunhos/)
python gerar_podcast.py --perfil leo --saida site_teste         # episódio completo
python gerar_podcast.py --perfil leo --saida site_teste --refazer   # refaz o de hoje
```

Se o episódio de hoje já existe, uma nova execução não faz nada (para não sobrescrever com as
sobras de notícias). Use `--refazer` para substituí-lo, por exemplo depois de mudar os temas.

## Problemas comuns

- **Gemini retorna 404 / modelo não encontrado**: o Google troca e aposenta modelos. Defina a
  variável `GEMINI_MODEL` (no workflow, em `env:`) com um modelo Flash que tenha plano gratuito,
  conforme https://ai.google.dev/gemini-api/docs/pricing. O padrão é `gemini-2.5-flash`.
- **Gemini retorna 429**: limite do plano gratuito. O script espera e tenta de novo; se persistir,
  aumente `pausa_entre_chamadas_s` no perfil ou reduza o número de temas.
- **edge-tts falha só no GitHub**: o serviço é não oficial e às vezes bloqueia IPs de nuvem.
  Rode de novo mais tarde; se virar rotina, a alternativa gratuita é trocar a voz por um TTS
  local (Piper), o que exige alterar a função `sintetizar` em `gerar_podcast.py`.
- **Episódio mais curto que 30 min**: faltaram notícias novas nas fontes, ou o Gemini escreveu
  menos que o pedido. Aumente `horas_retroativas`, adicione fontes ou ajuste
  `palavras_por_minuto` (o log mostra a estimativa de minutos).
- **O agendamento parou**: o GitHub desativa workflows agendados em repositórios sem atividade
  por 60 dias. Ele avisa por e-mail; basta reativar em *Actions* ou fazer qualquer commit
  (editar um tema já conta).
- **Quando a geração falha**, o job fica vermelho e o GitHub manda e-mail. Os episódios já
  publicados continuam no ar.

## Observações

- O repositório é público, então os temas em `config/` também são. A chave do Gemini fica
  protegida em *Secrets* e não aparece no código nem nos logs.
- O endereço do feed é difícil de adivinhar, mas não é secreto de verdade. Não compartilhe e
  não envie a diretórios de podcast: o conteúdo é um resumo pessoal de matérias de terceiros.
- Cada episódio fica por volta de 10 a 12 MB (voz em 48 kbps). Com 7 episódios por pessoa
  o site ocupa ~150 MB. A cada execução o branch `gh-pages` é recriado do zero, então o
  histórico do Git não cresce.
- Notícias já usadas em episódios anteriores não se repetem (o controle fica em `estado.json`).
