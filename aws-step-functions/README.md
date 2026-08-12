# 🍕 Assistente de Delivery Inteligente com AWS Step Functions & Amazon Bedrock

Este repositório contém a implementação de um sistema serverless de orquestração de pedidos de delivery. O projeto combina o poder de gerenciamento de fluxos do **AWS Step Functions** com a inteligência artificial generativa do **Amazon Bedrock** para criar uma experiência de atendimento automatizada, fluida e personalizada.

---

## 🎯 Objetivo do Projeto

O foco principal é estruturar uma infraestrutura escalável que receba o pedido de um cliente em linguagem natural, processe as etapas críticas de negócio (validação, pagamento e logística) e responda de forma humanizada, tudo isso sem gerenciar servidores (100% Serverless).

---

## 🏗️ Arquitetura e Componentes Cadastrados

Abaixo está o fluxo lógico de como as tecnologias se integram dentro da AWS:

```text
[ Cliente ] ──(Texto)──> [ Amazon Bedrock ] ──(JSON Estruturado)──> [ AWS Step Functions ]
                                                                             │
       ┌───────────────────────────────┬─────────────────────────────────────┼──────────────────────────────┐
       ▼                               ▼                                     ▼                              ▼
[ Valida Estoque ]            [ Processa Pagamento ]               [ Notifica Cozinha ]          [ IA Gera Mensagem Final ]
```

### 1. ⚙️ AWS Step Functions (O Orquestrador)
O Step Functions funciona como o "cérebro" operacional do sistema. Ele gerencia a **máquina de estados (State Machine)** que dita o caminho do pedido:
*   **Controle de Fluxo:** Garante que a cozinha só receba o pedido após a aprovação do pagamento.
*   **Tratamento de Erros:** Implementa políticas de *Retry* (tentar novamente) e *Catch* (capturar falhas) automáticas para quedas de gateway de pagamento ou indisponibilidade de estoque.
*   **Integração Nativa:** Conecta-se diretamente ao Bedrock e ao AWS Lambda sem necessidade de escrever código de infraestrutura complexo.

### 2. 🧠 Amazon Bedrock (A Inteligência)
O Bedrock é responsável por trazer a camada de IA Generativa para o delivery:
*   **Triagem Inicial:** Interpreta o texto livre do cliente (ex: *"Quero uma pizza de calabresa e um refri zero"*) e o transforma em dados estruturados (JSON) com os itens e quantidades.
*   **Personalização de Mensagens:** Gera notificações customizadas para o cliente de acordo com o status atual do pedido, adaptando o tom de voz da marca.

---

## 📄 Exemplo Prático: Fluxo em Amazon States Language (ASL)

O trecho de código abaixo demonstra como o Step Functions invoca nativamente o modelo **Amazon Nova** (ou Claude) no Bedrock para processar o texto do usuário:

```json
{
  "StartAt": "InterpretarPedidoBedrock",
  "States": {
    "InterpretarPedidoBedrock": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Parameters": {
        "ModelId": "amazon.nova-micro-v1:0",
        "Body": {
          "prompt": "Você é o atendente de um delivery. Extraia os itens do seguinte pedido em formato JSON estrito: 'Me manda dois x-burguers e uma batata frita.'",
          "max_tokens": 200
        }
      },
      "Next": "ProcessarPagamento"
    },
    "ProcessarPagamento": {
      "Type": "Pass",
      "End": true
    }
  }
}
```

---

## 🚀 Vantagens desta Abordagem

*   **Custo Eficiente:** Por ser um modelo *Pay-per-use*, você só paga quando houver pedidos trafegando pelo sistema.
*   **Manutenção Reduzida:** Menos funções Lambda para gerenciar, já que o Step Functions faz a integração direta com o Bedrock.
*   **Experiência do Usuário Elevada:** Chatbots tradicionais seguem caminhos rígidos. Com o Bedrock, o cliente pode se expressar livremente.

---

## 🛠️ Como Executar este Projeto

1.  **Pré-requisitos:**
    *   Conta ativa na AWS.
    *   Acesso liberado aos modelos do **Amazon Bedrock** (solicite no console da AWS na região `us-east-1` ou de sua preferência).
    *   AWS CLI configurado localmente.
2.  **Deploy:**
    *   Clone este repositório: `git clone https://github.com`
    *   Utilize o AWS SAM ou CloudFormation para subir a infraestrutura:
        ```bash
        sam deploy --guided
        ```
3.  **Testes:**
    *   Acesse o console do AWS Step Functions.
    *   Inicie uma nova execução passando um texto de pedido como entrada.

---

## 👨🏻‍💻 Contato

<p style="padding-top:5px">
	<img src="https://avatars.githubusercontent.com/u/52001930?s=400&u=fb999c966c5c652a8357cbede4b1112e79cbfe18&v=4" alt="avatar" style="width:96px;height:96px;border-radius:50%;object-fit:cover;">

<p>&nbsp&nbsp&nbsp Wagner Andrade<br>
    &nbsp&nbsp&nbsp
    <a href="https://github.com/wsawebmaster">
      <img src="https://img.icons8.com/ios-glyphs/28/github.png" alt="GitHub" width="28" height="28" style="vertical-align:middle;" />&nbsp;<strong>GitHub</strong>
    </a>&nbsp;|&nbsp;
    <a href="https://www.linkedin.com/in/wsawebmaster">
      <img src="https://img.icons8.com/color/28/linkedin.png" alt="LinkedIn" width="28" height="28" style="vertical-align:middle;" />&nbsp;<strong>LinkedIn</strong>
    </a>&nbsp;|&nbsp;
    <a href="mailto:wsawebmaster@yahoo.com.br">
      <img src="https://img.icons8.com/ios-glyphs/28/secured-letter.png" alt="Email" width="28" height="28" style="vertical-align:middle;" />&nbsp;<strong>Email</strong>
    </a>
</p>

