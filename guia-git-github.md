# Guia prático de Git e GitHub

Foco: uso diário no desktop (linha de comando), projetos Power BI (PBIP) e Databricks (interface, sem comandos).

---

## 1. O modelo mental (5 minutos que evitam 90% dos erros)

**Git** é o programa que registra o histórico do seu projeto. **GitHub** é o site que guarda uma cópia remota desse histórico.

Seus arquivos passam por 4 lugares:

```
Working directory  →  Staging area  →  Repositório local  →  Repositório remoto (GitHub)
 (você edita)        (git add)        (git commit)           (git push)
```

- **Working directory**: os arquivos como estão na sua pasta agora.
- **Staging area**: a "cesta" com o que vai entrar no próximo commit. Serve para você escolher *o que* entra.
- **Commit**: uma foto do projeto naquele momento, com mensagem e autor.
- **Remoto (origin)**: o repositório no GitHub. `push` envia, `pull` traz.

Regra de ouro: **commit é local**. Nada vai para o GitHub até você fazer `git push`.

---

## 2. Configuração inicial (uma vez por computador)

```bash
git config --global user.name "Lucas Costacurta Ferro"
git config --global user.email "seu-email-do-github@exemplo.com"
git config --global init.defaultBranch main
git config --global core.autocrlf true      # Windows: evita bagunça de quebra de linha
git config --list                           # confere tudo
```

Autenticação com o GitHub: use o **Git Credential Manager** (já vem com o Git for Windows; abre o navegador no primeiro `push`) ou um **Personal Access Token**. Nunca use sua senha, e nunca salve o token dentro de arquivos do projeto.

**Dica para o seu caso:** clone sempre numa pasta **fora do OneDrive** (ex.: `C:\repos`). O OneDrive sincroniza a pasta `.git` em paralelo com o Git e pode corromper o repositório.

---

## 3. Fluxo do dia a dia

### Começar um projeto

```bash
cd C:\repos
git clone https://github.com/COIN-SEDUC/nome-do-repo.git
cd nome-do-repo
```

`clone` baixa o repositório inteiro **com todo o histórico** e já configura o `origin`.

### O ciclo básico (decore este)

```bash
git pull                         # 1. traz o que mudou no GitHub (SEMPRE antes de começar)
# ... você trabalha, edita, salva arquivos ...
git status                       # 2. o que mudou?
git diff                         # 3. ver as mudanças linha a linha (ainda não "staged")
git add arquivo.sql              # 4. escolhe o que entra no commit
git commit -m "Ajusta filtro de status na consulta de matrículas"   # 5. registra
git push                         # 6. envia para o GitHub
```

- `git add .` adiciona tudo da pasta. É prático, mas **rode `git status` antes** para não levar arquivo indesejado.
- `git add -p` deixa você aprovar trecho por trecho. Ótimo para revisar o que está subindo.

### Trabalhando em dois computadores

O erro clássico é editar na máquina A, esquecer de enviar e editar na máquina B. Para evitar:

1. **Ao começar** em qualquer máquina: `git pull`.
2. **Ao terminar**: `git add`, `git commit`, `git push`. Mesmo que o trabalho esteja incompleto, comite numa branch sua (`dev`); é melhor um commit "WIP" do que arquivo esquecido na outra máquina.
3. Nunca deixe alterações sem commit ao trocar de máquina.

---

## 4. Comandos mais usados

| Objetivo | Comando |
|---|---|
| Ver o estado atual | `git status` |
| Ver histórico resumido | `git log --oneline --graph --all` |
| Ver o que mudou | `git diff` (não staged) / `git diff --staged` (staged) |
| Ver o remoto configurado | `git remote -v` |
| Baixar novidades sem mesclar | `git fetch` |
| Baixar e mesclar novidades | `git pull` (= `fetch` + `merge`) |
| Adicionar ao staging | `git add arquivo` / `git add .` / `git add -p` |
| Registrar | `git commit -m "mensagem"` |
| Enviar | `git push` |
| Ver branches | `git branch` (locais) / `git branch -a` (todas) |
| Criar e mudar de branch | `git switch -c nome-da-branch` |
| Mudar de branch | `git switch nome-da-branch` |
| Guardar alterações temporariamente | `git stash` / `git stash pop` |
| Ver quem alterou cada linha | `git blame arquivo` |

`fetch` vs `pull`: o `fetch` só *olha* o que mudou no GitHub; o `pull` também aplica na sua branch. Se quiser ver antes de aplicar, use `git fetch` e depois `git log HEAD..origin/main --oneline`.

---

## 5. Branches, merge e Pull Request

**Branch** é uma linha paralela de desenvolvimento. Ela permite mexer sem afetar o que está em produção.

Padrão que você já usa (e é uma boa prática):

- `main`: versão estável. É o que roda em produção/jobs. Ninguém edita direto.
- `dev` (ou `dev_nome`): onde cada pessoa trabalha.

```bash
git switch -c dev                # cria a branch dev e muda para ela
git push -u origin dev           # publica e liga a branch local à remota (só na primeira vez)
```

### Levando o trabalho da dev para a main

**Opção recomendada: Pull Request (PR) no GitHub.**
1. `git push` da sua branch.
2. No GitHub: *Compare & pull request* → base `main`, compare `dev`.
3. Descreva o que mudou, revise o "Files changed" e faça o *Merge*.
4. Localmente: `git switch main` e `git pull`.

O PR deixa registro de quem revisou e o quê, o que ajuda em auditoria. Também é o local onde outra pessoa pode comentar antes de entrar em `main`.

**Opção manual (via linha de comando):**
```bash
git switch main
git pull
git merge dev
git push
```

### Mantendo a dev atualizada com a main

Se a `main` recebeu mudanças (de outra pessoa, por exemplo), traga para a sua branch:
```bash
git switch dev
git fetch
git merge origin/main
```

---

## 6. Resolvendo conflitos

Conflito acontece quando duas pessoas (ou duas máquinas) alteram **o mesmo trecho** do mesmo arquivo. O Git não decide por você e marca assim:

```
<<<<<<< HEAD
sua versão
=======
versão que veio do outro lado
>>>>>>> origin/main
```

Passo a passo:
1. `git status` mostra os arquivos em conflito.
2. Abra o arquivo (o VS Code mostra botões *Accept Current / Incoming / Both*), escolha o resultado correto e **apague os marcadores** `<<<<<<<`, `=======`, `>>>>>>>`.
3. `git add arquivo` e `git commit` (ou `git merge --continue`).

Ficou perdido no meio do merge? `git merge --abort` volta ao estado anterior.

---

## 7. Desfazendo coisas (sem pânico)

| Situação | Comando |
|---|---|
| Descartar alterações **não commitadas** de um arquivo | `git restore arquivo` (⚠️ irreversível) |
| Tirar arquivo do staging (mantém a edição) | `git restore --staged arquivo` |
| Corrigir a mensagem do **último commit** (ainda não enviado) | `git commit --amend -m "nova mensagem"` |
| Desfazer o último commit **mantendo** as alterações | `git reset --soft HEAD~1` |
| Desfazer um commit **já enviado** (seguro) | `git revert <hash>` |
| Guardar o trabalho pela metade para trocar de branch | `git stash`, depois `git stash pop` |

Diferença importante: **`revert`** cria um *novo* commit que desfaz o antigo (seguro em histórico compartilhado). **`reset`** reescreve o histórico; use só em commits que ainda não foram para o GitHub. Evite `git reset --hard` e `git push --force`, a menos que saiba exatamente o que está fazendo.

Achou que perdeu um commit? `git reflog` mostra tudo por onde o Git passou e quase sempre dá para recuperar.

---

## 8. Power BI + Git

O arquivo `.pbix` é binário: o Git não consegue mostrar o que mudou nem mesclar. Por isso, use o formato **Power BI Project (`.pbip`)**, que salva o modelo e o relatório como pastas de texto (TMDL/JSON). Com isso você passa a ter:

- `git diff` mostrando medidas DAX, relacionamentos e tabelas alteradas;
- histórico real de quem mudou cada medida;
- possibilidade de revisar em Pull Request.

Como ativar: Power BI Desktop → *Arquivo → Opções → Recursos de visualização → "Armazenar o modelo semântico usando o formato TMDL"* (se ainda não estiver ativo) e depois *Salvar como → Power BI Project (.pbip)*.

**O que não deve ir para o Git** (o Power BI costuma gerar um `.gitignore` com isso, mas confira):

```gitignore
**/.pbi/localSettings.json
**/.pbi/cache.abf
```

Boas práticas específicas:
- **Feche o Power BI** antes de dar `git pull`/`switch`, senão ele pode sobrescrever ou travar arquivos.
- Faça commits pequenos: "cria medida de taxa de evasão" é melhor que "atualiza dashboard".
- Não versione dados (`.csv`, `.xlsx` grandes, extrações). O dado vem do Databricks; o Git guarda a *lógica*.

---

## 9. Databricks (Git folders, pela interface)

No Databricks você faz pela interface o que faria por comando:

| Interface | Equivale a |
|---|---|
| Botão **Pull** | `git pull` |
| **Commit & Push** (marca os arquivos, escreve a mensagem) | `git add` + `git commit` + `git push` |
| Seletor de **branch** / *Create branch* | `git switch` / `git switch -c` |
| **Merge** / **Rebase** no diálogo de Git | `git merge` / `git rebase` |

Boas práticas:
- Trabalhe **sempre na sua pasta pessoal (dev)**, nunca edite direto a pasta que roda os jobs.
- **Faça Pull antes de editar**, principalmente se você também mexe no mesmo projeto pelo desktop.
- Comite com mensagem clara: se usar o Genie Code/assistente para gerar código ou README, **revise o diff** antes do *Commit & Push*.
- Notebooks geram diffs grandes por causa de metadados. Prefira commitar SQL/Python em arquivos `.sql` e `.py` sempre que possível, e mantenha commits focados.
- Se aparecer conflito na interface, resolva no arquivo indicado e comite. A lógica é a mesma da seção 6.

---

## 10. Boas práticas

**Commits**
- Um commit = **uma ideia**. Facilita revisar e desfazer.
- Mensagem no imperativo, curta e específica: `Corrige join de turmas na tb_matriculas`. Evite `ajustes`, `teste`, `final_v2`.
- Se precisar explicar o porquê, use corpo na mensagem: `git commit` (sem `-m`) abre o editor.
- Comite com frequência, dê `push` ao fim do dia.

**Branches**
- Nunca trabalhe direto na `main`.
- Nome descritivo: `dev`, `dev_lucas`, `feature/filtro-por-diretoria`, `fix/erro-medida-evasao`.
- Apague branches já mescladas: `git branch -d nome` (local) e `git push origin --delete nome` (remoto).

**Segurança (muito importante em órgão público)**
- **Nunca** suba: tokens, senhas, strings de conexão, chaves de API, dados de alunos/CPF/dados pessoais.
- Use `.gitignore` desde o primeiro commit. Se algo sensível já subiu, apagar em um novo commit **não basta** (fica no histórico): avise o responsável e troque a credencial imediatamente.
- Se possível, ative o *secret scanning* do GitHub no repositório da organização.

**Documentação**
- `README.md`: o que é o projeto, como rodar, fontes de dados, o que mudou (data + resumo).
- `CONTRIBUTING.md`: o padrão de trabalho do time (branches, commits, revisão).
- Outros arquivos que o GitHub reconhece: `.gitignore`, `LICENSE`, `CODEOWNERS` (define quem revisa o quê), `.github/pull_request_template.md` (modelo de PR) e `SECURITY.md`.

**Rotina resumida**
1. `git pull`
2. Trabalhar
3. `git status` → `git diff`
4. `git add` (só o necessário)
5. `git commit -m "mensagem clara"`
6. `git push`
7. Se for para produção: Pull Request → revisão → merge na `main`

---

## 11. Erros comuns e o que fazer

| Mensagem / situação | Causa e solução |
|---|---|
| `fatal: not a git repository` | Você não está dentro da pasta do repo. `cd` para ela. |
| `rejected... (non-fast-forward)` ou `Updates were rejected` no push | O remoto tem commits que você não tem. Faça `git pull`, resolva conflitos se houver, e `push` de novo. |
| `Your local changes would be overwritten by merge` | Há edição não commitada. Comite ou use `git stash` antes do `pull`. |
| `You are in 'detached HEAD' state` | Você fez checkout de um commit, não de uma branch. `git switch main` (ou sua dev). |
| Arquivo indesejado no `git status` | Adicione ao `.gitignore`. Se já estava versionado: `git rm --cached arquivo`. |
| Subiu arquivo grande/sensível | Não force nada sozinho: avise o time, e revogue credenciais se for o caso. |

---

## Cola rápida (imprima ou deixe à mão)

```bash
git clone <url>                 # baixar repositório
git pull                        # atualizar antes de trabalhar
git switch -c dev               # criar branch de trabalho
git status                      # o que mudou?
git diff                        # ver mudanças
git add .                       # preparar (confira antes!)
git commit -m "mensagem"        # registrar
git push                        # enviar
git log --oneline --graph       # histórico
git stash / git stash pop       # guardar / recuperar trabalho pela metade
git restore arquivo             # descartar edição (irreversível)
git revert <hash>               # desfazer commit já publicado
```
