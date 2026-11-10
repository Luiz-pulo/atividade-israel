# atividade-israel

# DOCUMENTO DE ARQUITETURA E INFORMAÇÕES DE DESIGN (AID)

## Projeto: Hospital Connect — Plataforma Integrada de Telemedicina e Gestão Hospitalar

## SUMÁRIO

 1. [Bloco 1: Concepção do Sistema, Visão Geral e Viabilidade](#1-bloco-1-concepção-do-sistema-visão-geral-e-viabilidade)
 2. [Bloco 2: Atores, Personas e Engenharia de Requisitos](#2-bloco-2-atores-personas-e-engenharia-de-requisitos)
 3. [Bloco 3: Registros de Decisão de Arquitetura (ADRs)](#3-bloco-3-registros-de-decisão-de-arquitetura-adrs)
 4. [Bloco 4: Modelagem UML](#4-bloco-4-modelagem-uml)
 5. [Bloco 5: Modelo de Arquitetura C4](#5-bloco-5-modelo-de-arquitetura-c4)
 6. [Bloco 6: Engenharia de Banco de Dados e Adequação à LGPD](#6-bloco-6-engenharia-de-banco-de-dados-e-adequação-à-lgpd)
 7. [Bloco 7: Planejamento de Projeto, Custos e Matriz de Riscos](#7-bloco-7-planejamento-de-projeto-custos-e-matriz-de-riscos)
 8. [Bloco 8: Onboarding do Desenvolvedor (Guia "Dia 1 do Dev")](#8-bloco-8-onboarding-do-desenvolvedor-guia-dia-1-do-dev)
 9. [Bloco 9: Governança, Padrões de Código e DoD/DoR](#9-bloco-9-governança-padrões-de-código-e-doddor)

---

## 1. Bloco 1: Concepção do Sistema, Visão Geral e Viabilidade

### 1.1 Contexto e Escopo
O **Hospital Connect** é uma plataforma SaaS multitenant voltada para a gestão hospitalar integrada e prestação de serviços de telemedicina de alta disponibilidade. O sistema atende aos critérios do trabalho (mínimo de 4 atores e 12 funcionalidades principais).

* **Domínio:** Saúde Digital (E-Health) e Gestão Hospitalar.
* **Objetivo Principal:** Reduzir o tempo de espera em pronto-socorro virtual, garantir o histórico de saúde centralizado e assegurar conformidade total com a LGPD e regulamentações do Conselho Federal de Medicina (CFM).
* **Abordagem Técnica:** Arquitetura orientada a microsserviços desacoplados por mensageria via RabbitMQ e banco de dados relacional distribuído (PostgreSQL).

### 1.2 Estudo de Viabilidade Cloud (AWS Calculator)
Estimativa calculada com base na infraestrutura AWS para ambiente de produção:

| Componente Cloud | Recurso AWS Selecionado | Preço Mensal Estimado (USD) | Preço Mensal (BRL - 1 USD = 5,00 BRL) |
| :--- | :--- | :--- | :--- |
| **Cluster Kubernetes** | AWS EKS (2 Nodes t3.xlarge) | \$ 195.00 | R\$ 975,00 |
| **Banco de Dados** | AWS RDS PostgreSQL Multi-AZ (db.r6g.xlarge) | \$ 420.00 | R\$ 2.100,00 |
| **Cache em Memória** | ElastiCache Redis (cache.m6g.large) | \$ 110.00 | R\$ 550,00 |
| **Object Storage** | AWS S3 (Exames e Receitas - 500 GB) | \$ 15.00 | R\$ 75,00 |
| **Segurança e Rede** | WAF + Cloudfront + ALB | \$ 120.00 | R\$ 600,00 |
| **TOTAL ESTIMADO CLOUD** | **-** | **\$ 860.00 / mês** | **R\$ 4.300,00 / mês** |

---

## 2. Bloco 2: Atores, Personas e Engenharia de Requisitos

### 2.1 Atores e Matriz de Stakeholders
| Ator | Tipo | Descrição e Nível de Acesso |
| :--- | :--- | :--- |
| **Paciente** | Externo | Acessa agendamentos, realiza teleconsultas, visualiza prontuários e receitas digitais. |
| **Médico / Profissional de Saúde** | Interno | Atende teleconsultas, emite pareceres, preenche PEP e assina prescrições digitais. |
| **Recepcionista / Atendente** | Interno | Valida cadastros, confirma presenças presenciais e aloca salas de atendimento. |
| **Administrador Hospitalar** | Gestão | Gerencia contas de usuários, configura unidades, acessa relatórios de SLA e auditorias. |

### 2.2 Personas

#### Persona 1: Dr. Roberto Mendes (Médico Telemedicina)
* **Idade:** 42 anos | **Cargo:** Cardiologista
* **Motivações:** Agilidade no atendimento remoto, assinatura de receitas digitais sem fricção e acesso rápido ao histórico preexistente do paciente.
* **Frustrações:** Sistemas lentos que travam durante a chamada de vídeo e falta de integração entre a ficha do paciente e o prontuário.

#### Persona 2: Juliana Lima (Paciente)
* **Idade:** 29 anos | **Cargo:** Analista de Marketing
* **Motivações:** Agendar consultas sem precisar telefonar, receber alertas de horário e ter acesso às suas prescrições no smartphone.
* **Frustrações:** Perder receitas impressas, esperar em filas físicas e repetir o histórico médico a cada novo atendimento.

#### Persona 3: Carlos Eduardo (Administrador Hospitalar)
* **Idade:** 50 anos | **Cargo:** Diretor de Operações de TI
* **Motivações:** Manter o sistema operacional em 99,9% do tempo, garantir compliance integral com a LGPD e controlar custos de infraestrutura em nuvem.
* **Frustrações:** Sistemas que sofrem instabilidade em picos de agendamento e falta de trilha de auditoria para vazamento de dados.

### 2.3 Requisitos Funcionais (RFs) com Priorização MoSCoW

| ID | Descrição do Requisito Funcional | Categoria MoSCoW |
| :--- | :--- | :--- |
| **RF01** | O sistema deve permitir a autenticação multifator (MFA) para médicos e administradores. | Must Have |
| **RF02** | O paciente deve poder agendar consultas presenciais ou teleconsultas escolhendo especialidade e médico. | Must Have |
| **RF03** | O sistema deve disponibilizar salas virtuais de teleconsulta com suporte a áudio/vídeo via WebRTC. | Must Have |
| **RF04** | O médico deve registrar informações de atendimento no Prontuário Eletrônico do Paciente (PEP). | Must Have |
| **RF05** | O sistema deve emitir receitas médicas assinadas digitalmente utilizando certificados ICP-Brasil. | Must Have |
| **RF06** | O sistema deve enviar notificações push/E-mail/SMS para confirmação e lembrete de consultas. | Should Have |
| **RF07** | O paciente deve poder anexar exames prévios nos formatos PDF/JPEG/PNG antes da consulta. | Should Have |
| **RF08** | O recepcionista deve poder realizar a triagem inicial e dar entrada no atendimento presencial. | Should Have |
| **RF09** | O administrador deve ter acesso a dashboards com métricas de tempo médio de atendimento e no-show. | Could Have |
| **RF10** | O sistema deve permitir o pagamento online de consultas particulares via PIX e Cartão de Crédito. | Could Have |
| **RF11** | O paciente deve poder solicitar a exportação ou exclusão dos seus dados pessoais (LGPD). | Must Have |
| **RF12** | O farmacêutico deve conseguir validar o QR Code de uma prescrição eletrônica emitida pelo sistema. | Should Have |
| **RF13** | O sistema não permitirá o cancelamento de consultas a menos de 2 horas do horário sem justificativa. | Won't Have |
| **RF14** | O sistema não integrará transcrição de áudio por Inteligência Artificial nesta versão. | Won't Have |

### 2.4 Requisitos Não Funcionais (RNFs) Mensuráveis

| ID | Categoria | Descrição do Requisito Não Funcional Mensurável |
| :--- | :--- | :--- |
| **RNF01** | **Desempenho** | O tempo de resposta para qualquer consulta em API de leitura deve ser menor que 200ms no percentil 95 (p95). |
| **RNF02** | **Disponibilidade** | O sistema deve apresentar disponibilidade (SLA) de 99,9% ao mês (máximo de 43 minutos de downtime não planejado/mês). |
| **RNF03** | **Segurança** | Todos os dados em trânsito devem ser criptografados utilizando TLS 1.3 e os dados em repouso com AES-256. |
| **RNF04** | **Escalabilidade** | A arquitetura deve suportar até 5.000 requisições simultâneas por segundo (RPS) sem degradação do p95. |
| **RNF05** | **Conformidade** | O armazenamento e log de dados de saúde devem estar estritamente alinhados com a LGPD (Lei nº 13.709/2018). |
| **RNF06** | **Usabilidade** | A interface de telemedicina deve atingir pontuação mínima de 80 no teste System Usability Scale (SUS). |

### 2.5 Matriz de Rastreabilidade

| Requisito Funcional | Requisito Não Funcional | Caso de Uso | Módulo / Serviço |
| :--- | :--- | :--- | :--- |
| **RF02 (Agendar Consulta)** | RNF01, RNF04 | UC02 - Agendar Consulta | Scheduling Microservice |
| **RF03 (Salas WebRTC)** | RNF01, RNF02 | UC03 - Realizar Teleconsulta | Video Gateway Service |
| **RF04 (Prontuário PEP)** | RNF03, RNF05 | UC04 - Manter Prontuário | Electronic Health Record (EHR) |
| **RF05 (Assinatura ICP)** | RNF03, RNF05 | UC05 - Emitir Prescrição | Signature Integration Service |

### 2.6 Histórias de Usuário (User Stories - Given/When/Then)

#### US01 – Agendamento de Consulta pelo Paciente (Esforço: 5 Story Points)
* **Como** Paciente cadastrado no Hospital Connect,
* **Quero** selecionar a especialidade, o médico e o horário disponível no calendário,
* **Para que** eu possa agendar minha consulta de forma autônoma.
* **Cenário de Aceite (Given/When/Then):**
  * **Given** que eu esteja autenticado e navegue até "Novo Agendamento",
  * **When** eu selecionar "Cardiologia", escolher "Dr. Roberto" e confirmar "20/11/2026 às 14:00",
  * **Then** o sistema deve reservar o horário por 10 minutos, gerar o comprovante e enviar uma notificação push.

#### US02 – Atendimento e Registro de Prontuário PEP (Esforço: 8 Story Points)
* **Como** Médico com login validado e CRM ativo,
* **Quero** registrar a evolução do atendimento no prontuário do paciente durante a chamada,
* **Para que** as informações fiquem gravadas no histórico médico unificado.
* **Cenário de Aceite (Given/When/Then):**
  * **Given** que eu esteja em uma teleconsulta ativa com a paciente "Juliana Lima",
  * **When** eu preencher o campo "Diagnóstico", inserir o CID-10 e clicar em "Salvar Evolução",
  * **Then** os dados devem ser criptografados com chave AES-256 e gravados com log de auditoria associado ao CRM.

#### US03 – Assinatura Digital de Receita Médica (Esforço: 8 Story Points)
* **Como** Médico em atendimento,
* **Quero** assinar digitalmente a prescrição utilizando certificado ICP-Brasil,
* **Para que** o paciente receba uma receita válida legalmente.
* **Cenário de Aceite (Given/When/Then):**
  * **Given** que eu finalize a prescrição de medicamentos do paciente,
  * **When** eu selecionar "Assinar via Certificado Digital" e fornecer a chave de validação,
  * **Then** o sistema deve comunicar com o autorizador ICP-Brasil, gerar o PDF com carimbo de tempo e disponibilizar o QR Code de verificação.

---

## 3. Bloco 3: Registros de Decisão de Arquitetura (ADRs)

### ADR 01: Escolha do Estilo de Comunicação Entre Serviços
* **Status:** Aprovado
* **Contexto:** A plataforma necessita de comunicação entre agendamentos, prontuários, notificações e auditorias. Comunicações estritamente síncronas via REST HTTP provocariam acoplamento excessivo e falhas em cascata em horários de pico.
* **Decisão:** Adotar comunicação HTTP/REST (OpenAPI) para consultas síncronas de baixa latência e mensageria assíncrona orientada a eventos via **RabbitMQ** para tarefas de segundo plano (envio de notificações, geração de logs e auditoria).
* **Consequências:**
  * *Positivas:* Alta resiliência, desacoplamento e capacidade de absorver picos de carga.
  * *Negativas:* Necessidade de gerenciar a consistência eventual entre microsserviços.

### ADR 02: Estratégia de Persistência de Dados
* **Status:** Aprovado
* **Contexto:** O sistema lida com dados de saúde de alta criticidade e exige integridade relacional rígida (Prontuários, Agendamentos, Vínculos Médico-Paciente).
* **Decisão:** Adotar **PostgreSQL 16** como banco de dados relacional principal para o core business e **Redis** como camada de cache em memória para gestão de sessões e lock temporário de horários.
* **Consequências:**
  * *Positivas:* Garantia total das propriedades ACID, suporte a colunas JSONB para dados dinâmicos de exames e forte suporte a compliance LGPD.
  * *Negativas:* Necessidade de configuração de réplicas de leitura à medida que o volume de transações crescer.

---

## 4. Bloco 4: Modelagem UML

*Nota para apresentação PDF: Todos os diagramas devem ser renderizados com Título, Nomes dos Integrantes da Equipe e Data de Atualização no cabeçalho.*

### 4.1 Diagrama de Casos de Uso
```
                        +-------------------------------------------------------------+
                        |                      Hospital Connect                       |
                        +-------------------------------------------------------------+
                                                                                       
     (( Paciente )) --------> ( UC01: Manter Cadastro )                                
           |                                                                           
           +----------------> ( UC02: Agendar Consulta ) <-------- <<include>> -------- ( UC06: Notificar Horário )
           |                         |                                                 
           |                         +---------------------------- <<extend>> --------- ( UC10: Efetuar Pagamento )
           |                                                                           
           +----------------> ( UC03: Realizar Teleconsulta )                          
           |                                                                           
     (( Médico )) ----------> ( UC04: Preencher Prontuário - PEP )                     
           |                         |                                                 
           |                         +---------------------------- <<include>> -------- ( UC05: Assinar Prescrição )
           |                                                                           
(( Recepcionista )) --------> ( UC07: Fazer Triagem Presencial )                        
                                                                                       
 (( Administrador )) -------> ( UC08: Gerenciar Usuários )                             
           |                                                                           
           +----------------> ( UC09: Visualizar Audit Logs LGPD )                     
```

### 4.2 Diagrama de Classes (Totalmente Tipado e com Visibilidades Explicitadas)
```
+------------------------------------+          +------------------------------------+
|             Paciente               |          |               Médico               |
+------------------------------------+          +------------------------------------+
| - idPaciente: Long                 |          | - idMedico: Long                   |
| - cpf: String                      |          | - crm: String                      |
| - nome: String                     |          | - ufCrm: String                    |
| - dataNascimento: LocalDate        |          | - especialidade: String            |
| - email: String                    |          | - valorConsulta: BigDecimal        |
| - telefone: String                 |          +------------------------------------+
+------------------------------------+          | +consultarAgenda(): List<Agenda>   |
| +agendarConsulta(): Consulta       |          | +atenderPaciente(): Void           |
| +visualizarProntuario(): Prontuario|          +------------------------------------+
+------------------------------------+                            1
                  1                                               |
                  |                                               |
                  | 1..*                                          | 1..*
+------------------------------------------------------------------------------------+
|                                      Consulta                                      |
+------------------------------------------------------------------------------------+
| - idConsulta: Long                                                                 |
| - dataHora: LocalDateTime                                                          |
| - status: StatusConsultaEnum                                                       |
| - tipo: TipoConsultaEnum                                                           |
| - linkSalaVirtual: String                                                          |
+------------------------------------------------------------------------------------+
| +confirmar(): Boolean                                                              |
| +cancelar(justificativa: String): Boolean                                          |
+------------------------------------------------------------------------------------+
                  1
                  |
                  | 1
+------------------------------------------------------------------------------------+
|                                ProntuarioEletronico                                |
+------------------------------------------------------------------------------------+
| - idProntuario: Long                                                               |
| - dataCriacao: LocalDateTime                                                       |
| - diagnostico: String                                                              |
| - observacoesClinicas: String                                                      |
| - cid10: String                                                                    |
+------------------------------------------------------------------------------------+
| +adicionarEvolucao(texto: String): Void                                            |
+------------------------------------------------------------------------------------+
                  1
                  |
                  | 0..*
+------------------------------------------------------------------------------------+
|                                 PrescricaoMedica                                   |
+------------------------------------------------------------------------------------+
| - idPrescricao: Long                                                               |
| - medicamentos: String                                                             |
| - posologia: String                                                                |
| - hashAssinaturaICP: String                                                        |
| - qrCodeValidacao: String                                                          |
+------------------------------------------------------------------------------------+
| +assinarDigitalmente(certificado: Byte[]): Boolean                                |
+------------------------------------------------------------------------------------+
```

### 4.3 Diagramas de Sequência

#### 1. Autenticação com MFA
```
Paciente/Médico            Frontend / App              Auth Service             Redis Cache
      |                          |                          |                        |
      |--- 1. Login(user, pass)->|                          |                        |
      |                          |--- 2. POST /auth/login ->|                        |
      |                          |                          |--- 3. Valida Cred. --->|
      |                          |                          |<-- 4. Credenciais OK --|
      |                          |                          |--- 5. Gera OTP --------|
      |                          |<-- 6. Req. Token 2FA ----|                        |
      |<-- 7. Solicita Código ---|                          |                        |
      |--- 8. Insere OTP ------->|                          |                        |
      |                          |--- 9. POST /auth/mfa --->|                        |
      |                          |                          |--- 10. Valida OTP ---->|
      |                          |<-- 11. JWT Auth Token ---|                        |
```

#### 2. Happy Path - Agendamento e Teleconsulta
```
Paciente                 Scheduling Service         Video Gateway            Médico
   |                             |                        |                    |
   |-- 1. Reserva Horário ------>|                        |                    |
   |   (POST /agendamentos)      |                        |                    |
   |<-- 2. Agendamento Criado ---|                        |                    |
   |                             |-- 3. Criar Sala WebRTC|                    |
   |                             |----------------------->|                    |
   |                             |<-- 4. Room Token -----|                    |
   |-- 5. Entra na Sala Virtual ------------------------>|                    |
   |                             |                        |<-- 6. Entra na Sala|
   |==================== SESSÃO DE VÍDEO EM ANDAMENTO =========================|
```

#### 3. Cenário de Erro - Falha na Assinatura ICP-Brasil
```
Médico                    EHR Service               ICP-Brasil Auth API
   |                           |                             |
   |-- 1. Finalizar Receita -->|                             |
   |                           |-- 2. Valida Hash Cert. ---->|
   |                           |<-- 3. ERRO 503 (Indispon.)--|
   |<-- 4. Notifica Erro ------|                             |
   |    "Falha na Assinatura"  |                             |
   |                           |-- 5. Envia p/ Fila de ------> [ RabbitMQ DLQ ]
   |                           |      Retry Assíncrono       |
```

### 4.4 Diagrama de Componentes
```
+----------------------------------------------------------------------------------+
|                                API GATEWAY (Kong)                                |
+----------------------------------------------------------------------------------+
          |                                  |                                  |
          v                                  v                                  v
+------------------+                +------------------+               +------------------+
|  Auth Service    |                | Scheduling Serv. |               |   EHR Service    |
|  (Node.js/TS)    |                |  (Java Spring)   |               |  (Java Spring)   |
+------------------+                +------------------+               +------------------+
          |                                  |                                  |
          v                                  v                                  v
+----------------------------------------------------------------------------------+
|                                BUS DE MENSAGENS                                  |
|                                   (RabbitMQ)                                     |
+----------------------------------------------------------------------------------+
          |                                  |                                  |
          v                                  v                                  v
+------------------+                +------------------+               +------------------+
| Notification Svc |                |  Video Gateway   |               | Audit Service    |
|   (Go Lang)      |                |    (WebRTC)      |               |  (Python/Fast)   |
+------------------+                +------------------+               +------------------+
```

### 4.5 Diagrama de Implantação (Deployment)
```
[ Usuários / Browsers / Apps ]
              |
          (HTTPS / Port 443)
              v
[ Cloudflare WAF / CDN ]
              |
              v
[ AWS Application Load Balancer (ALB) ]
              |
      +-------+-------+
      |               |
      v               v
[ EKS Cluster Node 1 ]  [ EKS Cluster Node 2 ]
  - API Gateway Pods      - API Gateway Pods
  - Microservices Pods    - Microservices Pods
      |               |
      +-------+-------+
              |
      +-------+-----------------------+
      |                               |
      v                               v
[ AWS RDS PostgreSQL (Multi-AZ) ]   [ ElastiCache Redis Cluster ]
```

### 4.6 Diagramas de Atividades

#### Processo Principal: Agendamento e Pagamento de Consulta
```
(*) -> [Selecionar Médico e Horário] -> [Verificar Disponibilidade] 
     -> <Horário Livre?>
        |-- (Não) -> [Exibir Alerta de Indisponibilidade] -> [Selecionar Médico e Horário]
        |-- (Sim) -> [Bloquear Horário Temporariamente (10 min)] 
                  -> [Processar Pagamento] -> <Aprovado?>
                     |-- (Não) -> [Liberar Horário] -> Exibir Erro -> (*)
                     |-- (Sim) -> [Confirmar Agendamento] -> [Notificar Envolvidos] -> (*)
```

#### Processo Administrativo: Credenciamento e Auditoria de Médicos
```
(*) -> [Submeter Documentação e CRM] -> [Validação Automática no CFM]
     -> <CRM Válido e Ativo?>
        |-- (Não) -> [Rejeitar Credenciamento e Notificar] -> (*)
        |-- (Sim) -> [Gerar Perfil de Acesso Médica] -> [Registrar Log de Auditoria LGPD] -> (*)
```

---

## 5. Bloco 5: Modelo de Arquitetura C4

### C4 - Nível 1: Diagrama de Contexto
```
[ Paciente ] ----\
                  +---> [ SISTEMA HOSPITAL CONNECT ] <---> [ Serviços ICP-Brasil ]
[ Médico ] ------/              |                   <---> [ Gateway de Pagamento ]
                                v
                    [ Notificações SMS/Email ]
```

### C4 - Nível 2: Diagrama de Containers
* **Web App (React.js):** Interface SPA para pacientes e médicos.
* **Mobile App (Flutter):** Aplicação nativa para iOS e Android.
* **API Gateway (Kong):** Gerencia roteamento, autenticação, rate limiting e SSL.
* **Services Core (Java/Spring Boot):** Lógica de negócios de agendamento e prontuário.
* **Notification Microservice (Go):** Envio assíncrono de notificações de alta performance.
* **Database (PostgreSQL 16):** Armazenamento relacional transacional.
* **Cache & Locks (Redis):** Cache temporário e controle de concorrência.

---

## 6. Bloco 6: Engenharia de Banco de Dados e Adequação à LGPD

### 6.1 Dicionário de Dados e Classificação PII/LGPD

| Tabela | Coluna | Tipo de Dado | Restrições | Classificação LGPD | Descrição |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `tb_paciente` | `id_paciente` | `BIGSERIAL` | `PK` | Não sensível | Identificador único |
| `tb_paciente` | `cpf` | `VARCHAR(11)` | `UNIQUE, NOT NULL` | **Dado PII** | Documento CPF (Criptografado) |
| `tb_paciente` | `nome` | `VARCHAR(150)` | `NOT NULL` | **Dado PII** | Nome completo do paciente |
| `tb_paciente` | `data_nascimento` | `DATE` | `NOT NULL` | **Dado PII** | Data de nascimento |
| `tb_prontuario` | `diagnostico` | `TEXT` | `NOT NULL` | **Dado Sensível de Saúde** | Evolução clínica do paciente |
| `tb_prontuario` | `cid10` | `VARCHAR(10)` | `NULL` | **Dado Sensível de Saúde** | Código da doença (CID-10) |

### 6.2 Script DDL Executável (PostgreSQL)

```sql
-- Criacao da Tabela de Pacientes
CREATE TABLE tb_paciente (
    id_paciente BIGSERIAL PRIMARY KEY,
    cpf VARCHAR(11) UNIQUE NOT NULL,
    nome VARCHAR(150) NOT NULL,
    data_nascimento DATE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    telefone VARCHAR(20) NOT NULL,
    data_criacao TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Criacao da Tabela de Medicos
CREATE TABLE tb_medico (
    id_medico BIGSERIAL PRIMARY KEY,
    crm VARCHAR(20) NOT NULL,
    uf_crm VARCHAR(2) NOT NULL,
    nome VARCHAR(150) NOT NULL,
    especialidade VARCHAR(80) NOT NULL,
    valor_consulta NUMERIC(10, 2) NOT NULL,
    CONSTRAINT uk_medico_crm_uf UNIQUE (crm, uf_crm)
);

-- Criacao da Tabela de Consultas
CREATE TABLE tb_consulta (
    id_consulta BIGSERIAL PRIMARY KEY,
    id_paciente BIGINT NOT NULL,
    id_medico BIGINT NOT NULL,
    data_hora TIMESTAMP WITH TIME ZONE NOT NULL,
    status VARCHAR(30) NOT NULL,
    tipo VARCHAR(20) NOT NULL,
    link_sala_virtual VARCHAR(255),
    CONSTRAINT fk_consulta_paciente FOREIGN KEY (id_paciente) REFERENCES tb_paciente(id_paciente),
    CONSTRAINT fk_consulta_medico FOREIGN KEY (id_medico) REFERENCES tb_medico(id_medico)
);

-- Criacao da Tabela de Prontuario (LGPD Sensivel)
CREATE TABLE tb_prontuario (
    id_prontuario BIGSERIAL PRIMARY KEY,
    id_consulta BIGINT UNIQUE NOT NULL,
    diagnostico TEXT NOT NULL,
    observacoes_clinicas TEXT,
    cid10 VARCHAR(10),
    data_registro TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_prontuario_consulta FOREIGN KEY (id_consulta) REFERENCES tb_consulta(id_consulta)
);
```

---

## 7. Bloco 7: Planejamento de Projeto, Custos e Matriz de Riscos

### 7.1 Cronograma e Divisão por Sprints
* **Sprint 1 (Semanas 1-2):** Modelagem de requisitos, aprovação das ADRs e setup do ambiente DevOps/CI-CD.
* **Sprint 2 (Semanas 3-4):** Módulo de Autenticação MFA e Cadastros Base.
* **Sprint 3 (Semanas 5-6):** Engine de Agendamento e Integração Redis Lock.
* **Sprint 4 (Semanas 7-8):** Salas Virtuais (WebRTC) e Prontuário Eletrônico (PEP).
* **Sprint 5 (Semanas 9-10):** Assinatura ICP-Brasil, Testes de Carga e Auditoria LGPD.

### 7.2 Estimativa de Custos da Equipe (Pesquisa de Mercado Salarial BR)
*Fonte dos valores: Mediana nacional de salários de tecnologia (Glassdoor).*

| Papel na Equipe | Qtd | Salário Mensal Média | Custo Total / Mês | Custo Total (Projeto 3 Meses) |
| :--- | :--- | :--- | :--- | :--- |
| **Arquiteto de Software Lead** | 1 | R\$ 16.500,00 | R\$ 16.500,00 | R\$ 49.500,00 |
| **Desenvolvedor Senior Backend** | 2 | R\$ 12.000,00 | R\$ 24.000,00 | R\$ 72.000,00 |
| **Desenvolvedor Senior Frontend** | 1 | R\$ 10.500,00 | R\$ 10.500,00 | R\$ 31.500,00 |
| **Engenheiro DevOps / Infra** | 1 | R\$ 11.500,00 | R\$ 11.500,00 | R\$ 34.500,00 |
| **Analista de QA / Testes** | 1 | R\$ 7.500,00 | R\$ 7.500,00 | R\$ 22.500,00 |
| **TOTAL** | **6** | **-** | **R\$ 70.000,00** | **R\$ 210.000,00** |

### 7.3 Matriz de Riscos (8 Riscos Detalhados)

| ID | Descrição do Risco | Categoria | Probabilidade | Impacto | Plano de Mitigação |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **R01** | Indisponibilidade na API externa da ICP-Brasil. | Técnico | Média | Alto | Implementar Pattern Circuit Breaker e fila de retentativa assíncrona no RabbitMQ. |
| **R02** | Perda ou vazamento de dados sensíveis (LGPD). | Segurança | Baixa | Extremo | Encriptação em repouso (AES-256), controle RBAC estrito e logs de auditoria imutáveis. |
| **R03** | Instabilidade de banda durante chamadas WebRTC. | Operacional | Média | Alto | Utilizar servidores TURN/STUN distribuídos com adaptação de bitrate. |
| **R04** | Ultrapassar o orçamento estipulado para a nuvem. | Financeiro | Média | Médio | Configurar alertas de orçamento no AWS Budgets e auto-scaling com limites rígidos. |
| **R05** | Baixa adesão de médicos à interface do Prontuário. | Negócio | Média | Médio | Realizar testes de usabilidade com a persona e otimizar o fluxo para poucos cliques. |
| **R06** | Inconsistência de horários por requisições simultâneas. | Técnico | Média | Alto | Implementar Distributed Locking utilizando Redis Redlock no fluxo de reserva. |
| **R07** | Atraso no envio de lembretes aos pacientes. | Operacional | Baixa | Baixo | Utilizar provedores de mensageria redundantes (Twilio + AWS SNS). |
| **R08** | Alterações regulatórias nas normas do CFM. | Jurídico | Baixa | Alto | Acompanhamento jurídico contínuo e arquitetura parametrizada para regras de negócio. |

---

## 8. Bloco 8: Onboarding do Desenvolvedor (Guia "Dia 1 do Dev")

### 8.1 Pré-requisitos de Máquina
* Docker & Docker Compose instalado
* Java JDK 21
* Node.js v20+ e Yarn
* Git

### 8.2 Guia Passo a Passo de Execução

1. **Clonar o Repositório:**
   ```bash
   git clone https://github.com/hospital-connect/hospital-connect-core.git
   cd hospital-connect-core
   ```

2. **Configurar as Variáveis de Ambiente:**
   Copie o arquivo de exemplo para `.env`:
   ```bash
   cp .env.example .env
   ```

3. **Subir os Serviços de Infraestrutura via Docker:**
   ```bash
   docker-compose up -d db-postgres redis rabbitmq
   ```

4. **Executar as Migrações do Banco de Dados:**
   ```bash
   ./mvnw flyway:migrate
   ```

5. **Iniciar a Aplicação em Modo de Desenvolvimento:**
   ```bash
   ./mvnw spring-boot:run
   ```

6. **Acessar a Documentação Interativa da API (Swagger UI):**
   Acesse no navegador: `http://localhost:8080/swagger-ui.html`

---

## 9. Bloco 9: Governança, Padrões de Código e DoD/DoR

### 9.1 Padrões de Código e Versionamento
* **GitFlow:** Branches organizadas em `main`, `develop`, `feature/*`, `hotfix/*`.
* **Commits Semânticos:** Padrão Conventional Commits (ex: `feat: adiciona assinatura ICP-Brasil`, `fix: corrige timeout de agendamento`).
* **Qualidade de Código:** Análise estática via SonarQube com cobertura mínima exigida de 80% de testes unitários.

### 9.2 Definition of Ready (DoR) - Critérios para Iniciar uma Tarefa
1. História de Usuário escrita no formato *Given/When/Then*.
2. Critérios de aceite definidos e revisados pelo Product Owner (PO).
3. Dependências técnicas e de API identificadas.
4. Estimativa de pontos realizada pela equipe (Story Points).

### 9.3 Definition of Done (DoD) - Critérios para Concluir uma Tarefa
1. Código desenvolvido e seguindo os guias de estilo da equipe.
2. Cobertura de testes unitários e de integração igual ou superior a 80%.
3. Code Review aprovado por pelo menos 2 membros sêniores.
4. Alterações integradas na branch `develop` via pipeline de CI/CD sem falhas.
5. Documentação de API (Swagger) atualizada.
