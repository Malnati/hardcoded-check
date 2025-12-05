<!-- README.md -->
<div align="center">

![Code Literal Sentinel Banner](https://github.com/Malnati/code-literal-sentinel/assets/PLACEHOLDER_PARA_SUA_IMAGEM.jpg)

# 👁️ Code Literal Sentinel

[![GitHub Release](https://img.shields.io/github/v/release/Malnati/code-literal-sentinel?style=for-the-badge&color=orange)](https://github.com/Malnati/code-literal-sentinel/releases)
[![License](https://img.shields.io/github/license/Malnati/code-literal-sentinel?style=for-the-badge&color=blue)](LICENSE)

**O "Cão Farejador" para sua Governança de Código.**
*Varredura agressiva, relatórios isolados e preparação para análise de IA.*

</div>

---

## 📖 Sobre

O **Code Literal Sentinel** não é um linter comum. Ele foi desenhado com uma filosofia de **Recall > Precision**: ele prefere encontrar *tudo* o que parece suspeito (strings, números mágicos, segredos em potencial) do que deixar algo passar.

Em vez de poluir sua Pull Request com centenas de comentários, ele:
1.  Gera um relatório consolidado.
2.  Cria uma **Branch de Auditoria** isolada.
3.  Abre uma **Pull Request dedicada** contendo apenas as evidências encontradas.

Isso permite que desenvolvedores (ou Agentes de IA) revisem o "barulho" separadamente, sem bloquear o fluxo principal de desenvolvimento.

### ✨ Principais Recursos
* **🛡️ Anti-Loop & Idempotência:** Mecanismos avançados (Circuit Breaker e Content Signature) impedem que a Action entre em loop infinito ou gere relatórios duplicados.
* **🔎 Busca Profunda:** Detecta literais hardcoded em arquivos modificados.
* **📦 Zero Poluição:** Não toca na branch da feature. Todo o lixo é jogado numa branch `sentinel/audit/...`.
* **🤖 AI Ready:** Produz saídas estruturadas (JSON e Markdown) perfeitas para ingestão por LLMs.

---

## 🔐 Configuração de Segurança (Obrigatório)

> [!IMPORTANT]
> Para que o Sentinela possa criar a PR de relatório, você **DEVE** conceder permissão ao GitHub Actions.

1.  No seu repositório, vá em **Settings**.
2.  Na barra lateral esquerda: **Actions** -> **General**.
3.  Role até o final, na seção **Workflow permissions**.
4.  Marque a opção: **Allow GitHub Actions to create and approve pull requests**.
5.  Clique em **Save**.



---

## ⚙️ Inputs

| Input | Descrição | Padrão | Obrigatório |
| :--- | :--- | :--- | :---: |
| `token` | Token para operações Git (`secrets.GITHUB_TOKEN`). | - | **Sim** |
| `file_extensions` | Regex das extensões a serem analisadas. | `ts\|js\|jsx\|tsx\|java\|py...` | Não |
| `exclude_patterns` | Regex de caminhos/arquivos a ignorar. | `node_modules\|dist\|build...` | Não |

> **Nota:** O diretório de saída é gerenciado internamente pela Action para garantir a consistência da auditoria.

---

## 📤 Outputs

A action fornece um JSON rico na variável `result_json`, ideal para integrações (ex: Slack, Teams ou Comentários de PR).

```json
{
  "analysis": {
    "status": "FOUND",            // FOUND, CLEAN, SKIPPED, PREVIOUSLY_AUDITED
    "total_findings": 42,         // Quantidade de linhas suspeitas
    "report_pr_url": "https..."   // Link direto para a PR de auditoria
  },
  "ui": {
    "message": "Literais detectados (42).",
    "guidance": "Relatório disponível em: [Ver PR de Auditoria](...)",
    "color": "#dbab09"
  }
}
````

-----

## 🛠️ Workflow Recomendado

Copie e cole este workflow para ter uma auditoria completa com notificação visual.

```yaml
name: "Sentinel Audit"

on:
  pull_request:
    types: [opened, synchronize, reopened]
    # Opcional: Ignorar branches do próprio sentinel para economizar recursos
    # branches-ignore: ['sentinel/**']

permissions:
  contents: write       # Necessário para criar branch de relatório
  pull-requests: write  # Necessário para abrir a PR e comentar

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        # Nota: O Sentinel gerencia o fetch-depth automaticamente se necessário.

      # 1. Executa a Varredura e Gera a PR de Relatório
      - name: Run Sentinel
        id: sentinel
        uses: Malnati/code-literal-sentinel@v2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # file_extensions: "js|ts|py" # Customize se necessário

      # 2. Notifica na PR original (Usando Malnati/pr-comment)
      - name: Notify User
        uses: Malnati/pr-comment@v6
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          pr_number: ${{ github.event.pull_request.number }}
          message_id: "sentinel-audit-report" # ID para Sticky Comment
          
          header_title: "👁️ Code Literal Sentinel"
          header_subject: "Auditoria de Qualidade"
          header_actor: "github-actions[bot]"
          
          # Usa os outputs dinâmicos do Sentinel
          body_message: |
            ### ${{ fromJson(steps.sentinel.outputs.result_json).ui.message }}
            
            ${{ fromJson(steps.sentinel.outputs.result_json).ui.guidance }}
          
          # Define o status visual
          footer_result: ${{ fromJson(steps.sentinel.outputs.result_json).analysis.status == 'FOUND' && '⚠️ Revisão Sugerida' || '✅ Código Limpo' }}
          footer_advise: "Verifique o relatório para detalhes."
```

-----

<div align="center">
<sub>Developed with ❤️ and ☕ by <a href="https://github.com/Malnati">Ricardo Malnati</a></sub>
</div>
