# Documento de Estratégia de Segurança

Este documento descreve as medidas de segurança para a autenticação e autorização no sistema, com foco no Portal Web e na integração de APIs, conforme o [ADR-001](./adrs/arquitetura-geral.md).

## 1. Autenticação no Portal Web

**SEC-001: Autenticação de Usuários com JWT**

*   **Mecanismo:** A autenticação dos usuários no portal web será baseada em JSON Web Tokens (JWT).
*   **Fluxo:**
    1.  O usuário envia suas credenciais (e-mail e senha) para um endpoint de login (`/api/auth/login`).
    2.  O serviço de autenticação valida as credenciais no banco de dados.
    3.  Se as credenciais forem válidas, o serviço gera um **Access Token** (JWT de curta duração, ex: 15 minutos) e um **Refresh Token** (de longa duração, ex: 7 dias).
    4.  O Access Token é enviado de volta ao cliente e armazenado em memória (ex: state de uma SPA). Ele será incluído no cabeçalho `Authorization` de cada requisição para acessar rotas protegidas.
    5.  O Refresh Token é armazenado de forma segura no cliente (ex: em um cookie HTTP-Only) e usado para obter um novo Access Token quando o antigo expirar, sem exigir que o usuário faça login novamente.

## 2. Autorização no Portal Web

**SEC-002: Controle de Acesso Baseado em Papel (RBAC)**

*   **Mecanismo:** A autorização será implementada usando um sistema de papéis (Roles).
*   **Implementação:**
    *   O payload do JWT (Access Token) conterá as informações do papel do usuário (ex: `"role": "admin_concessionaria"`).
    *   O backend (API do portal) terá um middleware que decodifica o JWT a cada requisição e verifica se o papel do usuário tem permissão para acessar o recurso solicitado.
    *   Isso garante que um usuário padrão não possa acessar rotas de administração, como a gestão de tokens de API.

## 3. Autenticação das Integrações (Webhooks)

**SEC-003: Autenticação de Webhooks com Tokens de API**

*   **Mecanismo:** As chamadas de webhook provenientes dos CRMs (tanto da montadora quanto das concessionárias) devem ser autenticadas para garantir que apenas fontes legítimas possam enviar dados para a plataforma.
*   **Implementação:**
    1.  **Geração do Token:** Um administrador da concessionária gera um token de API único e secreto através do portal web.
    2.  **Configuração do Webhook:** Ao configurar o webhook no CRM da concessionária, este token é incluído em um cabeçalho HTTP customizado (ex: `X-API-Key`).
    3.  **Validação no Gateway:** O Gateway de Ingestão (Função Serverless) que recebe a chamada do webhook é responsável por:
        *   Extrair o token do cabeçalho.
        *   Validar o token contra um banco de dados seguro (ex: Hashicorp Vault ou um serviço de segredos da nuvem como AWS Secrets Manager).
        *   Se o token for válido e ativo, a requisição é processada. Caso contrário, é rejeitada com um status `401 Unauthorized`.

**SEC-004: Proteção Adicional com Segredos Compartilhados (Opcional)**

*   Para webhooks que suportam, como o do Zoho, um segredo compartilhado pode ser usado para verificar a assinatura da carga útil (payload), garantindo que o corpo da requisição não foi alterado em trânsito (verificação de integridade).

## 4. Segurança Geral

**SEC-005: Comunicação Segura com HTTPS**
*   Toda a comunicação entre clientes, portais e APIs deve ser obrigatoriamente feita sobre HTTPS (TLS) para criptografar os dados em trânsito.

**SEC-006: Armazenamento de Senhas**
*   As senhas dos usuários no banco de dados do portal devem ser armazenadas usando um algoritmo de hashing forte e com salt (ex: bcrypt ou Argon2).
