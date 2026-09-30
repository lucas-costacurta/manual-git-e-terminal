# Tutorial: Padronização de Repositórios (GitHub + Databricks + Power BI)

**Time COIN — SEDUC-SP**

> **Versão do padrão:** 1.0 (23/09/2026). Se o modelo de repositório mudar, confira se este manual está na versão mais recente.

## 1. Por que padronizar?

Criamos um **repositório-modelo** (`modelo_repositorio`) para que todo projeto do time nasça com a mesma estrutura. Assim, criar, manter e auditar um projeto fica mais fácil, organizado e previsível, e qualquer pessoa do time consegue entender e dar continuidade ao que foi desenvolvido.

O modelo contém:

| Item | Para que serve |
|---|---|
| `DDL/` | Scripts de definição de estrutura (ex.: `CREATE TABLE`, `ALTER TABLE`). |
| `DML/` | Scripts de manipulação/carga de dados (ex.: `INSERT`, `MERGE`, transformações). |
| `PBIP/` | Projeto do Power BI em formato versionável (`.pbip`). |
| `README.md` | Documentação viva: o que o projeto faz e o histórico de alterações. |

**Regra de ouro:** a branch `main` é a versão de produção. Os pipelines rodam a partir dela, então **nunca desenvolvemos direto na `main`**. Todo desenvolvimento acontece em uma branch pessoal de desenvolvimento.

---

## 2. Criar um novo repositório a partir do modelo

*Esta seção é para quem vai iniciar um projeto novo. Se você já está dentro de um repositório criado, pode ir direto para a seção 3.*

1. No GitHub, abra o repositório `modelo_repositorio` da organização **COIN-SEDUC**.
2. Clique no botão verde **Use this template** → **Create a new repository**.
3. Preencha:
   - **Owner:** `COIN-SEDUC`
   - **Repository name:** nome do projeto, sem espaços e em minúsculas (ex.: `dashboard_concluintes`). *[Ajuste aqui se o time tiver outra convenção de nomes.]*
   - **Visibility:** **Private**
   - **Description:** uma frase dizendo o objetivo do projeto.
4. Clique em **Create repository**.

O novo repositório já vem com `DDL`, `DML`, `PBIP` e `README.md`. Abra o `README.md` e preencha o nome do projeto, o objetivo e os responsáveis.

> Um repositório criado a partir de um template começa com histórico limpo, sem carregar os commits do modelo.

---

## 3. Trazer o repositório para o Databricks

Usaremos **duas cópias** do repositório no Databricks:

- **Pasta compartilhada** → contém a `main`. É a que os jobs/pipelines executam.
- **Pasta pessoal** (prefixo `dev`) → sua cópia de trabalho, onde você desenvolve e testa sem afetar ninguém.

### 3.1 Pasta compartilhada (branch `main`)

*Normalmente feita uma única vez, por quem cria o projeto.*

1. No Databricks, vá em **Workspace** → pasta compartilhada do time.
2. Clique em **Create** → **Git folder**.
3. Cole a URL do repositório (`https://github.com/COIN-SEDUC/nome_do_repositorio`), confirme o provedor **GitHub** e crie.
4. Confirme que a branch é a `main`.

> Se o Databricks pedir credenciais, configure sua conta GitHub em **Settings → Linked accounts** (com um Personal Access Token).

### 3.2 Pasta pessoal (branch de desenvolvimento)

1. Vá em **Workspace → Users → seu usuário** (sua pasta pessoal).
2. **Create → Git folder**, com a **mesma URL** do repositório.
3. Dê um nome com prefixo `dev`, por exemplo `dev_dashboard_concluintes`.
4. Dentro dessa pasta, abra o menu de Git (ícone da branch), clique em **Create branch** e crie a sua branch de desenvolvimento no padrão **`dev_seunome`** (ex.: `dev_lucas`).
5. Confirme que a pasta pessoal está apontando para `dev_seunome`, **não** para a `main`.

Pronto: tudo que você editar na pasta pessoal fica isolado. Se algo quebrar ou uma feature ficar pela metade, os pipelines de produção não são afetados.

> Se mais de uma pessoa trabalha no mesmo projeto, cada uma cria a sua branch (`dev_lucas`, `dev_maria`, ...).

---

## 4. Ciclo de desenvolvimento (Databricks)

Vale para qualquer nova feature ou correção de bug em scripts, notebooks e tabelas.

1. **Atualize sua branch** antes de começar: na pasta pessoal, faça **Pull** para trazer o que houver de novo.
2. **Desenvolva na pasta `dev`**: a alteração vai em `DDL/` (estrutura) ou `DML/` (carga e transformação), conforme o caso.
3. **Teste** antes de qualquer commit. Rode os notebooks/queries e confira os resultados.
4. **Atualize o `README.md`** com a data e o que foi feito (modelo na seção 6).
5. **Revise as alterações** na tela de Git do Databricks (diff) e confirme que só subirão os arquivos que você quis alterar.
6. **Commit e push** na sua branch `dev_seunome`, com uma mensagem clara (ex.: `fix: corrige filtro de status na carga de matrículas`).
7. **Leve as alterações para a `main`** (seção 5).
8. **Valide o pipeline** (seção 5.3).

---

## 5. Levar as alterações para a `main` e validar

### 5.1 Merge

1. Abra a **pasta compartilhada** (a que está na `main`).
2. Faça **Pull** para garantir que ela está atualizada.
3. No menu de Git, use **Merge** e selecione a sua branch `dev_seunome` para incorporá-la à `main`.
4. Resolva conflitos, se houver, e faça o **push** da `main`.

*Alternativa:* abrir um **Pull Request** no GitHub de `dev_seunome` para `main`. É a opção mais indicada quando outra pessoa precisa revisar o que você fez.

### 5.2 Atualize sua pasta pessoal

Depois do merge, volte à pasta `dev` e faça **Pull da `main`** para que sua branch fique alinhada com a produção.

### 5.3 Validar o pipeline

**Se o projeto tem um pipeline/job ativo**, confirme que ele continua funcionando:

1. Vá em **Jobs & Pipelines** e abra o job do projeto.
2. Rode-o manualmente (ou aguarde a próxima execução agendada).
3. Verifique se terminou com **sucesso** e se os dados de saída estão corretos.
4. Se falhou, **reverta ou corrija imediatamente** e avise o time.

---

## 6. Como registrar no README

Toda alteração entra no `README.md` **antes** do merge, em uma seção de histórico. Sugestão de formato:

```markdown
## Histórico de alterações

| Data | Responsável | Tipo | O que foi feito |
|---|---|---|---|
| 23/09/2026 | Nome | Feature | Adicionada coluna X na tabela Y (DDL) e ajustada a carga (DML). |
| 20/09/2026 | Nome | Correção | Corrigido filtro de status que duplicava registros. |
```

Registre: **data**, **quem fez**, **tipo** (feature, correção, ajuste) e **o que mudou e por quê**, de forma que outra pessoa entenda sem precisar perguntar.

---

## 7. Power BI (PBIP)

O formato `.pbip` salva o relatório como **arquivos de texto** (em vez de um `.pbix` binário). Isso permite versionar no GitHub, ver o que mudou entre versões e trabalhar com branches, exatamente como fazemos com o código.

### 7.1 Preparação (uma vez)

1. Instale o **Git** e clone o repositório na sua máquina (GitHub Desktop, VS Code ou `git clone`).
2. No Power BI Desktop, verifique se a opção de salvar como projeto está habilitada. Se não aparecer, ative em **Arquivo → Opções e configurações → Opções → Recursos de visualização** (**Power BI Project (.pbip)**) e reinicie o Desktop.

### 7.2 Criar o arquivo PBIP

1. Crie ou abra o seu relatório no Power BI Desktop.
2. Vá em **Arquivo → Salvar como** e escolha o tipo **Power BI Project (.pbip)**.
3. Salve **dentro da pasta `PBIP/`** do repositório clonado.
4. Serão criados o arquivo `.pbip` e as pastas `.Report` e `.SemanticModel`. Esses são os itens versionados.

### 7.3 Ciclo de desenvolvimento (Power BI)

O ciclo é o mesmo do Databricks:

1. Na sua máquina, crie/troque para a sua branch `dev_seunome` e faça **pull**.
2. Abra o `.pbip` e **desenvolva** a feature ou correção.
3. **Teste** (números conferem com a fonte? visuais, filtros e atualização funcionam?).
4. **Atualize o `README.md`** com data e o que foi feito.
5. **Revise** as alterações (diff) e faça **commit + push** na sua branch, com mensagem clara.
6. **Merge na `main`** (por Pull Request no GitHub, preferencialmente).
7. **Publique** a versão da `main` no workspace do Power BI Service e **valide** a atualização do modelo e o relatório publicado.

> Publique sempre a partir da versão que está na `main`, para que o que está no Service corresponda ao que está documentado no GitHub.

---

## 8. Usando o Genie Code

**Recomendado:** usar o Genie Code para **redigir/atualizar o README** (resumir alterações, organizar o histórico, padronizar a escrita). Lembre-se de conferir o texto antes de subir.

**Cuidado redobrado com commits:** evite pedir ao Genie Code que faça os commits por você. Um commit é um registro oficial do que entra no projeto, e ele pode incluir arquivos que você não pretendia alterar. Recomendamos que **os commits e todas as revisões sejam feitos por você, de forma clara**, conferindo o diff e escrevendo a mensagem antes de subir para o GitHub.

---

## 9. Checklist rápido

- [ ] Estou trabalhando na minha branch `dev_seunome` (não na `main`)
- [ ] Testei a alteração
- [ ] Atualizei o `README.md` com data e descrição
- [ ] Revisei o diff antes do commit
- [ ] Commit com mensagem clara e push feitos
- [ ] Merge na `main` concluído
- [ ] Pipeline/relatório validado após o merge
- [ ] Minha pasta `dev` está atualizada com a `main`

---

## 10. Por que vale o esforço

Esse processo adiciona um tempo extra a cada desenvolvimento: criar branch, testar, documentar, revisar e validar. Esse tempo é um investimento. Ele mantém tudo **documentado, auditável e seguro para a produção**, e garante que qualquer pessoa do time tenha condições de **entender e manter** o que foi desenvolvido, mesmo sem ter participado da construção.
