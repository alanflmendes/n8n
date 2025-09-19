📄 README – Automação de E-mails com Anexos no n8n


Este workflow foi desenvolvido no n8n para automatizar o tratamento de e-mails recebidos via Gmail.
Ele executa diferentes ações dependendo da presença ou não de anexos:

📎 Se houver anexo:

Processa e padroniza os arquivos.

Faz upload para uma pasta específica no Google Drive.

Consulta a cotação do dólar em uma API pública.

Envia e-mail de confirmação ao remetente, incluindo a taxa atual.

⚠️ Se não houver anexo:

Envia automaticamente uma mensagem solicitando reenvio com o arquivo.

🛠 Fluxo do Processo
Estrutura Geral
📥 Gmail Trigger
        │
        ▼
🔀 Gateway: Anexos ─── (false) ──▶ 📧 E-mail: Anexo não localizado
        │
       (true)
        ▼
🧩 Nodo de integração ─▶ ☁️ Upload Google Drive ─▶ 🌐 HTTP Request ─▶ 📧 E-mail: Concluído

🔩 Detalhes dos Nós
1. Gmail Trigger (Entrada)

Função: Dispara o fluxo quando chega um novo e-mail.

Configuração:

Baixa os anexos automaticamente (downloadAttachments: true).

Salva-os com prefixo anexo (dataPropertyAttachmentsPrefixName).

Observação: Puxa também informações do remetente para respostas automáticas.

2. Gateway: Anexos (Decisão)

Função: Verifica se existe conteúdo em {{$binary.anexo}}.

Encaminhamentos:

✅ True: Segue para processar anexos.

❌ False: Vai direto para o envio do e-mail de alerta (“Anexo não localizado”).

3. Nodo de Integração (Code Node)

Função: Percorre todos os binários (arquivos anexados), normaliza-os e renomeia para anexo.

Exemplo de código:

for (const item of items) {
  if (item.binary) {
    for (const key of Object.keys(item.binary)) {
      results.push({
        json: { fileName: item.binary[key].filename },
        binary: { anexo: item.binary[key] }
      });
    }
  }
}
return results;


Benefício: Facilita o uso do mesmo campo “anexo” nos próximos nós.

4. Google Drive: Upload

Função: Faz upload dos anexos para a pasta definida no Drive.

Entrada esperada: binary.anexo (do passo anterior).

Destino configurado: Pasta uploadArquivos (com ID fixo).

5. HTTP Request

Função: Chamada externa para API de câmbio.

Endpoint: https://open.er-api.com/v6/latest/USD

Resultado usado: {{ $json.rates.BRL }} para cotação USD → BRL.

6. Gmail: Mensagem de Conclusão

Função: Confirma sucesso da automação.

Mensagem enviada ao remetente:

Olá, [Nome do remetente]!

Confirmamos que o processo foi concluído com sucesso 🎉
Cotação atual: 1 USD = R$ [valor da API]

Obrigado!


7. Gmail: Anexo não localizado

Função: Retorna ao remetente caso não haja anexos.

Mensagem enviada:

Olá, [Nome do remetente]!

Recebemos sua solicitação, mas não havia nenhum anexo.
Por favor, reenvie com o arquivo.

Obrigado!


📂 Pré-requisitos

Conta Gmail com OAuth2 configurado.

Conta Google Drive com OAuth2 e permissão na pasta de destino.

Workflow ativo no n8n.

O ambiente está sendo executado via Docker
