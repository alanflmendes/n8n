README – Automação de E-mails com Anexos no n8n

1. Visão Geral

Este workflow implementa uma automação no n8n para tratamento de e-mails recebidos via Gmail.
A automação diferencia e-mails com anexos de e-mails sem anexos, executando rotinas distintas para cada caso.

E-mails com anexo:

Processamento e normalização dos arquivos.

Upload dos anexos em uma pasta do Google Drive.

Consulta de cotação do dólar via API externa.

Envio de e-mail de confirmação ao remetente, incluindo a cotação.

E-mails sem anexo:

Envio de aviso automático solicitando reenvio com o arquivo.

2. Fluxo do Processo
Gmail Trigger
      │
      ▼
IF "Gateway: Anexos" ─── (False) ──▶ Gmail: Anexo não localizado
       │
      (True)
       ▼
Code: Integração ─▶ Google Drive: Upload ─▶ HTTP Request ─▶ Gmail: Concluído

3. Detalhamento dos Nós
3.1 Gmail Trigger

Dispara a automação ao receber novos e-mails.

Configurações principais:

downloadAttachments: true

dataPropertyAttachmentsPrefixName: "anexo"

Também captura remetente para respostas automáticas.

3.2 IF – Gateway: Anexos

Avalia se existe conteúdo em {{$binary.anexo}}.

Encaminhamentos:

True: segue para processamento dos anexos.

False: envia resposta de “Anexo não localizado”.

3.3 Code – Integração

Itera sobre os anexos recebidos, normalizando-os para o campo binary.anexo.

Adiciona no JSON o nome de cada arquivo (json.fileName).

3.4 Google Drive – Upload

Recebe os binários de anexos e realiza upload em uma pasta definida do Drive.

Pasta configurada: uploadArquivos (ID fixo).

3.5 HTTP Request

Executa chamada para API de câmbio.

Endpoint utilizado: https://open.er-api.com/v6/latest/USD.

O campo rates.BRL é usado na resposta final ao remetente.

3.6 Gmail – Mensagem de Conclusão

Envia confirmação ao remetente.

Inclui a informação de cotação obtida na etapa anterior.

3.7 Gmail – Mensagem: Anexo não localizado

Responde automaticamente ao remetente informando ausência de anexo.

Solicita reenvio do e-mail com o arquivo.

4. Pré-requisitos

Conta Gmail com autenticação OAuth2 configurada no n8n.

Conta Google Drive com autenticação OAuth2 configurada.

Workflow ativo e credenciais validadas.

O ambiente foi executado via Docker
