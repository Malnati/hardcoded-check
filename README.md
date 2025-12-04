<!-- README.md -->
# 👁️ Code Literal Sentinel

[![GitHub Release](https://img.shields.io/github/v/release/Malnati/code-literal-sentinel?style=for-the-badge&color=orange)](https://github.com/Malnati/code-literal-sentinel/releases)

**Varredura agressiva de literais hardcoded para indexação e análise assistida por IA.**

O **Code Literal Sentinel** não é um linter tradicional. Ele atua como um "cão farejador" (hunter) que varre agressivamente o diff de Pull Requests em busca de qualquer string, número mágico ou possível segredo hardcoded.

> 🧠 **Filosofia:** Esta Action prioriza **Recall** sobre **Precision**. Ela propositalmente gera muitos falsos positivos ("agressiva") para garantir que nada escape. O arquivo de índice gerado deve ser consumido por uma ferramenta de Inteligência Artificial subsequente, que terá o discernimento (contexto) para filtrar o que é aceitável do que é débito técnico.

---

## 🚀 Funcionalidades

* **🔎 Busca Agressiva:** Utiliza padrões `grep` abrangentes para capturar strings e números.
* **📄 Geração de Índice:** Produz um arquivo físico (`txt`) contendo `Arquivo:Linha:Conteúdo` de todas as ocorrências.
* **🤖 AI-Ready:** O output é formatado para ser facilmente ingerido por prompts de LLMs (Large Language Models) para revisão de código.
* **📡 Diagnóstico JSON:** Retorna um payload JSON rico via `$GITHUB_OUTPUT` para orquestração de workflows.

---

## 📦 Inputs

| Input | Descrição | Padrão |
| :--- | :--- | :--- |
| `token` | **Obrigatório**. Token para acessar o diff da PR. | - |
| `file_extensions` | Regex das extensões de arquivo a serem analisadas. | `ts\|js\|java\|py\|go...` |
| `exclude_patterns` | Regex de pastas/arquivos a ignorar. | `node_modules\|dist\|.git` |
| `output_file` | Nome do arquivo de índice gerado. | `literals-index.txt` |

---

## 📤 Outputs

### 1. `result_json` (Variável de Ambiente)
Um objeto JSON contendo o diagnóstico da execução. Ideal para decidir se deve-se acionar o agente de IA.

```json
{
  "analysis": {
    "status": "FOUND",
    "total_findings": 42,
    "index_file": "literals-index.txt"
  },
  "ui": {
    "message": "Literais detectados.",
    "guidance": "O arquivo de índice contém 42 ocorrências. Envie para o agente de IA.",
    "color": "#dbab09"
  }
}
````

### 2\. `index_path` (Arquivo Físico)

O caminho para o arquivo de texto bruto gerado no workspace. Exemplo de conteúdo:

```text
src/auth/config.ts:15: const SECRET = "my-super-secret-key"
src/utils/calc.py:10: return value * 3.14159
src/views/home.tsx:45: <Button title="Click Me" />
```

-----

## 🛠️ Exemplo de Workflow (Sentinel + AI Analysis)

```yaml
name: "AI Code Review"

on: [pull_request]

permissions:
  contents: read

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0 # Necessário para o git diff

      # 1. O Sentinela coleta as evidências
      - name: Code Literal Sentinel
        id: sentinel
        uses: Malnati/code-literal-sentinel@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          output_file: "audit/literals.txt"

      # 2. Upload do Artefato (Para auditoria ou uso posterior)
      - name: Upload Index
        if: ${{ fromJson(steps.sentinel.outputs.result_json).analysis.status == 'FOUND' }}
        uses: actions/upload-artifact@v4
        with:
          name: raw-literals-index
          path: audit/literals.txt

      # 3. (Conceitual) Passo que enviaria o arquivo para uma IA
      - name: AI Processing
        if: ${{ fromJson(steps.sentinel.outputs.result_json).analysis.status == 'FOUND' }}
        run: |
          echo "Enviando ${{ steps.sentinel.outputs.index_path }} para análise de IA..."
          # ex: python script_ai.py --input audit/literals.txt
```

-----

<div align="center">
<sub>Developed by <a href="https://github.com/Malnati">Ricardo Malnati</a></sub>
</div>

