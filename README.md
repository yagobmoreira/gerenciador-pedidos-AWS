# Projeto Gerenciador de Pedidos - AWS 

![Arquitetura do Projeto](https://github.com/yagobmoreira/gerenciador-pedidos-AWS/blob/main/arquitetura-proposta.png)

# Projeto de Processamento de Pedidos na AWS

## 📌 Apresentação do Projeto

Este projeto tem como objetivo a construção de uma arquitetura
**serverless na AWS** para processamento de pedidos.\
Foram utilizadas diversas integrações entre serviços como **API Gateway,
Lambda, SQS, DynamoDB, EventBridge, S3 e SNS**, com foco em
escalabilidade, tolerância a falhas e automação do fluxo de dados.

### 📅 Linha do tempo de desenvolvimento

#### Dia 1 - Entrada e pré-validação de pedidos 

-   Criação do **API Gateway** para receber pedidos via endpoint REST
    (`/pedidos`).
-   Implementação da **Lambda de pré-validação** que insere mensagens em
    uma **fila SQS FIFO**.
-   Configuração da **fila SQS FIFO** com **DLQ** para garantir
    resiliência em caso de falhas.
-   Implementação da **Lambda de validação de pedidos**, consumindo da
    fila FIFO e publicando eventos no **EventBridge Bus**.

#### Dia 2 - Integração com arquivos JSON no S3 

-   Criação do **bucket S3** (`datalake-arquivos-yagomoreira`) para
    armazenar arquivos JSON.
-   Configuração de **evento de criação de objetos** no bucket,
    filtrando arquivos `.json`, que dispara mensagens para uma **fila
    SQS**.
-   Implementação da **Lambda de validação S3**, que:
    -   Registra histórico no **DynamoDB**
        (`controle-arquivos-historico-yagomoreira`).
    -   Publica notificações de erro em um **tópico SNS**.
    -   Encaminha pedidos válidos para a fila SQS FIFO de pedidos.

#### Dia 3 - Processamento de pedidos validados 

-   Criação da **fila de pedidos pendentes (SQS)** com DLQ.
-   Implementação da **Lambda de processamento**, que insere e atualiza
    registros no **DynamoDB** (`pedidos-db-yagomoreira`).
-   Regra no **EventBridge** que direciona eventos de novos pedidos
    validados para a fila de processamento.

#### Dia 4 - Operações de alteração e cancelamento de pedidos 

-   Criação das filas **SQS para alteração e cancelamento de pedidos**,
    cada uma com sua respectiva **DLQ**.
-   Implementação das **Lambdas de alteração e cancelamento**, que
    atualizam registros no DynamoDB.
-   Configuração de **regras no EventBridge** para rotear eventos de
    alteração/cancelamento para suas filas correspondentes.

------------------------------------------------------------------------

## 🏗️ Arquitetura construída

A arquitetura resultante é composta pelos seguintes elementos
principais:

-   **API Gateway** → Entrada de pedidos (HTTP POST).\
-   **Lambda Pre-Validacao** → Pré-processamento e envio para fila
    FIFO.\
-   **SQS FIFO** → Garantia de ordem no processamento dos pedidos.\
-   **Lambda ValidacaoPedidos** → Validação e envio de eventos para
    EventBridge.\
-   **EventBridge Bus** → Barramento central para orquestração de
    eventos.\
-   **SQS PedidosPendentes** → Buffer de pedidos validados, com DLQ.\
-   **Lambda ProcessaPedidos** → Inserção/atualização de dados no
    DynamoDB (tabela de pedidos).\
-   **S3 Data Lake** → Armazenamento de arquivos JSON de pedidos.\
-   **SQS S3ArquivosQueue** → Recebe notificações de criação de objetos
    no S3.\
-   **Lambda ValidacaoS3Arquivos** → Validação dos arquivos JSON,
    registro em histórico e envio de mensagens.\
-   **DynamoDB ControleArquivos** → Registro de histórico de arquivos
    processados.\
-   **SNS NotificacaoErroArquivos** → Notificação de erros em arquivos
    inválidos.\
-   **SQS AlteraPedidosQueue e CancelaPedidosQueue** → Filas dedicadas
    para operações específicas, cada uma com DLQ.\
-   **Lambdas AlteraPedido e CancelaPedido** → Consumo das filas de
    alteração/cancelamento, com atualização no DynamoDB.

------------------------------------------------------------------------

## 🤔 Reflexão sobre o aprendizado

A construção desta arquitetura permitiu compreender, na prática, como
**serviços serverless da AWS** podem ser integrados para criar soluções
escaláveis e resilientes.\
O aprendizado envolveu não apenas o uso individual de cada serviço, mas
principalmente o entendimento da **orquestração de eventos** com
EventBridge e SQS, bem como a **resiliência** garantida por DLQs e
monitoramento de erros via SNS.\
Esse projeto reforçou a importância de **arquiteturas orientadas a
eventos** no desenvolvimento de sistemas modernos e de fácil evolução.
