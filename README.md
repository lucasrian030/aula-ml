# Classificador de Chamados — Versionamento de Dados e Modelos (Git + DVC)

Repositório da Atividade Prática 07 (Engenharia de Sistemas Inteligentes — CK0444 / UFC).  
Este projeto demonstra o versionamento de datasets, partições e modelos com **DVC** vinculados ao histórico do **Git**, incluindo rastreabilidade completa de artefatos e restauração em clone limpo.

## Estrutura do Projeto

- `data/`: descritores DVC do dataset (`chamados.csv.dvc`) e da partição fixa (`split.csv.dvc`), além do manifesto (`manifesto.json`).
- `models/`: descritor DVC do modelo treinado (`model.json.dvc`).
- `src/train.py`: script de treinamento, validação e avaliação do classificador.
- `params.json`: parâmetros de pré-processamento/tokenização.
- `reports/`: métricas (`metrics.json`), manifesto de rastreabilidade do modelo (`modelo.json`) e relatório comparativo (`comparacao.md`).

### Tags Versionadas

| Tag | Descrição |
| --- | --- |
| `dados-v1` | Snapshot inicial dos chamados (com erro de anotação intencional no ID `006`) e partição fixa. |
| `modelo-r1` | Primeiro modelo (`chamados-r1`) treinado sobre `dados-v1`. |
| `dados-v2` | Dataset após a correção do rótulo do ID `006` (`comum` → `urgente`), mantendo `split.csv` fixo. |
| `modelo-r2` | Segundo modelo (`chamados-r2`) treinado sobre `dados-v2`. |

## Pré-requisitos

- **Git**
- **Python 3.10+** (o classificador utiliza apenas a biblioteca padrão do Python; validado em Python 3.13.2)
- **DVC 3.x** (`pip install "dvc>=3,<4"` ou `pipx install "dvc>=3,<4"`)

Verifique as ferramentas instaladas:

```bash
git --version
python3 --version
dvc --version
```

## Configuração de Acesso ao Armazenamento DVC

Os arquivos de dados (`data/chamados.csv`, `data/split.csv`) e o modelo (`models/model.json`) são armazenados fora do Git no remote padrão do DVC chamado `storage`, configurado em `.dvc/config` apontando para a pasta irmã `../dvc-storage`.

Para conferir o remote configurado:

```bash
dvc remote list
```

Caso o armazenamento esteja em outro diretório local (ou em um caminho remoto diferente), ajuste a URL do remote `storage`:

```bash
dvc remote modify storage url /caminho/para/dvc-storage
```

## Restauração de uma Versão Anterior em Clone Limpo

Para recuperar os artefatos exatos de uma versão (por exemplo, `modelo-r1`) sem precisar executar um novo treinamento:

1. Clone o repositório em uma pasta irmã ao diretório `dvc-storage`:

```bash
git clone https://github.com/lucasrian030/aula-ml.git aula-ml-restauracao
cd aula-ml-restauracao
```

2. Selecione a tag desejada (`modelo-r1` ou `modelo-r2`):

```bash
git switch --detach modelo-r1
```

3. Baixe os dados e o modelo correspondentes a partir do armazenamento DVC:

```bash
dvc pull
```

4. Verifique a integridade dos arquivos restaurados comparando seus hashes SHA-256 com o manifesto `reports/modelo.json`:

```bash
python3 - <<'PY'
import hashlib
import json
from pathlib import Path

manifesto = json.loads(Path("reports/modelo.json").read_text(encoding="utf-8"))
for caminho, campo in [
    ("data/chamados.csv", "dataset_sha256"),
    ("data/split.csv", "split_sha256"),
    ("models/model.json", "artifact_sha256"),
]:
    obtido = hashlib.sha256(Path(caminho).read_bytes()).hexdigest()
    if obtido != manifesto[campo]:
        raise SystemExit(f"Hash divergente para {caminho}")
    print(f"OK {caminho} {obtido}")
PY
```

Para confirmar o commit selecionado na restauração:

```bash
git rev-parse HEAD
```
