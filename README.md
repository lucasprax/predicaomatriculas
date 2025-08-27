# Análise Preditiva de Matrículas do Censo Escolar

Este projeto consiste em um script de Machine Learning para analisar os microdados do Censo da Educação Básica do INEP. O objetivo principal é prever o número de matrículas para diferentes etapas de ensino (ex: Creche, Ensino Fundamental, Ensino Médio) com base em dados históricos.

O script foi projetado para ser eficiente em termos de memória e processamento, utilizando técnicas como otimização de tipos de dados e paralelismo massivo para lidar com grandes volumes de dados.

##  Funcionalidades

- **Carregamento de Dados Paralelizado**: Carrega e combina de forma eficiente múltiplos arquivos CSV (um para cada ano do censo) utilizando múltiplos núcleos de CPU.
- **Otimização de Memória**: Reduz drasticamente o uso de memória do DataFrame, permitindo o processamento de dados que, de outra forma, não caberiam na RAM.
- **Pré-processamento Automatizado**: Executa uma pipeline completa de limpeza e preparação de dados, incluindo:
    - Remoção de colunas com alta cardinalidade de nulos.
    - Tratamento de valores ausentes (NaN).
    - Conversão de colunas categóricas para numéricas.
    - Remoção de outliers por *capping*.
    - Eliminação de colunas com variância nula.
- **Engenharia de Features Temporais**: Cria features de *lag* (ano anterior) e taxas de crescimento para capturar tendências temporais, também de forma paralela.
- **Seleção de Features Inteligente**: Utiliza a métrica de **Informação Mútua** para selecionar as features mais relevantes para cada modelo, melhorando o desempenho e a interpretabilidade.
- **Modelagem e Avaliação Paralelizada**:
    - Treina e avalia modelos de regressão para cada etapa de ensino de forma independente e em paralelo.
    - Compara o desempenho de três modelos diferentes: **Regressão Linear**, **Random Forest** e **LightGBM**.
    - Utiliza **Validação Cruzada (K-Fold)** para garantir a robustez dos resultados.
- **Relatório Detalhado**: Gera um resumo do desempenho dos modelos e exporta um arquivo CSV com métricas detalhadas (R², R² Ajustado, MAE, RMSE, etc.) para cada modelo e contexto.

## Fluxo de Execução do Pipeline

O script segue as seguintes etapas:

1.  **Configuração**: O usuário define os parâmetros iniciais no início da função `main()`, como caminhos dos arquivos, anos a serem analisados e se a amostragem será usada.
2.  **Carga e Combinação de Dados**: Os dados brutos dos arquivos `.csv` do Censo são carregados em paralelo.
3.  **Amostragem (Opcional)**: Se ativado, uma fração das escolas é selecionada aleatoriamente para acelerar a execução em ambientes de desenvolvimento.
4.  **Pré-processamento**: O DataFrame combinado passa pela pipeline de limpeza e transformação.
5.  **Engenharia de Features**: As features temporais (lags) são criadas em paralelo.
6.  **Modelagem Paralela**:
    - O script itera sobre cada etapa de ensino (ex: `IN_CRE`, `IN_PRE`, `IN_AI`).
    - Para cada etapa, um processo de trabalho (*worker*) é disparado em um núcleo de CPU separado.
    - O *worker* seleciona as melhores features, treina os 3 modelos e avalia seus desempenhos com validação cruzada.
7.  **Resumo e Exportação**: Os resultados de todos os *workers* são consolidados, um resumo é impresso no console e o relatório completo é salvo em um arquivo CSV.

##  Pré-requisitos

Para executar este projeto, você precisará ter o Python 3.7+ instalado. Todas as bibliotecas necessárias estão listadas no arquivo `requirements.txt`.

##  Instalação e Configuração do Ambiente

Siga estes passos para preparar seu ambiente de desenvolvimento.

1.  **Clone o repositório:**
    ```bash
    git clone [https://github.com/lucasprax/predicaomatriculas.git](https://github.com/lucasprax/predicaomatriculas.git)
    cd predicaomatriculas
    ```

2.  **Crie e ative um ambiente virtual:**
    É uma boa prática isolar as dependências do projeto.
    ```bash
    # Criar o ambiente virtual
    python -m venv venv

    # Ativar no Windows
    venv\Scripts\activate

    # Ativar no macOS/Linux
    source venv/bin/activate
    ```

3.  **Instale as dependências:**
    O arquivo `requirements.txt` contém todas as bibliotecas necessárias.
    ```bash
    pip install -r requirements.txt
    ```
    > **Nota:** Se o arquivo `requirements.txt` não existir, crie-o com o conteúdo abaixo e depois execute o comando `pip install` acima.
    > ```
    > pandas
    > numpy
    > scikit-learn
    > lightgbm
    > tqdm
    > pyarrow
    > jupyter
    > ```

##  Como Usar

Primeiro, prepare os dados e configure o script.

1.  **Baixe os Dados:** Vá ao [portal de microdados do INEP](https://www.gov.br/inep/pt-br/acesso-a-informacao/dados-abertos/microdados/censo-escolar) e faça o download dos arquivos anuais que deseja analisar.

2.  **Organize a Estrutura de Pastas:** Crie uma pasta base e organize os arquivos baixados da seguinte forma:
    ```
    /caminho/base/
    ├── microdados_ed_basica_2020/
    │   └── dados/
    │       └── microdados_ed_basica_2020.csv
    ├── microdados_ed_basica_2021/
    │   └── dados/
    │       └── microdados_ed_basica_2021.csv
    └── ...
    ```

3.  **Configurar o Script**: Abra o arquivo Python e ajuste os parâmetros na seção `main()` de acordo com suas necessidades:

    ```python
    def main():
        # --- PARÂMETROS DE EXECUÇÃO ---
        AMOSTRAGEM_ATIVA = True  # Mude para False para usar todos os dados
        FRACAO_AMOSTRA = 0.1  # 10% das escolas
        CAMINHO_BASE = '/caminho/para/os/microdados' # Ex: '/home/user/dados/censo'
        CAMINHO_SAIDA_CSV = '/caminho/para/salvar/resultados.csv'
        ANOS = range(2011, 2022) # Intervalo de anos a serem analisados
        TOP_N_FEATURES_POR_MODELO = 25 # Número de features a serem selecionadas
        # -----------------------------
    ```

Após a configuração, você pode executar o projeto de duas maneiras:

---

### Opção 1: Executando pelo Terminal

Este método é ideal para automação e para rodar o processo completo de uma só vez.

1.  **Ative o Ambiente Virtual:** Certifique-se de que seu ambiente virtual (`venv`) esteja ativado no terminal.

2.  **Execute o Notebook com `jupyter nbconvert`:**
    Use o comando abaixo para executar todas as células do notebook do início ao fim e salvar os resultados no próprio arquivo.

    ```bash
    jupyter nbconvert --to notebook --execute --inplace predicaomatriculas.ipynb
    ```
    - `--execute`: Executa o notebook.
    - `--inplace`: Salva os outputs (resultados, gráficos) diretamente no arquivo original.

O progresso da execução será exibido no terminal.

---

### Opção 2: Executando no Visual Studio Code

Este método é excelente para desenvolvimento interativo, permitindo executar o código célula por célula.

1.  **Instale as Extensões:**
    - Abra o Visual Studio Code.
    - Vá para a aba de Extensões (ícone de blocos no menu lateral).
    - Instale as extensões **Python** e **Jupyter** da Microsoft.

2.  **Abra a Pasta do Projeto:**
    - No VS Code, vá em `File > Open Folder...` e selecione a pasta do projeto que você clonou.

3.  **Selecione o Interpretador Python Correto:**
    - Pressione `Ctrl+Shift+P` para abrir a paleta de comandos.
    - Digite e selecione `Python: Select Interpreter`.
    - Escolha o interpretador que contém `('.venv')`. Isso garante que o VS Code use as bibliotecas que você instalou no ambiente virtual.

4.  **Abra e Execute o Notebook:**
    - Na árvore de arquivos à esquerda, clique no arquivo `predicaomatriculas.ipynb`.
    - O VS Code abrirá o notebook em uma interface interativa.
    - Para executar o projeto inteiro, clique no botão **"Run All"** (ícone de duas setas) na barra de ferramentas superior do notebook.
    - Você também pode executar cada célula individualmente clicando no ícone de "Play" que aparece à esquerda de cada célula.

Os resultados, logs e gráficos serão exibidos diretamente abaixo de cada célula executada.

## Saída

Ao final da execução, um arquivo CSV (definido pelo parâmetro `CAMINHO_SAIDA_CSV`) será gerado. Este arquivo contém um relatório detalhado com as seguintes colunas:

- `contexto`: A etapa de ensino que foi modelada (ex: `IN_CRE`).
- `modelo`: O modelo de machine learning utilizado (ex: `LightGBM`).
- `R2_Media_CV`: O R² médio obtido na validação cruzada.
- `R2_Ajustado_Media_CV`: O R² ajustado médio, que penaliza o excesso de features.
- `MAE_Media_CV`: O Erro Absoluto Médio.
- `RMSE_Media_CV`: A Raiz do Erro Quadrático Médio.
- `MAPE_Media_CV`: O Erro Percentual Absoluto Médio.
- `..._Std_CV`: O desvio padrão das métricas na validação cruzada.
- `num_amostras`: O número de registros usados para treinar o modelo.
- `num_features`: O número de features selecionadas.
- `features_utilizadas`: A lista das features que o modelo utilizou.

Este arquivo permite uma análise aprofundada para determinar qual modelo funciona melhor para cada etapa de ensino específica.
