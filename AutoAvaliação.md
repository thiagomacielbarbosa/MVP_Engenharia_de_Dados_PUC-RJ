# Autoavaliação do MVP

**Aluno:** Thiago Maciel Barbosa

## 1. Atingimento dos Objetivos

O projeto atingiu **100% dos objetivos propostos**. Foi possível construir um pipeline que não apenas ingere os dados, mas resolve problemas complexos da fonte original (cabeçalhos duplos e legendas não estruturadas) sem intervenção manual. As perguntas de negócio foram respondidas com clareza através do modelo Star Schema.

## 2. Principais Desafios

- **Qualidade da Fonte:** O arquivo Excel original possuía formatação visual (linhas de cabeçalho mescladas) que dificultava a leitura automática. Foi necessário usar o Pandas com parâmetros específicos (`header=1`) para resolver.
- **Tipagem de Dados:** Colunas numéricas como `Clutch_Max` continham caracteres ocultos ou formatação de texto, o que inicialmente zerou as médias. O problema foi solucionado alterando a estratégia de cast para `FLOAT` e usando `try_cast`.
- **Legenda Dinâmica:** A tabela de legendas estava "escondida" no meio de um texto livre. Desenvolvi uma lógica em Python para localizar a linha exata onde os dados começavam.

## 3. Próximos Passos

- Implementar orquestração com **Databricks Jobs** (agendamento diário).
- Conectar um **Power BI** diretamente às tabelas Delta para visualização interativa.
