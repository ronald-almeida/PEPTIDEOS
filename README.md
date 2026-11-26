# Referência em Peptídeos — checkout Pix

Checkout responsivo baseado na referência fornecida, sem cupom e com Pix como único pagamento. Inclui geração de cobrança, QR Code, Pix Copia e Cola, recuperação após recarregar e página de obrigado após confirmação no servidor.

- Produto: **COMBO BLACK REFERÊNCIA EM PEPTÍDEOS**.
- Autor: Thiago Bechara dos Santos.
- Preço: **R$ 10.000,00**, observado no checkout original em 26/11/2026; definido no servidor em `lib/checkout.js`.
- Prazo provisório: **até 2 dias**. `ACCESS_RELEASE_HOURS=2` troca para até 2 horas, tanto no checkout quanto no obrigado.
- Suporte reutilizado da Black Visionária: https://wa.me/5575988393742.
- Reembolso: https://reembolso-de-alunos.vercel.app/.

## Executar

Requer Node.js 24, sem dependências externas.

```sh
npm run dev
npm test
npm run build
```

O servidor local abre em `http://127.0.0.1:4173`. Copie `.env.example` para `.env.local` e preencha as variáveis para usar a integração real. Sem configuração, a página renderiza com a geração de pagamento desativada. Nunca publique `.env.local`.

## Configuração na Vercel

Importe `ronald-almeida/PEPTIDEOS`, use Node.js 24 e preserve `vercel.json`. Framework: Other; diretório público: `public`; build: `node scripts/check.mjs`. O arquivo `api/index.js` é a função de servidor e os endpoints são direcionados por rewrite.

| Variável | Uso |
| --- | --- |
| `NEXUM_CHAT_ID` | Identificador da mesma conta Nexum da Black Visionária, conforme solicitado. |
| `NEXUM_API_CREDENTIAL` | Credencial fixa da aba Integração da Nexum. Manter como segredo de produção. |
| `NEXUM_ORIGIN` | Origem autorizada pela Nexum; padrão `https://nexumpaybot.com`. |
| `CHECKOUT_SESSION_SECRET` | Segredo novo e exclusivo deste projeto, com pelo menos 32 caracteres. |
| `WHATSAPP_URL` | Opcional; padrão `https://wa.me/5575988393742`. |
| `ACCESS_RELEASE_HOURS` | `2` para duas horas ou `48` para dois dias; padrão 48. |

As credenciais existentes da Black Visionária são do tipo Secret e não podem ser lidas de volta pela API da Vercel. Cadastre os mesmos valores diretamente no novo projeto; não exponha credenciais no GitHub nem no navegador. Não coloque credenciais de produção em previews de teste.

Geração local de um segredo exclusivo:

```sh
node -e "console.log(require('node:crypto').randomBytes(32).toString('hex'))"
```

## Fluxo e limites

1. O navegador envia dados validados para `/api/order`. O servidor fixa o valor e salva uma sessão criptografada em cookie HttpOnly, SameSite=Strict e Secure em HTTPS.
2. `/api/pix` usa a credencial somente no servidor. Repetições preservam a chave de idempotência e o corpo da mesma cobrança, inclusive após timeout.
3. A confirmação usa `/api/status` a cada 30 segundos, respeitando `Retry-After`. Transação e valor são verificados antes de aceitar pagamento.
4. `/obrigado` consulta `/api/receipt`; parâmetros na URL não simulam confirmação.

Não há banco local: a Nexum mantém o registro financeiro e a sessão permite recuperar a cobrança no mesmo navegador por até sete dias. Em outro dispositivo, ou após apagar cookies, o suporte precisa localizar a compra na Nexum. A deduplicação em memória é uma proteção adicional por instância; não é um limitador distribuído contra clientes maliciosos. Antes de escalar tráfego, configure rate limiting na hospedagem.

Esta implementação **não entrega o curso nem envia emails automaticamente**. A equipe deve liberar o acesso e enviar as orientações dentro do prazo anunciado. Não existe webhook de entrega cadastrado, pois não foi fornecido um sistema de alunos; o checkout confirma pelo endpoint de status documentado. O celular é opcional, não é armazenado e não é enviado à Nexum. A API documenta CPF; CNPJ não é solicitado.

## Verificação

Os testes usam respostas simuladas para validar dados, idempotência, timeout, estados, valor do pagamento, sessão, bloqueio de recibo falso e disponibilidade sem credenciais. Nenhuma cobrança real é criada pelos testes.

```sh
node --test --test-isolation=none test/checkout.test.mjs
node test/browser-fixture.mjs
```

O segundo comando abre uma **prévia de teste** em `http://127.0.0.1:4174`, sem conexão com a Nexum. `/fixture/paid` simula confirmação e `/fixture/pending` volta ao estado pendente. Esses endpoints existem apenas no servidor de teste, fora da função e da pasta pública de produção.

## Referências

- Checkout: https://pay.hotmart.com/F108411640M
- API: https://app.nexumpaybot.com/api/index.html
- Capa, ícone Pix e ilustração QR obtidos da referência fornecida. O QR de pagamento é retornado pela Nexum, nunca reutilizado de outra compra.
- Cabeçalho e rodapé identificam o checkout e a processadora atuais. Não são reutilizados selo de proteção, termos ou notificações de vendas da Hotmart.
