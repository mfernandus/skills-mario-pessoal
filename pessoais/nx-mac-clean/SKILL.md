---
name: nx-mac-clean
description: "Limpeza e otimização diária do Mac de Mário + triagem da caixa de e-mail. Use quando ele pedir para 'limpar a máquina', 'otimizar o Mac', 'liberar espaço', 'deixar o Mac mais rápido', 'fazer a manutenção diária', 'limpar as porcarias do computador', 'limpar os e-mails' ou 'organizar a caixa de entrada' — mesmo sem citar CleanMyMac. Requer shell local no Mac (Desktop Commander ou máquina conectada)."
---

# NX Mac Clean

Rotina diária de limpeza + otimização do Mac de Mário e triagem da caixa de e-mail, no que **de fato tem efeito e é seguro**.

## Princípio (leia antes de agir)

O macOS moderno já faz a maior parte da manutenção sozinho. Do Sequoia em diante — e o macOS atual em 2026 é o **Tahoe 26** — os antigos scripts `periodic daily weekly monthly` foram removidos (a própria pasta `/etc/periodic/` sumiu). Portanto esta skill **não** faz "ritual de limpeza": nada de forçar rebuild de Spotlight, nada de RAM cleaner, nada de apagar dados pessoais. Só mexe no que é regenerável (caches, logs, lixo temporário) e no que tem efeito real (DNS, Launch Services, installers antigos).

No Tahoe 26, boa parte do que aparece inflado como **"System Data"** em Ajustes > Geral > Armazenamento é justamente cache, log e estado temporário regenerável — exatamente o que este script mira, com segurança.

**Regra de ouro: SEMPRE roda em dry-run primeiro. Só apaga de fato depois que Mário confirmar.** Apagar arquivos é ação irreversível — nunca use `--apply` sem o "sim" dele no chat.

## Dado real x cache x nuvem (lição importante)

Com "Otimizar Armazenamento" ligado, o `du` mostra o tamanho **lógico**, que inclui arquivos **na nuvem (dataless)** que NÃO ocupam disco. A autoridade sobre espaço real é a linha **DISCO** (`df`), não o `du`. Consequências:

- Pastas grandes de apps — **WhatsApp** (`Group Containers/group.net.whatsapp.WhatsApp.shared/Message`), **Google Drive** (`~/Library/CloudStorage`), **Airmail/Mail**, **Teams**, **Wispr Flow** — são **dado real do usuário ou cópia de nuvem**, NÃO cache. Nunca apague por script: trate no próprio app (ex.: WhatsApp > Ajustes > Armazenamento; Drive > "Disponível apenas online").
- O único "grande" que costuma ocupar disco de verdade e ser regenerável é o `Claude/vm_bundles` (imagens de VM; re-baixa depois). Não apague com o app Claude em uso — pode derrubar a sessão; feche o app antes.

Por isso o script tem uma seção **GRANDES CONSUMIDORES** que apenas **lista** essas pastas (com rótulo dado/nuvem) e um **TOP-8** automático das maiores pastas de apps — tudo **somente leitura**. O `--apply` continua tocando **só** cache regenerável.

## Pré-requisito

Precisa de um **shell local no Mac de Mário** — via Desktop Commander ou com a máquina conectada a esta sessão (`device_bash`). Sem isso, avise e ofereça entregar o script (ou o instalador de um comando) para ele rodar manualmente no Terminal.

## Fluxo

### 1. Limpeza + otimização do Mac

Grave o script (seção **Script**) em `~/bin/nx_mac_clean.sh`, dê permissão e rode **primeiro em modo simulação**:

```bash
chmod +x ~/bin/nx_mac_clean.sh
~/bin/nx_mac_clean.sh            # dry-run: diagnostica e mostra o que faria
```

Apresente a Mário: a linha DISCO (espaço real), os GRANDES CONSUMIDORES (o que é dado/nuvem, tratado no app) e o recuperável de cache por categoria. **Espere ele confirmar.** Só então:

```bash
~/bin/nx_mac_clean.sh --apply           # limpa o cache do dia
~/bin/nx_mac_clean.sh --apply --deep     # inclui caches de dev (Xcode, npm, brew, pip, yarn)
```

`--apply` sem `--deep` é a rotina **diária** recomendada. O `--deep` vale semanal ou quando o disco apertar.

### 2. Limpeza / organização da caixa de e-mail

Via conector **Gmail** (não pelo script). Cheque as ferramentas com `tool_search` e trabalhe assim:

- Levante candidatos a triagem: promoções/newsletters antigas, notificações automáticas, threads resolvidas, malas-diretas (`older_than:`, remetentes recorrentes, `category:promotions`).
- **Nunca apague em definitivo.** Mover para lixeira ou aplicar rótulo/arquivar é o teto — e só depois de mostrar a lista e ter o "sim". Esvaziar lixeira é proibido.
- Ofereça criar rótulos/filtros para o recorrente, mas filtro é config persistente: descreva e confirme antes.
- Entregue um resumo por balde e execute só o aprovado.

(Obs.: se o cliente de e-mail no Mac for **Airmail** e não o Mail nativo, o cache dele aparece no TOP-8 como dado local — trate no próprio Airmail, não por script.)

### 3. Relatório final

Feche curto, direto (estilo Mário):
- espaço livre real (linha DISCO) e recuperado de cache (medido, não estimado);
- grandes consumidores e o que é dado/nuvem (o que exige a mão dele no app);
- caixa de e-mail: quantos triados e como;
- recomendações que exijam ação dele.

## Automação diária (launchd)

Agende apenas o **relatório dry-run** (`--report`), e deixe o `--apply` sempre com confirmação humana. `.plist` roda `~/bin/nx_mac_clean.sh --report` às 8h e grava log em `/tmp/`. Ative com:

```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/net.next.macclean.plist \
  || launchctl load ~/Library/LaunchAgents/net.next.macclean.plist
```

Desativar: `launchctl bootout gui/$(id -u)/net.next.macclean`. Há também um **instalador de um comando** (seção abaixo) que grava script + `.plist` (com o usuário via `$HOME`/`$(id -u)`, sem editar `SEU_USUARIO`) e ativa.

## Instalador de um comando

Quando a skill não toca a máquina direto, entregue a Mário um instalador que ele roda **uma vez** no Terminal: grava o script em `~/bin`, cria o launchd (dry-run às 8h) e ativa. Uso: `bash ~/Downloads/nx_mac_clean_install.sh` para instalar; `--uninstall` para remover o agendamento. O `.plist` usa `Hour 8 / Minute 0` e chama `--report`. O `--apply` nunca é automatizado.

## Segurança

- Dry-run por padrão; `--apply` só com confirmação explícita no chat.
- `--apply` toca **só** cache/logs/lixo regenerável. A seção GRANDES CONSUMIDORES é **somente leitura**.
- Dado real / nuvem (WhatsApp, Drive, Airmail, Teams, vm_bundles) nunca é apagado por script — trate no app.
- Todo run gera log em `$TMPDIR/nx_mac_clean_<timestamp>.log`.
- Se algo parecer arriscado (apagar Documentos, mexer em `/System`, esvaziar lixeira de e-mail), **recuse e explique**.

---

## O que o script APAGA no --apply (e por que é seguro)

Cada item é **regenerável**: o macOS ou o app recria sozinho. Nada aqui toca documentos, fotos, e-mails ou configurações.

### Rotina diária (`--apply`)

| Categoria | Caminho | Por que é seguro |
|---|---|---|
| Cache de usuário | `~/Library/Caches` | Apps recriam sob demanda |
| Logs de usuário | `~/Library/Logs` | Histórico de log, não dado |
| Relatórios de diagnóstico | `~/Library/Logs/DiagnosticReports` | Crash reports antigos |
| Saved App State (>7d) | `~/Library/Saved Application State` | Só "reabrir janelas onde estavam" |
| Installers antigos (>30d) | `~/Downloads/*.dmg`, `*.pkg` | Já instalados |
| Lixeira | `~/.Trash` | Já era lixo |
| Adobe Media Cache | `~/Library/Application Support/Adobe/Common/Media Cache*` | Premiere/After Effects recriam |

### Modo dev (`--apply --deep`)

Xcode DerivedData, Xcode iOS DeviceSupport, cache npm, `brew cleanup -s`, cache pip, cache Yarn.

### Manutenção (só no `--apply`, pede sudo)

Flush de DNS (`dscacheutil -flushcache` + `killall -HUP mDNSResponder`) e rebuild do Launch Services (`lsregister -kill -r`). Rodando sem interatividade (launchd/Desktop Commander), o sudo é pulado com segurança.

### O que NÃO faz

Não força Spotlight, não roda RAM cleaner/`purge`, não toca `/System` nem dados de usuário, não esvazia lixeira de e-mail, e **não apaga as pastas grandes de apps** (só as lista).

### Comparação honesta com o CleanMyMac

Cobre a faxina de espaço (cache, logs, lixo Adobe/dev, installers) igual ou melhor, transparente e sem assinatura, e ainda **diagnostica os grandes consumidores** e organiza e-mail. Não faz: varredura de malware, remoção de apps com todos os rastros, detecção de duplicatas. Para essas três, vale manter o CleanMyMac ao lado.

---

## Script

Grave exatamente este conteúdo em `~/bin/nx_mac_clean.sh` (medição blindada contra `pipefail`/`du` e com diagnóstico dos grandes):

```bash
#!/usr/bin/env bash
#
# nx_mac_clean.sh — Limpeza e otimização diária do Mac (padrão Next)
#
# SEMPRE roda em modo DRY-RUN (só mostra o que faria). Para apagar de fato,
# passe --apply. Tudo é registrado num log.
#
# Uso:
#   ./nx_mac_clean.sh                 # simula (dry-run), nada é apagado
#   ./nx_mac_clean.sh --apply         # executa a limpeza de fato
#   ./nx_mac_clean.sh --apply --deep  # inclui categorias de desenvolvedor
#   ./nx_mac_clean.sh --report        # só o diagnóstico, sem limpar
#
set -uo pipefail

APPLY=0
DEEP=0
REPORT_ONLY=0
for arg in "$@"; do
  case "$arg" in
    --apply)  APPLY=1 ;;
    --deep)   DEEP=1 ;;
    --report) REPORT_ONLY=1 ;;
    -h|--help)
      grep '^#' "$0" | sed 's/^# \{0,1\}//' | head -30
      exit 0 ;;
    *) echo "Argumento desconhecido: $arg"; exit 1 ;;
  esac
done

TS="$(date +%Y%m%d_%H%M%S)"
LOG="${TMPDIR:-/tmp}/nx_mac_clean_${TS}.log"
HUMAN="$HOME"

log(){ echo "$1" | tee -a "$LOG"; }
hr(){ log "------------------------------------------------------------"; }

size_of(){
  local kb
  [ -e "$1" ] || { echo 0; return; }
  kb=$(du -sk "$1" 2>/dev/null | awk 'END{print $1+0}')
  echo "${kb:-0}"
}
human_kb(){ awk -v k="$1" 'BEGIN{ split("KB MB GB TB",u); i=1; while(k>=1024 && i<4){k/=1024;i++} printf "%.1f %s", k, u[i] }'; }

disk_report(){
  hr
  log "DISCO"
  df -H / | tail -1 | awk '{print "  Total: "$2"  |  Usado: "$3"  |  Livre: "$4"  |  "$5" ocupado"}' | tee -a "$LOG"
  local purge
  purge=$(diskutil info / 2>/dev/null | awk -F': *' '/Purgeable/{print $2}' | head -1)
  [ -n "${purge:-}" ] && log "  Purgeable (o macOS libera sob demanda): $purge"
}

build_targets(){
  TARGETS=()
  TARGETS+=("Cache de usuário|$HUMAN/Library/Caches|0")
  TARGETS+=("Logs de usuário|$HUMAN/Library/Logs|0")
  TARGETS+=("Relatórios de diagnóstico|$HUMAN/Library/Logs/DiagnosticReports|0")
  TARGETS+=("Cache de containers salvos (Saved App State)|$HUMAN/Library/Saved Application State|7")
  TARGETS+=("Downloads .dmg/.pkg antigos (>30d)|__INSTALLERS__|30")
  TARGETS+=("Lixeira|$HUMAN/.Trash|0")
  TARGETS+=("Adobe Media Cache|$HUMAN/Library/Application Support/Adobe/Common/Media Cache Files|0")
  TARGETS+=("Adobe Media Cache (DB)|$HUMAN/Library/Application Support/Adobe/Common/Media Cache|0")

  if [ "$DEEP" -eq 1 ]; then
    TARGETS+=("Xcode DerivedData|$HUMAN/Library/Developer/Xcode/DerivedData|0")
    TARGETS+=("Xcode iOS DeviceSupport antigo|$HUMAN/Library/Developer/Xcode/iOS DeviceSupport|0")
    TARGETS+=("Cache npm|$HUMAN/.npm/_cacache|0")
    TARGETS+=("Cache Homebrew|__BREW__|0")
    TARGETS+=("Cache pip|$HUMAN/Library/Caches/pip|0")
    TARGETS+=("Cache Yarn|$HUMAN/Library/Caches/Yarn|0")
  fi
}

clean_target(){
  local desc="$1" path="$2" mindays="$3"
  local reclaimed=0

  if [ "$path" = "__INSTALLERS__" ]; then
    local files
    files=$(find "$HUMAN/Downloads" -maxdepth 1 \( -name '*.dmg' -o -name '*.pkg' \) -mtime +"$mindays" 2>/dev/null)
    [ -z "$files" ] && { log "  · $desc — nada a limpar"; return; }
    while IFS= read -r f; do
      [ -z "$f" ] && continue
      reclaimed=$((reclaimed + $(size_of "$f")))
      if [ "$APPLY" -eq 1 ]; then rm -f "$f"; fi
    done <<< "$files"
    log "  · $desc — $(human_kb $reclaimed)$([ $APPLY -eq 0 ] && echo ' (simulado)')"
    RECLAIMED_TOTAL=$((RECLAIMED_TOTAL + reclaimed))
    return
  fi

  if [ "$path" = "__BREW__" ]; then
    if command -v brew >/dev/null 2>&1; then
      if [ "$APPLY" -eq 1 ]; then
        brew cleanup -s >>"$LOG" 2>&1
        log "  · $desc — limpo (brew cleanup -s)"
      else
        local out; out=$(brew cleanup -n 2>/dev/null | tail -1)
        log "  · $desc — ${out:-simulação} (simulado)"
      fi
    else
      log "  · $desc — Homebrew não instalado, pulando"
    fi
    return
  fi

  [ ! -d "$path" ] && { log "  · $desc — pasta não existe, pulando"; return; }

  if [ "$mindays" -gt 0 ]; then
    while IFS= read -r item; do
      [ -z "$item" ] && continue
      reclaimed=$((reclaimed + $(size_of "$item")))
      if [ "$APPLY" -eq 1 ]; then rm -rf "$item" 2>/dev/null; fi
    done < <(find "$path" -mindepth 1 -maxdepth 1 -mtime +"$mindays" 2>/dev/null)
  else
    reclaimed=$(size_of "$path")
    if [ "$APPLY" -eq 1 ]; then
      find "$path" -mindepth 1 -maxdepth 1 -exec rm -rf {} + 2>/dev/null
    fi
  fi

  log "  · $desc — $(human_kb $reclaimed)$([ $APPLY -eq 0 ] && echo ' (simulado)')"
  RECLAIMED_TOTAL=$((RECLAIMED_TOTAL + reclaimed))
}

run_maintenance(){
  [ "$APPLY" -eq 0 ] && { log "  (manutenção pulada em modo simulação)"; return; }
  sudo dscacheutil -flushcache 2>/dev/null
  sudo killall -HUP mDNSResponder 2>/dev/null && log "  · Cache de DNS limpo"
  /System/Library/Frameworks/CoreServices.framework/Frameworks/LaunchServices.framework/Support/lsregister \
    -kill -r -domain local -domain system -domain user 2>/dev/null && log "  · Menu 'Abrir com' reconstruído"
}

# ---------- diagnóstico dos grandes (somente leitura) ----------
big_consumers_report(){
  local TO=""
  command -v gtimeout >/dev/null 2>&1 && TO="gtimeout 90"
  hr
  log "GRANDES CONSUMIDORES (somente leitura — NADA aqui é apagado)"
  log "  (tamanhos lógicos; o espaço real livre está na linha DISCO acima)"
  local BIGS=(
    "Claude — vm_bundles (imagens de VM; regenerável, re-baixa)|$HUMAN/Library/Application Support/Claude/vm_bundles"
    "Claude — sessões de agente|$HUMAN/Library/Application Support/Claude/local-agent-mode-sessions"
    "WhatsApp — mídia + conversas (DADO REAL, tratar no app)|$HUMAN/Library/Group Containers/group.net.whatsapp.WhatsApp.shared"
    "Google Drive — sincronizados (NUVEM/DADO, tratar no app)|$HUMAN/Library/CloudStorage"
    "Google Drive — cache DriveFS|$HUMAN/Library/Application Support/Google/DriveFS"
    "Mail — app nativo|$HUMAN/Library/Mail"
  )
  local b d p kb
  for b in "${BIGS[@]}"; do
    IFS='|' read -r d p <<< "$b"
    [ -e "$p" ] || continue
    kb=$($TO du -sk "$p" 2>/dev/null | awk 'END{print $1+0}')
    log "  · $d — $(human_kb ${kb:-0})"
  done
  hr
  log "TOP-8 pastas de apps (Application Support / Group Containers / Containers)"
  $TO du -sk "$HUMAN/Library/Application Support"/* "$HUMAN/Library/Group Containers"/* "$HUMAN/Library/Containers"/* 2>/dev/null \
    | sort -rn | head -8 | while IFS=$'\t' read -r k rest; do
        log "  · $(human_kb ${k:-0}) — ${rest#$HUMAN/Library/}"
      done
}

log "NX MAC CLEAN — $(date '+%d/%m/%Y %H:%M')"
log "Modo: $([ $APPLY -eq 1 ] && echo 'APPLY (apaga de fato)' || echo 'DRY-RUN (só simula)')$([ $DEEP -eq 1 ] && echo ' + DEEP (dev)')"
log "Log: $LOG"

disk_report
big_consumers_report
FREE_BEFORE=$(df -k / | tail -1 | awk '{print $4}')

if [ "$REPORT_ONLY" -eq 1 ]; then
  hr; log "Somente relatório solicitado. Nada foi alterado."
  echo "$LOG"; exit 0
fi

hr
log "LIMPEZA"
RECLAIMED_TOTAL=0
build_targets
for t in "${TARGETS[@]}"; do
  IFS='|' read -r d p m <<< "$t"
  clean_target "$d" "$p" "$m"
done

hr
log "MANUTENÇÃO"
run_maintenance

hr
log "RESUMO"
log "  Recuperável identificado (cache): $(human_kb $RECLAIMED_TOTAL)"
if [ "$APPLY" -eq 1 ]; then
  sleep 1
  FREE_AFTER=$(df -k / | tail -1 | awk '{print $4}')
  DELTA=$((FREE_AFTER - FREE_BEFORE))
  [ "$DELTA" -lt 0 ] && DELTA=0
  log "  Espaço livre ganho (medido): $(human_kb $DELTA)"
else
  log "  (nada foi apagado — rode de novo com --apply para efetivar)"
fi
hr
echo "$LOG"
```