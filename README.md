# Trajeto

Planejador de viagens independente. Monta roteiros e compara voos e hospedagens pelo **custo total** e pelo **tempo real de deslocamento**, sem resultados patrocinados. Cada valor exibido informa a origem: tempo real, cache, estimativa, a confirmar ou exemplo.

## O que já funciona

| Recurso | Fonte | Chave necessária |
|---|---|---|
| Destino (geocodificação) | OpenStreetMap Nominatim | Não |
| Atrações e horários de funcionamento por data | OpenStreetMap Overpass + biblioteca `opening_hours` | Não |
| Câmbio oficial | PTAX – Banco Central do Brasil | Não |
| Voos (preço final, conexões, bagagem, regras) | Duffel Flights API | `DUFFEL_ACCESS_TOKEN` |
| Hospedagem (valor total, cobranças na acomodação, nota) | Duffel Stays API | `DUFFEL_ACCESS_TOKEN` (Stays habilitado na conta) |
| Tempos reais a pé | OpenRouteService Matrix | `ORS_API_KEY` |
| Mapa | Tiles do OpenStreetMap via Leaflet | Não |

Sem chaves, o site funciona com atrações, horários e câmbio reais; voos e hospedagem aparecem como "integração não configurada". O **modo demonstração** preenche voos e hotéis com valores ilustrativos, marcados como *Exemplo*.

Motor de decisão (`public/js/engine.js`):
- seleção de atrações por interesse e relevância;
- agrupamento geográfico por dia (k-means balanceado);
- alocação dos grupos às datas evitando dias de fechamento;
- sequência por vizinho mais próximo com refinamento 2-opt;
- simulação de horários com janelas de funcionamento, almoço e limite de retorno;
- custo efetivo de cada hospedagem com o roteiro recalculado a partir do endereço;
- custo generalizado dos voos (preço final + tempo × valor do tempo + risco de conexão);
- ranking com pesos ajustáveis; cenários comparativos; orçamento por categoria.

Também inclui: salvar no navegador, link de compartilhamento (`/?r=id`), calendário `.ics`, impressão/PDF e registro de todas as consultas a fornecedores em `data/quotes.jsonl` (auditoria de origem e neutralidade).

## Rodar localmente

Requer Node.js 22.9 ou superior.

```bash
npm install
cp .env.example .env      # preencha as chaves que tiver
npm start                 # http://localhost:3000
npm run demo              # força o modo demonstração
npm test                  # 19 testes do motor, adaptadores e API
```

## Obter as chaves

1. **Duffel**: crie uma conta em duffel.com, gere um *access token* no painel. Comece com o token de teste (`duffel_test_...`), que retorna dados de companhias de teste. Para preços reais, ative o modo live na conta e, para hotéis, solicite o acesso ao Stays.
2. **OpenRouteService**: cadastre-se em openrouteservice.org e gere uma chave gratuita.

Confirme as condições comerciais e os limites de uso de cada fornecedor antes de publicar.

## Publicar

O projeto é um único serviço Node que serve a API e o site. Exemplos:

**Render / Railway** (mais simples)
1. Suba esta pasta para um repositório no GitHub.
2. Crie um *Web Service* apontando para o repositório.
3. Build: `npm install` · Start: `npm start`.
4. Cadastre `DUFFEL_ACCESS_TOKEN`, `ORS_API_KEY` e `HTTP_USER_AGENT` nas variáveis de ambiente.
5. Para manter os roteiros compartilhados entre reinícios, anexe um disco persistente e defina `DATA_DIR` para ele.

**Docker** (qualquer nuvem)
```bash
docker build -t trajeto .
docker run -p 3000:3000 --env-file .env -v trajeto-data:/app/data trajeto
```

## Antes de abrir ao público

- **Nominatim e Overpass públicos** têm política de uso justo: defina `HTTP_USER_AGENT` com um contato e, com tráfego relevante, use instâncias próprias ou um provedor pago (variáveis `NOMINATIM_URL` e `OVERPASS_URL`).
- **Tiles do OpenStreetMap**: para tráfego alto, contrate um provedor de tiles e troque a URL em `public/js/app.js`.
- **Cache e roteiros** ficam em memória e em arquivos. Com mais de uma instância, troque por Redis e PostgreSQL (as interfaces estão isoladas em `server/lib/core.js` e nas rotas de `/api/itineraries`).
- **Reserva**: as ofertas da Duffel podem ser compradas pela própria API (pedidos e pagamentos), etapa ainda não implementada. Hoje o site direciona a busca para fontes neutras.

## Estrutura

```
server/
  index.js            API, segurança, limites de uso, roteiros salvos
  lib/core.js         HTTP com timeout, cache com validade, registro de cotações
  providers/          duffel.js · osm.js · routing.js · fx.js · demo.js
public/
  index.html          interface
  js/engine.js        motor de decisão (puro, testado)
  js/app.js           telas, mapa, cenários, exportações
test/                 testes com node:test
```

## Próximas etapas sugeridas

1. Datas flexíveis (±3 dias) e aeroportos alternativos automáticos na busca de voos.
2. Tarifas detalhadas da hospedagem escolhida (cancelamento e café da manhã confirmados).
3. Multicidades com otimização da ordem das cidades.
4. Contas de usuário e roteiros na nuvem.
5. Alertas de variação de preço por e-mail.
