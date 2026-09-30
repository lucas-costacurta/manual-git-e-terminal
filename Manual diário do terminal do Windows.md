# Manual diário do terminal do Windows

Guia prático para quem já usa Git Bash e de vez em quando precisa do terminal do Windows.

## 1. Qual terminal usar

| Terminal | Quando usar |
| --- | --- |
| **Git Bash** | Git, comandos estilo Linux (`ls`, `grep`, `cat`). Seu padrão do dia a dia. |
| **PowerShell** | Tarefas do Windows: processos, rede, variáveis de ambiente, serviços, permissões. |
| **CMD** | Só para scripts antigos (`.bat`). Evite em uso novo. |

**Regra prática:** fique no Git Bash para código e Git. Abra o PowerShell quando o comando for algo do *sistema* Windows. Comandos como `ipconfig`, `tasklist`, `ping` e `python` funcionam nos dois.

**Dica:** o **Windows Terminal** (loja da Microsoft) junta Git Bash, PowerShell e CMD em abas no mesmo app.

---

## 2. Navegação e arquivos

| O que fazer | Git Bash | PowerShell |
| --- | --- | --- |
| Onde estou | `pwd` | `pwd` |
| Listar | `ls -la` | `ls` (ou `dir`) |
| Entrar na pasta | `cd pasta` | `cd pasta` |
| Voltar um nível | `cd ..` | `cd ..` |
| Ir para a home | `cd ~` | `cd ~` |
| Criar pasta | `mkdir nome` | `mkdir nome` |
| Criar arquivo vazio | `touch arq.txt` | `ni arq.txt` |
| Copiar | `cp a.txt b.txt` | `cp a.txt b.txt` |
| Mover/renomear | `mv a.txt b.txt` | `mv a.txt b.txt` |
| Apagar arquivo | `rm a.txt` | `rm a.txt` |
| Apagar pasta | `rm -rf pasta` | `rm -r -fo pasta` |
| Ver conteúdo | `cat arq.txt` | `cat arq.txt` |
| Ver só o início/fim | `head -20` / `tail -20` | `gc arq -Head 20` / `gc arq -Tail 20` |
| Abrir no Explorer | `explorer .` | `ii .` |

**Caminhos com espaço:** use aspas: `cd "C:\Users\Lucas\Meus Documentos"`.

**Caminhos no Git Bash:** o disco `C:` vira `/c/`. Exemplo: `cd /c/Users/lucas`. No PowerShell é `C:\Users\lucas`.

**Cuidado:** `rm` não manda para a lixeira. Apagou, acabou. Antes de `rm -rf`, rode `ls` na pasta para conferir onde está.

---

## 3. Buscar coisas

| O que fazer | Git Bash | PowerShell |
| --- | --- | --- |
| Texto dentro de arquivos | `grep -rn "texto" .` | `sls "texto" -r` (`Select-String`) |
| Achar arquivo por nome | `find . -name "*.pbip"` | `gci -r -filter *.pbip` |
| Contar linhas | `wc -l arq.csv` | `(gc arq.csv).Count` |
| Filtrar saída de outro comando | `comando \| grep x` | `comando \| sls x` |

Exemplo prático: achar em quais arquivos TMDL uma medida é usada:

```bash
grep -rn "Total Alunos" --include="*.tmdl" .
```

---

## 4. Processos e sistema

```powershell
tasklist                          # lista processos
tasklist | findstr /i excel       # filtra por nome
taskkill /im EXCEL.EXE /f         # fecha o processo à força
taskkill /pid 1234 /f             # fecha por PID
systeminfo                        # versão do Windows, memória, etc.
```

Se o Power BI Desktop travar, `taskkill /im PBIDesktop.exe /f` resolve (você perde o que não foi salvo).

**Executar como administrador:** clique com o botão direito no terminal → *Executar como administrador*. Só faça isso quando o comando exigir.

---

## 5. Rede

```powershell
ipconfig                   # seu IP
ipconfig /flushdns         # limpa cache de DNS (resolve "site não abre" após mudança)
ping servidor.com          # testa se responde
nslookup servidor.com      # resolve o nome em IP
tracert servidor.com       # caminho até o destino
netstat -ano | findstr :8501   # quem está usando a porta 8501
curl -I https://site.com   # só os cabeçalhos HTTP
```

**Porta em uso** (comum com Streamlit/Jupyter): ache o PID com `netstat -ano | findstr :PORTA` e encerre com `taskkill /pid NUMERO /f`.

---

## 6. Variáveis de ambiente e PATH

```powershell
echo $env:PATH                        # PowerShell: ver o PATH
$env:MINHA_VAR = "valor"              # define só para esta sessão
setx MINHA_VAR "valor"                # define de forma permanente (vale em novos terminais)
where python                          # onde o executável está (CMD/PowerShell)
```

No Git Bash: `echo $PATH`, `export MINHA_VAR=valor` e `which python`.

**Erro clássico: "comando não reconhecido"** quase sempre significa que o programa não está no PATH. Confira com `where nome`. Se não aparecer, reinstale marcando "Add to PATH" ou adicione a pasta em *Variáveis de Ambiente*. Depois **feche e abra o terminal**, porque sessões abertas não enxergam o PATH novo.

**Token ou senha:** prefira variável de ambiente a escrever a credencial direto no código, e nunca faça commit dela.

---

## 7. Python no dia a dia

```bash
python --version
python -m venv .venv                  # cria ambiente virtual
source .venv/Scripts/activate         # ativa no Git Bash
.venv\Scripts\Activate.ps1            # ativa no PowerShell
pip install pandas                    # instala pacote
pip freeze > requirements.txt         # salva dependências
pip install -r requirements.txt       # recria o ambiente
deactivate                            # sai do ambiente
```

**Por que usar venv:** cada projeto tem suas versões de pacotes, sem um quebrar o outro. Note que o caminho é `Scripts` no Windows e `bin` no Linux.

Se o PowerShell recusar ativar o venv ("execução de scripts desabilitada"), rode uma vez:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

---

## 8. Git: o essencial (Git Bash)

```bash
git status                    # o que mudou
git pull                      # traz alterações do remoto
git add .                     # prepara tudo
git commit -m "mensagem"      # registra
git push                      # envia
git log --oneline -10         # últimos 10 commits
git diff                      # diferenças ainda não preparadas
git switch -c minha-branch    # cria e troca de branch
git restore arquivo           # descarta alterações do arquivo (irreversível)
```

**Hábito que evita problema:** rode `git status` antes de `add`, e `git pull` antes de começar a trabalhar.

---

## 9. Atalhos que poupam tempo

| Atalho | Efeito |
| --- | --- |
| `Tab` | Autocompleta nomes de pasta/arquivo |
| `↑` / `↓` | Navega no histórico de comandos |
| `Ctrl + C` | Interrompe o comando em execução |
| `Ctrl + R` | Busca no histórico (Git Bash) |
| `Ctrl + L` ou `clear` / `cls` | Limpa a tela |
| `Ctrl + A` / `Ctrl + E` | Início / fim da linha (Git Bash) |
| `history` | Lista comandos anteriores |
| Clique direito (no PowerShell) | Cola o texto copiado |

**Abrir terminal na pasta certa:** no Explorer, clique na barra de endereço, digite `cmd` ou `powershell` e Enter. Ou clique com o botão direito na pasta → *Open Git Bash here*.

---

## 10. Encadear e redirecionar

```bash
comando1 && comando2          # roda o 2º só se o 1º der certo
comando > saida.txt           # grava a saída (sobrescreve)
comando >> saida.txt          # acrescenta ao final
comando1 | comando2           # passa a saída do 1º como entrada do 2º
```

Exemplo: listar pastas e salvar num arquivo:

```bash
ls -la > lista.txt
```

---

## 11. Quando algo dá errado

| Sintoma | O que verificar |
| --- | --- |
| `command not found` / `não reconhecido` | Está no PATH? (`where` / `which`) Reabriu o terminal? |
| `Permission denied` / `Acesso negado` | Arquivo aberto em outro programa? Precisa rodar como administrador? |
| Caracteres estranhos (acentos) | Codificação: use `chcp 65001` no CMD ou salve o arquivo em UTF-8 |
| Comando "travado" | `Ctrl + C`. Se não responder, feche a aba |
| Caminho não encontrado | Confira com `pwd` e `ls` onde você realmente está |
| Erro de SSL/certificado em rede corporativa | Pode ser proxy da empresa; fale com a TI antes de desabilitar verificação |

**Para pedir ajuda (ao Claude ou à TI):** copie a mensagem de erro **inteira**, o comando que rodou e em qual terminal (Git Bash ou PowerShell). Isso resolve a maioria dos casos de primeira.

---

## 12. Cola rápida

```text
pwd / ls / cd          onde estou, o que tem aqui, ir para
grep / sls / find      buscar texto ou arquivo
tasklist / taskkill    ver e encerrar processos
ipconfig / ping        diagnóstico de rede
netstat -ano           portas em uso
where / which          achar executável
python -m venv .venv   ambiente virtual
git status             sempre antes de mexer
```