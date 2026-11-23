# Comparação de Modelos: `chamados-r1` vs `chamados-r2`

## 1. Alteração Realizada

Na branch de revisão, foi corrigido o erro de anotação no conjunto de treinamento em `data/chamados.csv`:

- **Chamado `006`** (`bloqueio de acesso`): rótulo alterado de `comum` para `urgente`, alinhando-se à regra de rotulagem definida no manifesto (*queda, falha crítica e bloqueio são urgentes*).
- **`data/manifesto.json`**: atualizado de `chamados-v1` para `chamados-v2` com o registro da correção.
- **`data/split.csv`**: mantido **inalterado** entre `dados-v1` e `dados-v2`, garantindo avaliação sob a mesma partição de teste fixa.

## 2. Comparação de Métricas

Avaliação realizada no mesmo conjunto de teste fixo (4 exemplos: 2 da classe `comum` e 2 da classe `urgente`).

| Métrica | `chamados-r1` (`modelo-r1` / `chamados-v1`) | `chamados-r2` (`modelo-r2` / `chamados-v2`) | Variação |
| --- | :---: | :---: | :---: |
| **Acurácia** | `0.75` (3/4) | `1.00` (4/4) | `+0.25` |
| **Recall (`urgente`, suporte = 2)** | `0.50` (1/2) | `1.00` (2/2) | `+0.50` |
| **Recall (`comum`, suporte = 2)** | `1.00` (2/2) | `1.00` (2/2) | `0.00` |

### Rastreabilidade e Hashes (SHA-256)

| Artefato / Campo | `chamados-r1` | `chamados-r2` | Alterado? |
| --- | --- | --- | :---: |
| `split_sha256` (`data/split.csv`) | `a543f066e63278bf40d73dcb0cf6b523f949fdeeb2469155c008a9416d0c10f2` | `a543f066e63278bf40d73dcb0cf6b523f949fdeeb2469155c008a9416d0c10f2` | Não |
| `dataset_sha256` (`data/chamados.csv`) | `abbead0c669e9ebb58d5453321061c21eda5db454a60f504e32c2effb10cd52f` | `b80cc51104b3307c504fccf644ec4bd107d14269a0c173ce8a147204d031bd23` | Sim |
| `artifact_sha256` (`models/model.json`) | `72a3a67936139a3473838a6dcfa7aae1acb4c23467be1ef1056d51d485b94840` | `7f9d94ca8dd86ccc8ad90b0839613c61a33e2168bd23bf083a4bbe1375f908ac` | Sim |

## 3. Evidência de Restauração em Clone Limpo (`modelo-r1`)

A restauração da versão original foi validada em um clone limpo (`aula-ml-restauracao`), selecionando a tag `modelo-r1` (`git switch --detach modelo-r1`) e recuperando os artefatos com `dvc pull` sem novo treinamento.

- **Commit verificado (`git rev-parse HEAD`):** `deef0c05eac2f16c3e0e5cf91bdf9e1893374547`
- **Saída da validação de hashes SHA-256:**

```text
OK data/chamados.csv abbead0c669e9ebb58d5453321061c21eda5db454a60f504e32c2effb10cd52f
OK data/split.csv a543f066e63278bf40d73dcb0cf6b523f949fdeeb2469155c008a9416d0c10f2
OK models/model.json 72a3a67936139a3473838a6dcfa7aae1acb4c23467be1ef1056d51d485b94840
```

Todos os hashes conferiram exatamente com os valores registrados em `reports/modelo.json` na versão `chamados-r1`.

## 4. Decisão

- **Critério adotado:** melhorar o *recall* da classe `urgente` sem reduzir o *recall* da classe `comum`, utilizando exatamente a mesma partição de teste.
- **Resultado:** o modelo `chamados-r2` aumentou o *recall* de `urgente` de `0.50` para `1.00`, manteve o *recall* de `comum` em `1.00`, elevou a acurácia geral de `0.75` para `1.00` e preservou o `split_sha256`.
- **Decisão final:** **Aceitar `chamados-r2` (`modelo-r2`)** para este exercício (observando que o conjunto de teste possui 4 exemplos, sendo suficiente para demonstrar o efeito da correção do rótulo neste contexto).
