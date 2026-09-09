# Documentação Técnica e Fiscal: Devolução de Mercadorias com ICMS-ST (SP)

Este documento orienta o desenvolvimento e a parametrização das regras de emissão de NF-e para operações de **Devolução de Compra com Substituição Tributária (ICMS-ST)** promovidas por nossa empresa (Contribuinte Substituído) no estado de São Paulo.

---

## 🚨 1. Regra Crítica de Validação do XML (`vOutro`)

Para que o valor total da nota fiscal de devolução coincida com o valor financeiro originalmente pago ao fornecedor (composto pelo produto + o ICMS-ST retido na compra), o sistema **não deve preencher as tags de imposto ST próprias**. 

O valor do ICMS-ST deve ser adicionado ao campo de **Outras Despesas Acessórias**, evitando a **Rejeição 610 da SEFAZ** (*Total da NF-e difere do somatório dos valores que compõem o valor total*).

*   **Tag do Item:** `vOutro` (id: I17a) $\rightarrow$ Preencher com o valor proporcional do ICMS-ST do item.
*   **Tag do Total da Nota:** `vOutro` (id: W15) $\rightarrow$ Somatória dos valores de `vOutro` de todos os itens.
*   **Campos do Grupo de Imposto ST (`vBCST` e `vICMSST`):** Devem permanecer zerados/não preenchidos.

---

## 📊 2. Matriz de Cenários e Parametrização

O sistema deve identificar o perfil fiscal do **Destinatário (Fornecedor Original)** para aplicar uma das duas regras abaixo:

### Cenário A: Destinatário é Substituído (Ex: Distribuidor / Revenda)
*Operação onde o fornecedor não é o fabricante e também recebeu a mercadoria com o imposto retido.*

*   **CFOP:** `5.411` (Operação Interna) ou `6.411` (Operação Interestadual).
*   **CST ICMS:** **`060`** (ICMS cobrado anteriormente por substituição tributária).
*   **ICMS Próprio (`vBC`, `pICMS`, `vICMS`):** Zerado (R$ 0,00).
*   **Tag `vOutro` (Item e Total):** Inserir o valor do ICMS-ST proporcional do item.
*   **Texto em Informações Complementares (`infCpl`):**
    > *"Devolução [parcial/total] de mercadoria recebida com a NF-e nº [X], de [DATA]. BC ICMS-ST Retido: R$ [X] | Valor ICMS-ST Retido: R$ [X]. Imposto Recolhido por Substituição - Artigo 274 do RICMS/SP. Motivo: [MOTIVO]."*

### Cenário B: Destinatário é Substituto RPA (Ex: Indústria / Fabricante)
*Operação onde o fornecedor é o próprio fabricante/importador responsável pela retenção original do imposto.*

*   **CFOP:** `5.411` (Operação Interna) ou `6.411` (Operação Interestadual).
*   **CST ICMS:** **`000`** (Tributada integralmente).
    *   *Nota: **Não usar o CST `010`**, pois a devolução não está gerando uma nova retenção de ST na cadeia.*
*   **ICMS Próprio (`vBC`, `pICMS`, `vICMS`):** **Destacar normalmente**. O sistema deve replicar exatamente a alíquota e a base de cálculo proporcional extraídas dos itens da NF-e de compra original para devolver o crédito ao fabricante.
*   **Tag `vOutro` (Item e Total):** Inserir o valor do ICMS-ST proporcional do item.
*   **Texto em Informações Complementares (`infCpl`):**
    > *"BC ICMS-ST Retido na origem: R$ [X] | Valor ICMS-ST Retido na origem: R$ [X]. Devolução [parcial/total] conforme NF-e de origem nº [X]. Motivo: [MOTIVO]."*

---

## ⚖️ 3. Base Legal (São Paulo)

A estrutura acima atende estritamente às exigências do Regulamento do ICMS do Estado de São Paulo e entendimentos consolidados do Fisco paulista:

1.  **Legislação Básica:**
    *   **RICMS-SP/2000:** Artigo 4º, inciso IV; Artigo 127, § 15; Artigo 274, *caput*.
    *   **Decisão Normativa CAT nº 04/2010 (Itens 3 e 4):** Dispõe sobre os procedimentos e preenchimento de documentos fiscais relativos ao desfazimento da substituição tributária por devolução de mercadoria.
2.  **Respostas a Consultas Fiscais (Sefaz-SP):**
    *   As manifestações oficiais do Estado (ex: **RC 31442/2025, RC 29407/2024 e RC 26707/2022**) chancelam o uso do campo `vOutro` para o valor do ICMS-ST em devoluções feitas por substituídos, determinando que o imposto retido não seja preenchido nas tags próprias de imposto para evitar conflitos na validação do arquivo XML.
