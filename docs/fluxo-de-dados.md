# Documento de Fluxo de Dados

Este documento detalha os fluxos de dados para as principais integrações do sistema, conforme definido no [ADR-001: Escolha da Arquitetura Híbrida](./../adrs/arquitetura-geral.md). A arquitetura utiliza uma abordagem orientada a eventos para garantir desacoplamento, resiliência e escalabilidade.

## 1. Fluxo de Distribuição de Leads (Montadora -> Concessionária)

Este fluxo descreve como um novo lead capturado no Zoho CRM da montadora é automaticamente distribuído para o CRM de uma concessionária.

**Diagrama de Sequência de Alto Nível:**
*(Nota: O diagrama será adicionado em `/diagramas/fluxo-distribuicao-leads.puml` posteriormente)*

**Passos do Fluxo:**

1. **Origem (Zoho CRM):**

   * Um novo lead é criado ou atribuído à montadora no Zoho CRM.
   * **Gatilho:** Um Webhook configurado no Zoho CRM é acionado pelo evento de criação/atualização do lead.
2. **Gateway de Ingestão (Função Serverless):**

   * O Webhook envia uma requisição HTTP POST para um endpoint de API exposto por uma Função Serverless (ex: AWS Lambda + API Gateway).
   * Esta função é responsável por:
     * **Validação Inicial:** Verifica a autenticidade da requisição (ex: usando um segredo compartilhado).
     * **Transformação Mínima:** Converte o payload do webhook em um formato de evento padronizado (ex: CloudEvents).
     * **Publicação:** Publica o evento `LeadDistribuicaoSolicitada` em um tópico específico no Broker de Mensagens (ex: RabbitMQ).
3. **Broker de Mensagens (RabbitMQ/Kafka):**

   * O evento é recebido e enfileirado de forma durável, garantindo que não seja perdido mesmo que os serviços consumidores estejam temporariamente indisponíveis.
4. **Serviço de Roteamento de Leads (Consumidor):**

   * Um microserviço ou função serverless consumidor assina o tópico `LeadDistribuicaoSolicitada`.
   * Ao receber um evento, este serviço executa a lógica de negócio principal:
     * **Análise e Enriquecimento:** Consulta o banco de dados do portal ou outras fontes para obter as regras de distribuição (ex: por região geográfica, por produto, etc.).
     * **Seleção da Concessionária:** Determina para qual concessionária o lead deve ser enviado.
     * **Transformação de Dados:** Mapeia os campos do lead do formato da montadora para o formato esperado pelo CRM da concessionária de destino.
5. **Destino (CRM da Concessionária):**

   * O Serviço de Roteamento realiza uma chamada de API (REST/SOAP) para o endpoint do CRM da concessionária, enviando os dados do lead.
   * O CRM da concessionária processa a requisição e cria o novo lead.
6. **Atualização de Status (Feedback Loop):**

   * Após a confirmação da criação do lead no CRM da concessionária, o Serviço de Roteamento pode, opcionalmente, publicar um novo evento, como `LeadDistribuidoComSucesso`.
   * Este evento pode ser consumido por outros serviços para, por exemplo, atualizar o status do lead no portal web, notificando que ele foi enviado com sucesso.

## 2. Fluxo de Atualização de Oportunidade (Concessionária -> Montadora)

Este fluxo descreve como uma mudança no estágio de uma oportunidade no CRM da concessionária é refletida de volta no Zoho CRM da montadora.

**Diagrama de Sequência de Alto Nível:**
*(Nota: O diagrama será adicionado em `/diagramas/fluxo-atualizacao-oportunidade.puml` posteriormente)*

**Passos do Fluxo:**

1. **Origem (CRM da Concessionária):**

   * Um vendedor atualiza o estágio de uma oportunidade (ex: de "Qualificação" para "Proposta Apresentada").
   * **Gatilho:** Um Webhook configurado no CRM da concessionária é acionado por este evento.
2. **Gateway de Ingestão (Função Serverless):**

   * O Webhook da concessionária envia uma requisição HTTP POST para um endpoint de API dedicado (similar ao fluxo de leads).
   * A função serverless:
     * **Validação:** Valida a requisição (autenticação via token da concessionária).
     * **Padronização:** Converte o payload em um evento padronizado `OportunidadeAtualizada`.
     * **Publicação:** Publica o evento no Broker de Mensagens.
3. **Broker de Mensagens (RabbitMQ/Kafka):**

   * O evento é enfileirado de forma segura.
4. **Serviço de Sincronização de Oportunidades (Consumidor):**

   * Um microserviço ou função serverless consumidor assina o tópico `OportunidadeAtualizada`.
   * Ao receber o evento, o serviço executa a lógica de sincronização:
     * **Identificação:** Extrai os identificadores do lead/oportunidade e da concessionária.
     * **Mapeamento de Status:** Traduz o status do CRM da concessionária para o status equivalente no Zoho CRM da montadora (ex: "Proposta Apresentada" -> "Proposal/Price Quote").
     * **Formatação:** Prepara a requisição para a API do Zoho CRM.
5. **Destino (Zoho CRM):**

   * O Serviço de Sincronização realiza uma chamada de API para o Zoho CRM, atualizando o estágio da oportunidade correspondente.
6. **Atualização no Portal:**

   * Similarmente ao outro fluxo, um evento `OportunidadeSincronizadaComSucesso` pode ser publicado para que o portal web reflita o novo status em sua interface.
