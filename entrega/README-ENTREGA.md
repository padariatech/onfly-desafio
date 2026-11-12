# Entrega — Consolidador de reservas Onfly

## Defeitos corrigidos

Foram encontrados quatro defeitos no CLI. Cada um está em um arquivo de `src/onfly_report/`.

### 1. Conversão de câmbio invertida

- **Arquivo:** `src/onfly_report/services/currency_converter.py`, função `convert_to_brl`.
- **Causa raiz:** a conversão dividia `amount / rate_to_brl`. A tabela define `rate_to_brl` como quantos reais vale 1 unidade da moeda, então o correto é multiplicar.
- **Correção:** `amount * rate`, com `Decimal` em vez de `float`.

### 2. Noites calculadas pelo dia do mês

- **Arquivo:** `src/onfly_report/services/stay_calculator.py`, função `calculate_room_nights`.
- **Causa raiz:** o cálculo usava `checkout.day - checkin.day`. Isso funciona só dentro do mesmo mês e gera valores negativos na virada de mês (ex.: 30/01 → 02/02 dava -28).
- **Correção:** `(checkout - checkin).days * rooms`, a diferença real entre as datas.

### 3. Filtro de status por lista negativa

- **Arquivo:** `src/onfly_report/services/status_policy.py`, função `is_billable`.
- **Causa raiz:** a regra era `status != "CANCELLED"`, o que deixava `NO_SHOW` entrar nos totais. A regra do README é que só `CONFIRMED` é faturável.
- **Correção:** `status == "CONFIRMED"`. Os contadores por status continuam contando todas as reservas, como o README pede.

### 4. Ranking de hotéis em ordem crescente

- **Arquivo:** `src/onfly_report/services/report_builder.py`, ordenação de `ranked_hotels`.
- **Causa raiz:** `sorted` usa ordem crescente por padrão e não havia inversão. O ranking mostrava os hotéis de menor valor primeiro.
- **Correção:** `key=lambda item: (-item[1]["amount"], item[0])`. Isso ordena por valor decrescente e mantém o desempate por `hotel_id` crescente. Usar só `reverse=True` quebraria esse desempate.

## Como investiguei

1. Rodei `pytest -q` e li cada falha. Cada teste verifica uma regra e mostra o valor obtido.
2. Comparei o código de cada função com a regra correspondente no `README.md`, que é a especificação.
3. Confirmei a direção do câmbio com a tabela `data/taxas_cambio.csv` e o teste de conversão (`100 BRL + 100 USD` a `5.42` deve dar `642.00`).
4. Corrigi cada função de forma isolada e reexecutei os testes a cada mudança.

## Como validar

Requisitos: Python 3.11+.

```bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt

pytest -q
```

Rodar o CLI na amostra de validação e comparar com a tabela "Amostra de validação" do `README.md`:

```bash
PYTHONPATH=src python -m onfly_report \
  --input data/exemplo-validacao.csv \
  --rates data/taxas_cambio.csv \
  --output output/exemplo.json
```

Rodar o relatório completo:

```bash
PYTHONPATH=src python -m onfly_report \
  --input data/reservas_hoteis.csv \
  --rates data/taxas_cambio.csv \
  --output output/report.json
```

## Resultado

- `pytest -q`: **5 passed** (5 testes que antes falhavam, todos passando).
- Amostra de validação: `total_reservations` 12, `confirmed_reservations` 9, `cancelled_reservations` 2, `no_show_reservations` 1, `total_room_nights` 28, `total_amount_brl` 7566.00, `average_nightly_rate_brl` 270.21. Os totais por mês e o top 5 de hotéis batem com a tabela do README.
- Relatório completo (`data/reservas_hoteis.csv`): 300 reservas, 235 confirmadas, 1347 noites, total de R$ 396170.00. Saída com código `0`.
- `ruff check src tests`: sem erros.
