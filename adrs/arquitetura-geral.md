# ADR-001: Escolha da Arquitetura Híbrida (Microserviços + Orientada a Eventos/Serverless)

## 1. Status

Aceita
**Data:** 2025-07-22
**Responsável:** Edson L Bezerra Filho

## 2. Contexto

O desafio envolve projetar uma arquitetura de integração entre o Zoho CRM de uma montadora e CRMs de diversas concessionárias. Os requisitos primários são:

* Distribuição automática de leads da montadora para as concessionárias.
* Atualização automática do estágio da oportunidade do CRM da concessionária para o Zoho CRM da montadora.

* Criação de um portal web que permita às concessionárias visualizar leads/oportunidades, gerenciar acessos e tokens, e consultar a documentação das APIs.

**OBS.:** *Os requisitos da vaga indicam familiaridade com Node.js ou Python, bancos de dados relacionais e NoSQL, integração e hospedagem em nuvem (incluindo serverless), autenticação (OAuth2.0, JWT) e ferramentas de mensageria (RabbitMQ, Kafka).*

Há uma forte preferência por uma arquitetura que ofereça desassociação de nuvem/plataforma e otimização de custos sem comprometer a performance.

## 3. Decisão

A arquitetura geral da solução será Híbrida, combinando Microserviços para o Portal Web e uma abordagem Orientada a Eventos com componentes Serverless para os fluxos de integração (distribuição de leads e atualização de oportunidades).

### 3.1. Detalhes da Decisão

#### **Portal Web (Microserviço):**

* Será implementado como um microserviço autônomo.
* **Tecnologias sugeridas:** Python com o framework Django.
* **Funcionalidades Principais:** Servir a interface de usuário para concessionárias, gerenciar autenticação/autorização, exibição de leads e oportunidades, e disponibilização de documentação de APIs.
* **Implantação:** Provavelmente via contêineres (Docker) orquestrados (Kubernetes ou serviço similar na nuvem) para garantir portabilidade e escalabilidade.

#### **Fluxos de Integração (Orientada a Eventos com Serverless):**

* **Captura de Eventos:** Utilização de webhooks ou APIs RESTful nos CRMs de origem (Zoho CRM e CRMs das concessionárias) para disparar eventos de alteração de leads e oportunidades.
* **Ingestão e Publicação em Mensageria:** Estes eventos serão capturados por funções serverless (ex: AWS Lambda, Oracle Functions, ou Azure Functions) que atuarão como gateways de entrada. Essas funções farão uma validação e transformação mínima, e então publicarão uma mensagem formatada em um Broker de Mensagens (ex: RabbitMQ, Kafka).
* **Processamento de Eventos:** Outros microserviços ou funções serverless consumidores serão responsáveis por consumir as mensagens do broker, aplicar a lógica de negócio específica (transformação de dados, validações complexas, roteamento para concessionárias específicas) e, finalmente, atualizar o sistema de destino (Zoho CRM ou CRM da concessionária).
* **Implantação:** Funções serverless gerenciadas pelo provedor de nuvem para a ingestão, e microserviços/funções serverless para os consumidores, permitindo escalabilidade automática e pagamento por uso.

## 4. Alternativas Consideradas

### Arquitetura Baseada Exclusivamente em Microserviços:

**Prós:** Alta modularidade, flexibilidade tecnológica, escalabilidade independente, resiliência.

**Contras:** Maior complexidade operacional para todo o sistema (incluindo gestão de infraestrutura para microserviços de integração), possível latência em comunicações síncronas entre serviços, gerenciamento de consistência de dados distribuídos em cenários transacionais síncronos.

### Arquitetura Exclusivamente Serverless:

**Prós:** Redução de custos operacionais (pay-as-you-go), escalabilidade automática, foco no código.

**Contras:** Potenciais "cold starts" para todas as funcionalidades (incluindo o portal web, que requer resposta rápida), possíveis limitações de recursos, maior vendor lock-in (para a camada de computação serverless), complexidade maior para o desenvolvimento de um portal web interativo.

### Arquitetura Exclusivamente Orientada a Eventos (EDA):

**Prós:** Desacoplamento forte, resiliência natural, escalabilidade horizontal, excelente para fluxos assíncronos.

**Contras:** A complexidade de implementar um portal web interativo e funcional inteiramente baseado em eventos seria excessiva. Rastreamento e depuração podem ser mais complexos sem uma visão síncrona para partes do sistema.

## 5. Justificativa

A decisão pela Arquitetura Híbrida seria a mais adequada por otimizar os pontos fortes das diferentes abordagens, mitigando suas fraquezas no contexto específico do desafio.

### **Melhor Ajuste aos Requisitos:**

O portal web se beneficia da estabilidade e familiaridade de um microserviço tradicional, enquanto os fluxos de integração se alinham perfeitamente com a natureza assíncrona e escalável de eventos e serverless.

### Otimização de Custos e Performance:

O uso de serverless para eventos garante que os custos sejam proporcionais ao uso, ideal para picos e vales de tráfego de leads e atualizações.

O microserviço do portal pode ser dimensionado de forma mais previsível, mantendo uma performance de resposta consistente para os usuários.

### Desassociação de Nuvem/Plataforma:

O microserviço do portal pode ser empacotado em contêineres e implantado em qualquer ambiente compatível com Docker/Kubernetes, garantindo alta portabilidade.

Para a camada de integração, embora as funções serverless sejam específicas da nuvem, a lógica de negócio encapsulada nelas pode ser mais portável. A escolha de um broker de mensagens amplamente adotado (como RabbitMQ ou Kafka) atua como uma camada de abstração crucial, permitindo que os serviços não dependam diretamente de implementações de fila específicas de um provedor de nuvem.

### Resiliência e Escalabilidade:

A natureza assíncrona e o desacoplamento promovidos pela EDA para as integrações aumentam significativamente a resiliência do sistema a falhas temporárias e permitem uma escalabilidade massiva para lidar com grandes volumes de dados.

## 6. Consequências

### Positivas:

**Otimização de Recursos:** Aloca recursos de computação de forma eficiente para cada parte da solução, resultando em um melhor custo-benefício geral.

**Alta Disponibilidade e Resiliência:** Falhas em uma parte do sistema de integração (ex: um consumidor de eventos) não necessariamente interrompem o fluxo de dados em outras partes.

**Flexibilidade para Evolução:** A modularidade dos microserviços e o desacoplamento da EDA facilitam a adição de novas funcionalidades ou a modificação das existentes sem impactar todo o sistema.

**Melhor Experiência do Usuário no Portal:** O microserviço dedicado ao portal pode ser otimizado para responder rapidamente às interações do usuário.

### Negativas:

**Complexidade de Monitoramento:** Rastrear o fluxo de dados através de múltiplos componentes (portal, funções serverless, broker de mensagens, consumidores) exigirá ferramentas robustas de Observabilidade (APM, logs centralizados, tracing distribuído).

**Consistência Eventual:** Para os dados que trafegam via eventos, a consistência será eventual, o que significa que pode haver um pequeno atraso até que as informações sejam completamente propagadas entre todos os sistemas. Isso deve ser comunicado e gerenciado na interface do portal.

**Curva de Aprendizado:** A equipe pode precisar de proficiência tanto em desenvolvimento de microserviços/web quanto em padrões de arquitetura orientada a eventos e serverless.
