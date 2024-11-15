# Agente com AWS Step Functions

Para criar um agente de autoatendimento de delivery usando **AWS Step Functions**, vamos construir um fluxo que gerencie cada etapa do pedido: desde o recebimento até a entrega. Utilizaremos **AWS Lambda** para processar tarefas específicas em cada etapa, **Amazon DynamoDB** para armazenar dados do pedido e **Amazon SNS** para enviar notificações ao cliente.

## Estrutura do Agente de Autoatendimento de Delivery

### Componentes principais
- **AWS Step Functions**: Orquestra cada etapa do pedido.
- **AWS Lambda**: Cada função executa uma etapa, como validar pedido, confirmar pagamento e atualizar o status.
- **Amazon DynamoDB**: Armazena informações do pedido (ex.: status, itens e informações do cliente).
- **Amazon SNS**: Envia notificações para o cliente em cada etapa.

### Fluxo de Trabalho no AWS Step Functions

O fluxo inclui as seguintes etapas:

1. **Receber Pedido**: Inicia o pedido e salva informações iniciais no DynamoDB.
2. **Validar Estoque**: Verifica a disponibilidade dos itens solicitados.
3. **Confirmar Pagamento**: Valida o pagamento.
4. **Atualizar Status do Pedido**: Atualiza o status no DynamoDB para "Em preparo".
5. **Notificar Cliente**: Usa SNS para enviar uma notificação ao cliente sobre o status.
6. **Preparar Pedido**: Simula o preparo do pedido.
7. **A Caminho**: Atualiza o status para "A caminho" e notifica o cliente.
8. **Entregar Pedido**: Finaliza o pedido, define o status como "Entregue" e notifica o cliente.

### Exemplo de Fluxo no JSON do AWS Step Functions

Este é um exemplo de JSON para o fluxo do AWS Step Functions:

```json
{
  "StartAt": "ReceberPedido",
  "States": {
    "ReceberPedido": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:ReceberPedido",
      "Next": "ValidarEstoque"
    },
    "ValidarEstoque": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:ValidarEstoque",
      "Next": "ConfirmarPagamento"
    },
    "ConfirmarPagamento": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:ConfirmarPagamento",
      "Next": "AtualizarStatusParaEmPreparo"
    },
    "AtualizarStatusParaEmPreparo": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:AtualizarStatusPedido",
      "Parameters": {
        "status": "Em preparo"
      },
      "Next": "NotificarClienteEmPreparo"
    },
    "NotificarClienteEmPreparo": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:NotificarCliente",
      "Parameters": {
        "message": "Seu pedido está em preparo!"
      },
      "Next": "PrepararPedido"
    },
    "PrepararPedido": {
      "Type": "Wait",
      "Seconds": 300,
      "Next": "AtualizarStatusParaACaminho"
    },
    "AtualizarStatusParaACaminho": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:AtualizarStatusPedido",
      "Parameters": {
        "status": "A caminho"
      },
      "Next": "NotificarClienteACaminho"
    },
    "NotificarClienteACaminho": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:NotificarCliente",
      "Parameters": {
        "message": "Seu pedido está a caminho!"
      },
      "Next": "EntregarPedido"
    },
    "EntregarPedido": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:EntregarPedido",
      "Next": "NotificarClienteEntregue"
    },
    "NotificarClienteEntregue": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGION:ACCOUNT_ID:function:NotificarCliente",
      "Parameters": {
        "message": "Seu pedido foi entregue!"
      },
      "End": true
    }
  }
}

Configuração das Funções Lambda

	1.	ReceberPedido: Recebe o pedido, armazena os dados no DynamoDB e inicia o processo.
	2.	ValidarEstoque: Verifica se todos os itens estão disponíveis.
	3.	ConfirmarPagamento: Processa e confirma o pagamento do pedido.
	4.	AtualizarStatusPedido: Atualiza o status do pedido no DynamoDB.
	5.	NotificarCliente: Envia uma notificação via SNS sobre o status atual do pedido.
	6.	EntregarPedido: Finaliza o processo, definindo o status como “Entregue”.

Cada função Lambda pode acessar o DynamoDB e o SNS para atualizar o status e enviar notificações conforme necessário.

Passos para Implementação

	1.	Criar Tabelas no DynamoDB: Configure uma tabela para armazenar o status do pedido, ID do pedido, cliente, e itens.
	2.	Criar Funções Lambda: Implemente as funções para cada etapa.
	3.	Configurar o Step Functions: No console da AWS, crie um novo Step Function e insira o JSON de fluxo.
	4.	Configurar SNS: Configure SNS para enviar notificações ao cliente.

Esse fluxo de trabalho automatizado garante que o cliente seja informado de cada atualização do pedido e que todas as etapas sejam executadas de forma ordenada e gerenciável.

