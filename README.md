# AWS Security Automation — Foundation / Bootstrap

<p align="center">
  <a href="#-português">Português</a> •
  <a href="#-english">English</a>
</p>

---

## 🇧🇷 Português

Antes de escanear e corrigir problemas de segurança, há um problema mais fundamental a ser resolvido: **padronização**.

Este repositório define uma base compartilhada para automação de segurança na AWS, fornecendo um modelo de execução consistente, saídas versionadas e uma estrutura de repositório reutilizável para scanners e scripts de remediação.

> **Importante:** Este repositório é um template do GitHub (*GitHub template*).  
> Cada módulo de automação de segurança é criado como seu próprio repositório a partir deste template, contendo toda a estrutura de diretórios internamente.

Sem uma base comum, a automação de segurança torna-se rapidamente uma coleção de scripts desconectados. Este projeto transforma esses scripts em um framework coeso e auditável.

### Objetivos do Projeto

Esta versão fundacional concentra-se em:

- **Estabelecer um contrato único de execução** para todos os scripts.
- **Produzir saídas versionadas e auditáveis** para manter um histórico imutável de eventos.
- **Permitir a reutilização** em ambientes escaláveis:
  - Ambientes multi-contas (*multi-account*)
  - Pipelines de CI/CD
  - Fluxos analíticos e relatórios no Amazon Athena
  - Arquiteturas de Remediação como Código (*Remediation-as-Code - RaC*)
- **Criar uma base escalável** para futuras versões de automação de segurança.

Este framework atua como a pedra fundamental definitiva para toda a série de automação.

### Contrato de Execução (Scan / Plan / Remediate)

Todos os projetos criados a partir deste template seguem um contrato previsível de três fases:

1. **Scan (Somente Leitura / Read-Only)**  
   Produz descobertas granulares e evidências criptográficas. Nunca altera o estado ou as configurações do ambiente.
2. **Plan (Opcional, Não Mutável / Non-Mutating)**  
   Transforma as descobertas brutas do scanner em ações operacionais propostas, funcionando como uma pré-visualização de impacto e mudanças.
3. **Remediate (Alteração de Estado, Controlado / State-Changing, Controlled)**  
   Aplica as alterações de engenharia de forma explícita e oferece suporte estrito ao modo de teste (*dry-run*). Produz evidências precisas da remediação (planejado vs. aplicado).

> **Regra de Ouro:** Nenhuma remediação acontece acidentalmente — cada execução do plano de controle suporta e encoraja a validação prévia via *dry-run*.

### Verificação Automatizada e Testes do Plano de Controle

Para garantir que cada controle de segurança se comporte exatamente como projetado, este template introduz uma camada de testes automatizados orquestrada pelo script `test-control-plane.sh`.

Este script atua como o motor central de verificação do módulo, validando todo o ciclo de vida de cada controle de segurança em ambientes reais ou simulados por meio de uma sequência de 4 etapas:

1. **Provisionamento do Estado Inseguro:** Coordena com o diretório `lab_insecure/` para implantar recursos intencionalmente vulneráveis, simulando um desvio de conformidade (*drift*) ou falha de *compliance* do mundo real.
2. **Validação da Detecção (Scan):** Aciona os scripts em `scanner/` para verificar se o plano de controle identifica a vulnerabilidade com precisão e registra as descobertas corretamente.
3. **Avaliação da Estratégia (Plan):** Testa a fase de planejamento para garantir que a mitigação proposta esteja alinhada às suas diretrizes arquiteturais sem alterar recursos.
4. **Aplicação e Validação da Remediação (Remediate):** Executa os scripts em `remediate/` (primeiro em modo *dry-run*, depois em modo de aplicação efetiva) e confirma se o estado vulnerável foi corrigido com sucesso.

Ao executar `test-control-plane.sh`, você garante um ciclo de feedback rápido e confiável para desenvolver, testar e auditar proteções de segurança localmente antes de promovê-las aos pipelines de produção.

### Implantação e Replicação via AWS CloudShell

O AWS CloudShell fornece um terminal baseado em navegador, pré-autenticado e com a AWS CLI e o Git já instalados, tornando-o o ambiente ideal para replicar todo o ecossistema do plano de controle.

Em vez de clonar repositórios individualmente, você pode usar a GitHub CLI (`gh`) nativa ou autenticada no CloudShell para clonar em lote todos os repositórios relacionados e executar a suíte de verificação local.

#### Passo 1: Autenticar e Clonar em Lote
Autentique sua sessão do GitHub caso necessário e execute o loop de descoberta para clonar automaticamente todos os repositórios de plano de controle correspondentes ao seu perfil de ecossistema:

```bash
# Clona todos os planos de controle do ecossistema de uma só vez
for repo in $(gh repo list -L 100 --json nameWithOwner -q '.[].nameWithOwner' | grep -E 'AWS|Plane'); do
    gh repo clone "$repo"
done
```

#### Passo 2: Acessar e Configurar o Módulo Alvo
Acesse a pasta do plano de controle que deseja testar e conceda permissão de execução ao orquestrador principal de testes:

```bash
cd Security-Control-Plane
chmod +x test-control-plane.sh
```

#### Passo 3: Executar a Suíte de Verificação
Execute o motor de automação de testes para implantar o laboratório, validar as métricas de detecção e realizar a remediação em modo *dry-run*:

```bash
./test-control-plane.sh
```

### Estratégia de Saídas (Outputs)

Todas as execuções geram saídas imutáveis com registros de data e hora (*timestamps*):

- **As saídas nunca são sobrescritas**, garantindo a integridade histórica.
- **Cada sequência de execução é auditável de forma independente.**
- **Legível por máquina por padrão**, simplificando agregações.

Formatos recomendados:
- **JSON:** A fonte da verdade definitiva para dados brutos de execução.
- **CSV:** Dados estruturados ideais para ingestão analítica e consultas no Amazon Athena.
- **Markdown:** Resumos limpos e legíveis por humanos, projetados para triagem rápida de engenharia.

As saídas integram-se nativamente com Amazon Athena, barreiras de controle (*gates*) de CI/CD, plataformas de SIEM/SOAR e fluxos de auditoria de engenharia.

### Como Utilizar Este Template

1. Clique em **Use this template** no repositório principal no GitHub.
2. Crie um novo repositório dedicado a um domínio específico de segurança.
3. Implemente a lógica customizada do domínio de infraestrutura sob:
   - `scanner/`
   - `remediate/`
   - `lab_insecure/`
4. Execute `./test-control-plane.sh` (localmente ou via execução em lote no AWS CloudShell) para validar as implementações de ponta a ponta.
5. Mantenha o contrato de execução e a estrutura de saídas imutáveis preservados.

> Cada módulo permanece totalmente autocontido, enquanto todos os módulos em sua *landing zone* comportam-se de maneira consistente.

### Implementações de Referência

Os seguintes repositórios compõem o ecossistema principal criado a partir desta base e demonstram como o template é aplicado em diferentes pilares de segurança:

- [AWS Security Scripts](#) — *Link*
- [AWS Security Hub as Control Plane](#) — *Link*
- [AWS IAM & Identity as Control Plane (Inclui CIEM / Gerenciamento de Privilégios)](#) — *Link*
- [AWS Governance as Control Plane](#) — *Link*
- [AWS Software Defined Perimeter as Control Plane](#) — *Link*
- [AWS PaaS & Managed Services Permissions Control Plane (CodeBuild, CodeDeploy, SageMaker, RDS)](#) — *Link*
- [AWS Network Traffic as Control Plane (Roteamento VPC, Security Groups, NACLs, portas expostas)](#) — *Link*
- [AWS Credentials & Exposure Monitoring Control Plane (HIBP, validação e remediação de access keys)](#) — *Link*

> **Nota Estratégica:** Cada repositório segue o mesmo contrato de execução (*scan* → *plan* → *remediate*), utiliza o script `test-control-plane.sh` para testes de regressão confiáveis de cada controle e gera saídas versionadas. Eles podem ser utilizados individualmente ou integrados a um ecossistema unificado de Plano de Controle de Segurança.

### Estrutura do Repositório

Cada repositório derivado deste template possui exatamente a seguinte estrutura:

```text
aws-security-automation-<modulo>/
├── test-control-plane.sh  # Motor de orquestração de testes para controles de segurança
├── scanner/               # Scripts de detecção (somente leitura)
├── remediate/             # Scripts de remediação (suporta dry-run)
├── lab_insecure/          # Recursos intencionalmente inseguros para testes
├── reports/               # Relatórios consolidados e sumários
├── outputs/               # Saídas de execução versionadas (imutáveis)
├── lib/                   # Utilitários compartilhados (CLI, clientes AWS, gravadores, validadores)
└── docs/                  # Documentação, diagramas de arquitetura e guias
```

---

## 🇺🇸 English

Before scanning and fixing security issues, there is a more fundamental problem to solve: **standardization**.

This repository defines a shared foundation for AWS security automation, providing a consistent execution model, versioned outputs, and a reusable repository structure for scanners and remediation scripts.

> **Important:** This is a GitHub template.  
> Each security automation module is created as its own repository using this template, and each repository contains the full directory structure inside itself.

Without a common foundation, security automation quickly becomes a collection of disconnected scripts. This project turns those scripts into a cohesive, auditable framework.

### Project Goals

This foundational release focuses on:

- **Establishing a single execution contract** for all scripts.
- **Producing versioned, auditable outputs** to maintain an immutable history of events.
- **Enabling reuse** across scaling environments:
  - Multi-account environments
  - CI/CD pipelines
  - Athena analytics and reporting workflows
  - Remediation-as-Code (RaC) architectures
- **Creating a scalable base** for future security automation releases.

This framework serves as the definitive cornerstone for the entire automation series.

### Execution Contract (Scan / Plan / Remediate)

All projects created from this template follow a predictable, three-phase contract:

1. **Scan (Read-Only)**  
   Produces granular findings and cryptographic evidence. It never mutates state or environment configurations.
2. **Plan (Optional, Non-Mutating)**  
   Transforms raw scanner findings into proposed operational actions, acting as a preview of impact and changes.
3. **Remediate (State-Changing, Controlled)**  
   Applies engineering changes explicitly and strictly supports dry-run mode. Produces precise remediation evidence (planned vs. applied).

> **Key Rule:** No remediation happens accidentally — every single control plane execution supports and encourages dry-run validation first.

### Automated Verification & Control Plane Testing

To guarantee that each security control behaves exactly as designed, this template introduces an automated testing layer driven by the `test-control-plane.sh` script.

This script acts as the core verification engine for the module, validating the entire lifecycle of each security control against live or simulated environments using a 4-step sequence:

1. **Deploying the Insecure State:** Coordinates with the `lab_insecure/` directory to deploy intentionally vulnerable resources, simulating a real-world drift or compliance failure.
2. **Validating Detection (Scan):** Triggers the `scanner/` scripts to verify that the control plane accurately flags the vulnerability and logs the findings properly.
3. **Evaluating the Strategy (Plan):** Tests the plan phase to ensure the proposed mitigation matches your architectural guardrails without mutating resources.
4. **Applying and Verifying Remediation (Remediate):** Executes the `remediate/` scripts (first in dry-run, then in enforcement mode) and verifies that the vulnerable state has been successfully corrected.

By running `test-control-plane.sh`, you ensure a reliable, rapid feedback loop for developing, testing, and auditing security guardrails locally before promoting them to production pipelines.

### Deployment & Replication via AWS CloudShell

AWS CloudShell provides a pre-authenticated, browser-based shell that already has the AWS CLI and Git installed, making it the ideal environment to replicate your entire control plane ecosystem.

Instead of cloning repositories individually, you can use the GitHub CLI (`gh`) already built-in or authenticated inside CloudShell to replicate all related control plane repositories in batch, and then execute the local verification suite.

#### Step 1: Authenticate and Clone in Batch
Authenticate your GitHub session if required, and execute the discovery loop to automatically clone all security control plane repositories matching your ecosystem profile:

```bash
# Bulk clone all ecosystem control planes at once
for repo in $(gh repo list -L 100 --json nameWithOwner -q '.[].nameWithOwner' | grep -E 'AWS|Plane'); do
    gh repo clone "$repo"
done
```

#### Step 2: Navigate and Configure Target Module
Enter the specific control plane folder you intend to test and grant execution permissions to the core testing orchestrator:

```bash
cd Security-Control-Plane
chmod +x test-control-plane.sh
```

#### Step 3: Execute Verification Suite
Run the test automation engine to deploy the laboratory infrastructure, validate detection metrics, and perform dry-run remediation:

```bash
./test-control-plane.sh
```

### Output Strategy

All executions generate immutable, timestamped outputs:

- **Outputs are never overwritten**, ensuring historical integrity.
- **Each execution sequence is independently auditable.**
- **Machine-readable by default** to simplify aggregation.

Recommended formats include:
- **JSON:** The definitive source of truth for raw execution data.
- **CSV:** Structured data optimal for analytical ingestion and Athena queries.
- **Markdown:** Clean, human-readable summaries designed for quick engineering triage.

Outputs integrate natively with Amazon Athena, CI/CD deployment gates, SIEM/SOAR platforms, and engineering audit workflows.

### How This Template Is Used

1. Click **Use this template** on the upstream GitHub repository.
2. Create a new repository tailored to a specific security domain.
3. Implement your custom infrastructure domain logic under:
   - `scanner/`
   - `remediate/`
   - `lab_insecure/`
4. Run `./test-control-plane.sh` (locally or via bulk execution in AWS CloudShell) to validate your control plane implementations end-to-end.
5. Keep the execution contract and immutable output structure intact.

> Each module remains completely self-contained, but all modules across your landing zone behave consistently.

### Reference Implementations

The following repositories form the core ecosystem created using this foundation and demonstrate how the template applies across distinct security pillars:

- [AWS Security Scripts](#) — *Link*
- [Security Hub as Control Plane](#) — *Link*
- [IAM & Identity as Control Plane (Includes CIEM / Privilege Management)](#) — *Link*
- [AWS Governance as Control Plane](#) — *Link*
- [Software Defined Perimeter as Control Plane](#) — *Link*
- [PaaS & Managed Services Permissions Control Plane (CodeBuild, CodeDeploy, SageMaker, RDS)](#) — *Link*
- [AWS Network Traffic as Control Plane (VPC routing, Security Groups, NACLs, exposed ports)](#) — *Link*
- [Credentials & Exposure Monitoring Control Plane (HIBP, access key validation & remediation)](#) — *Link*

> **Strategic Note:** Each repository follows the same execution contract (*scan* → *plan* → *remediate*), leverages `test-control-plane.sh` for reliable regression testing of every control, and produces versioned outputs. They can be used independently or as part of a unified Security Control Plane ecosystem.

### Repository Structure

Every repository created from this template includes this exact structure inside itself:

```text
aws-security-automation-<module>/
├── test-control-plane.sh  # Test orchestration engine for security controls
├── scanner/               # Detection scripts (read-only)
├── remediate/             # Remediation scripts (supports dry-run)
├── lab_insecure/          # Intentionally insecure resources for testing
├── reports/               # Consolidated reports and summaries
├── outputs/               # Versioned execution outputs (immutable)
├── lib/                   # Shared helpers (CLI, AWS clients, writers, validators)
└── docs/                  # Documentation, architectural diagrams, and guides
```
