<!-- README.md -->
# 👁️ Code Literal Sentinel

**Scanner agressivo de literais. Gera um relatório em branch dedicada para análise de IA.**

Esta Action varre o código modificado em uma PR, identifica strings hardcoded, cria uma nova branch com um relatório Markdown e abre uma **Pull Request dedicada** apontando para a branch de origem. Isso permite que desenvolvedores e agentes de IA revisem as descobertas de forma isolada e estruturada.

## 🚀 Como Funciona

1.  **Scan:** Busca agressiva (grep) por strings e números mágicos.
2.  **Branching:** Cria uma branch `sentinel/report-...` a partir da sua branch de feature.
3.  **Relatório:** Commita o arquivo `reports/code-literal-sentinel/YYYYMMDD.md`.
4.  **Feedback:** Abre uma PR "Sentinel Report" -> "Sua Feature Branch" e retorna o link.

## 📦 Inputs

| Input | Descrição | Padrão |
| :--- | :--- | :--- |
| `token` | **Obrigatório**. Token com permissões `contents: write` e `pull-requests: write`. | - |
| `report_dir` | Diretório onde o relatório será salvo. | `reports/code-literal-sentinel` |
| `file_extensions` | Extensões alvo. | `ts\|js...` |
| `exclude_patterns` | Padrões ignorados. | `node_modules...` |

## 🛠️ Exemplo de Uso

```yaml
name: "Sentinel Scan"

on: [pull_request]

permissions:
  contents: write       # Necessário para criar branch/commit
  pull-requests: write  # Necessário para criar PR e comentar

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # 1. Roda o Sentinel (Gera a PR de relatório)
      - name: Run Sentinel
        id: sentinel
        uses: Malnati/code-literal-sentinel@v2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

      # 2. Comenta na PR original com o Link
      - name: Notify
        uses: Malnati/pr-comment@v6
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          pr_number: ${{ github.event.pull_request.number }}
          message_id: "sentinel-report"
          header_title: "👁️ Code Literal Sentinel"
          header_subject: "Relatório Gerado"
          header_actor: "github-actions[bot]"
          
          body_message: |
            ### ${{ fromJson(steps.sentinel.outputs.result_json).ui.message }}
            
            ${{ fromJson(steps.sentinel.outputs.result_json).ui.guidance }}
          
          footer_result: ${{ fromJson(steps.sentinel.outputs.result_json).analysis.status == 'FOUND' && '⚠️ Revisão' || '✅ Limpo' }}
          footer_advise: "Verifique a PR de relatório gerada."
```

---

### O que acontece no final?

1.  O desenvolvedor abre uma PR `feature/x` -> `develop`.
2.  A Action roda e encontra literais.
3.  Ela cria uma branch `sentinel/report-...` e commita o arquivo MD lá.
4.  Ela abre uma PR `sentinel/report-...` -> `feature/x`.
5.  Ela comenta na PR `feature/x`: **"Literais detectados. Um relatório detalhado foi gerado. [Clique aqui para acessar a PR de Análise]"**.

Isso cria um fluxo de trabalho perfeito para uma ferramenta de IA subsequente pegar essa nova PR, ler o arquivo Markdown e comentar sugestões de refatoração diretamente nele!

<div align="center">
<sub>Developed by <a href="https://github.com/Malnati">Ricardo Malnati</a> 🤍 </sub>
</div>

