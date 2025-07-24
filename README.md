# Bem-vindo à Documentação de Arquitetura CCM

Esta documentação detalha a arquitetura do sistema de integração de CRMs, cobrindo desde as decisões de alto nível até os fluxos de dados e segurança.

Use a navegação lateral para explorar os diferentes tópicos:

*   **Registros de Decisão de Arquitetura (ADRs):** Entenda as decisões chave e suas justificativas.
*   **Documentos de Detalhamento:** Aprofunde-se em aspectos específicos como fluxos de dados, funcionalidades e segurança.
*   **Diagramas C4 Model:** Visualize a arquitetura em diferentes níveis de abstração (Contexto, Contêineres, Componentes).
*   **Diagramas de Sequência:** Compreenda os fluxos de interação entre os componentes do sistema.

## Como Visualizar a Documentação Localmente

Para visualizar esta documentação em seu navegador, siga os passos abaixo:

1.  **Certifique-se de ter o Java instalado** em sua máquina, pois o PlantUML (utilizado para gerar os diagramas) requer Java.
2.  **Ative o ambiente virtual** do projeto:
    ```bash
    source venv/bin/activate
    ```
3.  **Instale as dependências do MkDocs** (se ainda não o fez):
    ```bash
    pip install mkdocs mkdocs-material mkdocs-plantuml
    ```
4.  **Inicie o servidor de desenvolvimento do MkDocs**:
    ```bash
    mkdocs serve
    ```
    Isso iniciará um servidor local e fornecerá um link (geralmente `http://127.0.0.1:8000`) que você pode abrir em seu navegador.

## Contribuição

Para contribuir com a documentação, edite os arquivos Markdown (`.md`) e PlantUML (`.puml`) nas pastas `adrs/`, `docs/` e `diagramas/`. As alterações serão refletidas automaticamente ao rodar `mkdocs serve`.
