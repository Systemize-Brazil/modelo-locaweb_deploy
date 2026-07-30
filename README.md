# Modelo de Deploy Locaweb com Multi-Ambientes (Produção e Homologação)

Este repositório contém a configuração de integração contínua (CI) e deploy contínuo (CD) utilizando **GitHub Actions** para enviar automaticamente os arquivos do projeto para servidores hospedados na Locaweb, via protocolo FTP.

Ele é especialmente configurado para suportar **Múltiplos Ambientes** (Homologação e Produção), permitindo fluxos de trabalho organizados e seguros, essenciais para arquiteturas complexas como ERPs multitenant e sistemas de contabilidade em PHP 8.

---

## 🚀 Como funciona o Workflow?

O workflow `Deploy Locaweb` está configurado no arquivo `deploy_locaweb.yml` e possui duas formas principais de ser acionado:

### 1. Automático (Push)
- Um *push* na branch `homologacao` inicia automaticamente o deploy para o ambiente de **Homologação**.
- Um *push* na branch `main` (ou merge aprovado) inicia automaticamente o deploy para o ambiente de **Produção**.

### 2. Manual (Workflow Dispatch)
Através da aba *Actions* no GitHub, você pode rodar o workflow manualmente. Ao clicar em "Run workflow", você verá um menu *dropdown* perguntando:
- **"Qual ambiente você deseja atualizar?"** (Opções: `homologacao` ou `producao`).

---

## ⚙️ Detalhamento dos Jobs e Etapas

O script é dividido em dois *jobs* principais, cada um atrelado a um *Environment* (Ambiente) específico no GitHub. 

### Job 1: `deploy_producao`
Este job roda caso ocorra um push na branch `main` ou se "producao" for selecionado na execução manual.

### Job 2: `deploy_homologacao`
Este job roda caso ocorra um push na branch `homologacao` ou se "homologacao" for selecionado na execução manual.

---

### As Etapas (Steps) de cada Job:

Ambos os jobs executam basicamente as mesmas 3 etapas, mudando apenas os *secrets* que puxam de cada ambiente.

#### 0. 🔒 Validar Secrets do Ambiente
Um script shell simples que verifica se as três variáveis de ambiente necessárias (`FTP_SERVER`, `FTP_USERNAME`, e `FTP_PASSWORD`) estão preenchidas. 
* **Por que?** Evita que o workflow fique rodando em vão e falhe só no momento da transferência por falta de credenciais.

#### 1. 🚚 Obter o código do projeto (`actions/checkout@v4`)
Baixa o código-fonte para a máquina virtual do GitHub.
* **Detalhe importante:** Utiliza o parâmetro `fetch-depth: 2`. Isso permite que a action FTP consiga comparar o commit atual com o anterior e envie *apenas* os arquivos que foram alterados, economizando muito tempo de deploy.

#### 2. 📂 Sincronizar arquivos para o servidor (`SamKirkland/FTP-Deploy-Action@v4.4.0`)
Ação principal que realiza o upload FTP para a Locaweb. Configurações chaves utilizadas:
- **`server-dir`**: Está configurado para `/public_html/`. (Certifique-se de que a barra final esteja sempre presente).
- **`state-name`**: Cria arquivos ocultos diferentes (`.estado-deploy-producao.json` e `.estado-deploy-homologacao.json`) para rastrear o que já foi enviado em cada servidor, impedindo conflitos.
- **`exclude`**: Ignora pastas desnecessárias para o servidor de produção, como `.git`, `node_modules`, pastas do GitHub e arquivos Markdown/gitignore.

---

## 🛠️ Como configurar no seu Repositório

Para que esse workflow funcione perfeitamente, você deve configurar os *Environments* e os *Secrets* no painel do GitHub.

1. Vá em **Settings** > **Environments**.
2. Clique em **New environment** e crie um chamado `producao`.
3. Repita o processo e crie outro ambiente chamado `homologacao`.
4. Dentro de *cada um dos ambientes criados*, adicione os seguintes **Environment secrets**:
   - `FTP_SERVER`: O IP ou hostname do servidor FTP da Locaweb correspondente àquele ambiente (ex: `ftp.meusistema.com.br`).
   - `FTP_USERNAME`: O nome de usuário do FTP.
   - `FTP_PASSWORD`: A senha do FTP.

> 💡 **Nota:** Como os secrets estão separados por ambiente (`environment: producao` e `environment: homologacao`), você pode usar credenciais e servidores completamente diferentes para a fase de testes e para a fase de produção, garantindo segurança na separação de dados.
