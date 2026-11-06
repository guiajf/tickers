## Classificação de ativos de renda variável

À medida em que incorporamos diferentes ativos à nossa hipotética carteira de investimentos, novos desafios se impõem à tarefa de classificação e enquadramento tributário.

Embora o Relatório Consolidado fornecido pela **B3** contenha informações segmentadas por classe de ativo, o extrato de negociações não permite aplicar filtro categórico. 

Os códigos de negociação - *tickers* - em geral formados por quatro letras seguidas de um ou dois números, em alguns casos são incapazes de fornecer pistas inequívocas para segmentação do ativo.

*Tickers* terminados em números de 1 a 10, indicam ações; terminados de 32 a 35 ou 39, indicam **BDR** - certificados de depósito de valores mobiliários; 

*Tickers* que terminam em **11** são empregados para designar diferentes categorias:

. *Units*: pacote composto por ações ON e PN;

. *FII*: Fundos de Investimento Imobiliário;

. *FIAGRO*: Fundos de Investimento nas Cadeias Produtivas Agroindustriais;

. *FIP-IE*: Fundos de Investimento em Participações - Infraestrutura;

. *FI-Infra*: Fundos de Investimento em Infraestrutura;

. *ETF*: Exchange Trade Fund - conhecidos como fundos de índices.


**Por que é importante saber?**

Porque o regime de tributação varia em função da classe do ativo.

Os ganhos líquidos em operações efetuadas com **Ações** e com **Ouro** (ativo financeiro) são isentos, se o total das vendas de cada ativo, realizadas no mês (considera-se o mês inteiro, por exemplo: janeiro, fevereiro etc.), for menor ou igual a R$ 20.000,00.

Ocorrendo venda, no mesmo mês, de **Ações** e de **Ouro** (ativo financeiro), os limites de isenção citados acima aplicam-se separadamente a cada modalidade de ativo.

**Prejuízos** podem ser compensados no mesmo mês ou nos meses seguintes, inclusive em operações com **BDR**s ou **ETF**s de ações e **cripto**. 

**Units** são *pacotes* que reúnem diferentes tipos de ações da mesma empresa em um único código. Geralmente, misturam Ações Ordinárias (com direito a voto) e Ações Preferenciais (com preferência em dividendos). Exemplos: SANB11 (Santander) e TAEE11 (Taesa).

A Receita Federal orienta que **Units** não se enquadram na isenção dos R$ 20.000 mensais aplicável às ações.  Qualquer lucro obtido na venda de Units deve ser tributado. A alíquota é de **15%** para **operações comuns** (swing trade) e 20% para operações day trade.  **O imposto não é retido na fonte**, cabendo ao investidor calcular o ganho líquido e **gerar o DARF até o último dia útil** do mês seguinte à venda.

**FII**s são isentos de IR sobre os rendimentos, mas sofrem incidência de alíquota de 20% sobre o ganho líquido na venda das cotas. O pagamento deve ser feito pelo investidor, por meio de DARF, até o último dia do mês seguinte. Não existe isenção para vendas mensais abaixo de R$ 20 mil, todo lucro na venda das cotas é tributado.

**FIAGRO**s também são isentos de IR sobre os rendimentos, mas sofrem incidência de alíquota de 20% sobre o ganho líquido na venda das cotas. O pagamento deve ser feito pelo investidor, por meio de DARF, até o último dia do mês seguinte. Não existe isenção para vendas mensais abaixo de R$ 20 mil, todo lucro na venda das cotas é tributado. Os prejuízos em **FII** e **FIAGRO** podem ser compensados entre si.

**FIP-IE** e **FI-Infra** também são isentos de IR sobre os rendimentos e possuem alíquota de 0% sobre o ganho de capital, para os residentes que operam no mercado de renda variável.

No caso de **ETF**s de renda variável, incide alíquota de 15% sobre o 
lucro obtido na venda de um ativo. Prejuízos podem ser compensados no mesmo mês ou nos meses seguintes, inclusive em operações com ações ou ETFs de ações e cripto. Já no caso de ETFs de fundos imobiliários, incide alíquota de 20% 
sobre o lucro obtido na venda de um ativo. Prejuízos podem ser compensados no mesmo mês ou nos meses seguintes, com lucros de ETFs da mesma classe (FII) ou
cotas dos próprios fundos imobiliários. Não contam com nenhuma faixa de isenção, como no caso das ações. O DARF deve ser recolhido pelo investidor até o último dia útil do mês seguinte ao da apuração de resultados.

Então, é importante saber identificar corretamente a classe de ativo, cujo *ticker* termina em **11**, por causa dos mecanismos de compensação de prejuízo e dos limites de isenção para vendas mensais abaixo de R$ 20 mil, que se aplicam em alguns casos e em outros não.

Além disso, algumas classes obedecem a regras específicas:
No caso de **BDR**s de renda variável, incide alíquota de 15% sobre o lucro obtido na venda de um ativo. Prejuízos podem ser compensados no mesmo mês ou nos meses seguintes, inclusive em operações com ações ou ETFs de ações e cripto. O DARF deve ser recolhido pelo investidor até o último dia útil do mês seguinte ao da apuração de resultados. Não contam com nenhuma faixa de isenção, como no caso das ações.

No caso dos *tickers* terminados em **1**, incide alíquota de 15% sobre o valor líquido da venda dos direitos de subscrição. Não está incluída no limite de isenção de venda até R$ 20 mil, como no caso das ações. O DARF deve ser recolhido pelo investidor até o último dia útil do mês seguinte ao da apuração de resultados.

Para automatizar a classificação dos ativos de renda variável, desenvolvemos uma rotina em Python, compartilhada a seguir.


### Importamos as bibliotecas


```python
import pandas as pd
import requests
import json
import numpy as np
import re
import os
from datetime import datetime
```

## Carregamos os arquivos e criamos o data frame

No nosso caso, o arquivo csv contém o extrato de negociação tratado previamente.


```python
# Função específica para limpar o formato brasileiro dos arquivos
def limpar_arquivo_b3(df):
    for col in ['Preço', 'Valor']:
        # Garante que é string para aplicar as substituições
        df[col] = df[col].astype(str)
        # Remove 'R$' e espaços
        df[col] = df[col].str.replace('R$', '', regex=False)
        df[col] = df[col].str.strip()
        # Remove pontos de milhar
        df[col] = df[col].str.replace('.', '', regex=False)
        # Substitui vírgula decimal por ponto
        df[col] = df[col].str.replace(',', '.', regex=False)
        # Remove qualquer caractere não numérico restante
        df[col] = df[col].apply(lambda x: re.sub(r'[^\d.-]', '', x))
        # Converte para float
        df[col] = pd.to_numeric(df[col], errors='coerce').fillna(0.0)
    return df

# Carrega arquivo de 2024 (formato americano)
df_2024 = pd.read_csv("b3.csv", 
                     sep=";", 
                     usecols=['Data', 'Operação', 'Mercado', 'Instituição', 
                             'Ativo', 'Quantidade', 'Preço', 'Valor'])

# Carrega e limpa arquivo de 2025
df_2025 = pd.read_csv("b3_122025.csv", sep=";")
df_2025 = limpar_arquivo_b3(df_2025)

# Carrega e limpa arquivo de 2026
df_2026 = pd.read_csv("b3_092026.csv", sep=";")
df_2026 = limpar_arquivo_b3(df_2026)

# Concatena todos os anos
df = pd.concat([df_2024, df_2025, df_2026], ignore_index=True)

# ==============================================================================
# Padronização dos Tickers (Remoção de 'F' fracionário)
# ==============================================================================
# 1. str.replace(r'F$', '', regex=True): Remove o 'F' APENAS se estiver no final da string
# 2. str.upper(): Garante que todos os tickers fiquem em maiúsculo para o merge com a API/JSON
df['Ativo'] = df['Ativo'].astype(str).str.replace(r'F$', '', regex=True).str.upper()

# Converte data (com errors='coerce' para evitar falhas em datas inválidas)
df["Data"] = pd.to_datetime(df["Data"], format='%d/%m/%Y', errors='coerce')

# Exibir as primeiras linhas do DataFrame resultante (ignorando as 10 últimas para visualização)
display(df.head(-10))

```


<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Data</th>
      <th>Operação</th>
      <th>Mercado</th>
      <th>Instituição</th>
      <th>Ativo</th>
      <th>Quantidade</th>
      <th>Preço</th>
      <th>Valor</th>
      <th>Prazo/Vencimento</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-12-30</td>
      <td>Compra</td>
      <td>Oferta Pública</td>
      <td>CAIXA ECONOMICA FEDERAL</td>
      <td>CXAG11</td>
      <td>31</td>
      <td>103.70</td>
      <td>3214.70</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-12-20</td>
      <td>Compra</td>
      <td>Mercado à Vista</td>
      <td>BANCO BTG PACTUAL S/A</td>
      <td>B3SA3</td>
      <td>400</td>
      <td>11.72</td>
      <td>4688.00</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-12-09</td>
      <td>Compra</td>
      <td>Mercado à Vista</td>
      <td>BANCO BTG PACTUAL S/A</td>
      <td>VALE3</td>
      <td>300</td>
      <td>75.31</td>
      <td>22593.00</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-12-01</td>
      <td>Compra</td>
      <td>Mercado à Vista</td>
      <td>BANCO BTG PACTUAL S/A</td>
      <td>ABCB4</td>
      <td>1</td>
      <td>17.33</td>
      <td>17.33</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-12-01</td>
      <td>Compra</td>
      <td>Mercado à Vista</td>
      <td>BANCO BTG PACTUAL S/A</td>
      <td>ABCB4</td>
      <td>11</td>
      <td>17.20</td>
      <td>189.20</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>5390</th>
      <td>2026-02-03</td>
      <td>Compra</td>
      <td>Mercado à Vista</td>
      <td>SAFRA DISTRIBUIDORA DE TITULOS E VALORES MOBIL...</td>
      <td>BBDC4</td>
      <td>2000</td>
      <td>22.02</td>
      <td>44040.00</td>
      <td>-</td>
    </tr>
    <tr>
      <th>5391</th>
      <td>2026-02-02</td>
      <td>Compra</td>
      <td>Mercado à Vista</td>
      <td>SAFRA DISTRIBUIDORA DE TITULOS E VALORES MOBIL...</td>
      <td>BBDC4</td>
      <td>4100</td>
      <td>21.52</td>
      <td>88232.00</td>
      <td>02/02/2026</td>
    </tr>
    <tr>
      <th>5392</th>
      <td>2026-01-28</td>
      <td>Venda</td>
      <td>Mercado à Vista</td>
      <td>SAFRA DISTRIBUIDORA DE TITULOS E VALORES MOBIL...</td>
      <td>POMO4</td>
      <td>10400</td>
      <td>6.67</td>
      <td>69368.00</td>
      <td>-</td>
    </tr>
    <tr>
      <th>5393</th>
      <td>2026-01-28</td>
      <td>Venda</td>
      <td>Mercado à Vista</td>
      <td>SAFRA DISTRIBUIDORA DE TITULOS E VALORES MOBIL...</td>
      <td>POMO4</td>
      <td>600</td>
      <td>6.66</td>
      <td>3996.00</td>
      <td>-</td>
    </tr>
    <tr>
      <th>5394</th>
      <td>2026-01-26</td>
      <td>Compra</td>
      <td>Mercado à Vista</td>
      <td>SAFRA DISTRIBUIDORA DE TITULOS E VALORES MOBIL...</td>
      <td>POMO4</td>
      <td>3000</td>
      <td>6.37</td>
      <td>19110.00</td>
      <td>-</td>
    </tr>
  </tbody>
</table>
<p>5395 rows × 9 columns</p>
</div>


### Realizamos a classificação via API Brapi

Na primeira versão, acessamos o arquivo público com a relação dos produtos listados na B3, até então disponibilizado diariamente no endereço https://www.b3.com.br/pt_br/market-data-e-indices/servicos-de-dados/market-data/consultas/boletim-diario/dados-publicos-de-produtos-listados-e-de-balcao/.

Infelizmente, o arquivo foi descontinuado a partir de 15/12/2025.

Alternativamente, implementamos a **API** da [Brapi](https://brapi.dev/) como fonte primária.


```python
url = "https://brapi.dev/api/quote/list"
response = requests.get(url).json()
df_api = pd.DataFrame(response['stocks'])

def classificar_via_api(row):
    ticker = str(row.get('stock', '')).upper().strip()
    name = str(row.get('name', '')).upper()
    sub_type = str(row.get('subType', '')).lower()
    
    categoria = 'Não Classificado'
    descricao = 'Verificar regra tributária'
    
    if sub_type == 'fii':
        categoria, descricao = 'FII', '20% GC | Sem isenção | Compensa com FIIs'
    elif sub_type == 'fi-agro':
        categoria, descricao = 'FII', '20% GC | Sem isenção | Compensa com FIAGROs'
    elif sub_type == 'etf':
        categoria = 'ETF'
        if any(word in name for word in ['IMOBILIÁRIO', 'FII', 'IFIX', 'REAL ESTATE', 'FIAGRO', 'CRI']):
            descricao = '20% GC | Sem isenção | Compensa com FIIs/ETFs FII'
        else:
            descricao = '15% GC | Sem isenção | Compensa com Ações/ETFs/Cripto'
            
    # ATUALIZAÇÃO: FI-Infra para residentes tem isenção total (0%) conforme Lei 12.431/2011, Art. 3º
    elif sub_type == 'fi-infra':
        categoria = 'ETF' # Mantido em ETF para compatibilidade com o script dirpf_2026
        descricao = '0% GC (Alienação/Resgate) | Isento | Compensa com FI-Infra (Lei 12.431/2011, Art. 3º)'
        
    elif sub_type == 'unit':
        categoria, descricao = 'UNIT', '15% GC (Swing) / 20% (Day) | Sem isenção | Compensa com Units'
    elif sub_type == 'bdr':
        categoria, descricao = 'BDR', '15% GC | Sem isenção | Compensa com Ações/ETFs/Cripto'
        
    # NOVA CLASSE: FIP (inclui FIP-IE)
    elif sub_type == 'fip':
        # NOTA: Adicione 'FIP' à lista CATEGORIAS_INVESTIMENTO no seu script dirpf_2026.py.
        # Se preferir NÃO alterar o script dirpf_2026, mude a linha abaixo para: categoria = 'ETF'
        categoria = 'FIP' 
        descricao = '15% GC (FIP Comum) ou 0% na Alienação (FIP-IE) | Sem isenção 20k | DISTINGUIR MANUALMENTE'
        
    elif sub_type == 'stock':
        categoria = 'Ação'
        if 'DIR' in name or 'DIREITO' in name or (ticker.endswith('1') and len(ticker) == 5):
            descricao = '15% s/ Valor Líquido | Sem isenção | Regra específica (Direito)'
        else:
            descricao = '15% GC | Isento até R$ 20k/mês | Compensa com Ações'
            
    return pd.Series([categoria, descricao])

df_api[['Categoria', 'Descrição']] = df_api.apply(classificar_via_api, axis=1)
df_categorias_api = df_api[['stock', 'Categoria', 'Descrição']].dropna()
df_categorias_api.columns = ['Ativo', 'Categoria', 'Descrição']
df_categorias_api['Ativo'] = df_categorias_api['Ativo'].str.upper()

```

### Executamos fallback para corrigir falhas

Carregamos o JSON do [Luciano Ayres](https://github.com/lucianoayres/brazil-stocks-tickers-json) como fallback para garantir 100% de cobertura.



```python
# Identifica quais ativos do seu df_base original ainda não foram classificados
ativos_nao_classificados = df[~df['Ativo'].isin(df_categorias_api['Ativo'])]['Ativo'].unique()

if len(ativos_nao_classificados) > 0:
    print(f"⚠️ {len(ativos_nao_classificados)} ativos não classificados pela API. Acionando fallback JSON...")
    
    try:
        with open('tickers-cash-market.json', 'r', encoding='utf-8') as file:
            json_data = json.load(file)
            
        # Função extraída do seu notebook original
        def extract_tickers(json_data, target_tickers):
            tickers_found = {}
            for key, value in json_data['data'].items():
                cod_neg = value.get('codNeg', '').upper()
                nome_curto = value.get('nomeCurto', '').upper()
                espec_papel = value.get('especPapel', '').upper()
                
                if cod_neg not in target_tickers:
                    continue
                    
                categoria_json = 'Ação' # Default
                if espec_papel.startswith('DIR'):
                    categoria_json = 'Direitos de subscrição'
                elif espec_papel.startswith('UNT'):
                    categoria_json = 'UNIT'
                elif len(cod_neg) >= 2 and cod_neg[-2:].isdigit() and (32 <= int(cod_neg[-2:]) <= 35 or cod_neg.endswith('39')):
                    categoria_json = 'BDR'
                elif 'FII' in nome_curto:
                    categoria_json = 'FII'
                elif 'FIAGRO' in nome_curto:
                    categoria_json = 'Fiagro'
                elif 'FIP' in nome_curto or 'FIP-IE' in nome_curto:
                    categoria_json = 'FIP'
                elif any(word in nome_curto for word in ['ISHARES', 'INVESTO', 'ETF']):
                    categoria_json = 'ETF'
                elif len(cod_neg) >= 2 and not cod_neg[-1:].isdigit() and cod_neg.endswith('F'):
                    categoria_json = 'Ação' # Fração
                    
                tickers_found[cod_neg] = categoria_json
            return tickers_found

        ticker_dict_fallback = extract_tickers(json_data, ativos_nao_classificados)
        
        # Mapeia as categorias do JSON para as categorias rígidas do seu script dirpf_2026
        mapa_regras_json = {
            'FII': ('FII', '20% GC | Sem isenção | Compensa com FIIs'),
            'Fiagro': ('FII', '20% GC | Sem isenção | Compensa com FIAGROs'),
            'ETF': ('ETF', '15% GC | Sem isenção | Compensa com Ações/ETFs/Cripto'),
            'BDR': ('BDR', '15% GC | Sem isenção | Compensa com Ações/ETFs/Cripto'),
            'UNIT': ('UNIT', '15% GC (Swing) / 20% (Day) | Sem isenção | Compensa com Units'),
            'Ação': ('Ação', '15% GC | Isento até R$ 20k/mês | Compensa com Ações'),
            'Direitos de subscrição': ('Ação', '15% s/ Valor Líquido | Sem isenção | Regra específica (Direito)'),
            'FIP': ('FIP', '15% GC (FIP Comum) ou 0% na Alienação (FIP-IE) | Sem isenção 20k | DISTINGUIR MANUALMENTE')
        }
        
        dados_fallback = []
        
        # CORREÇÃO: Iteramos sobre TODOS os ativos não classificados, não apenas os que o JSON achou
        for ativo in ativos_nao_classificados:
            if ativo in ticker_dict_fallback:
                # O JSON conseguiu classificar
                cat_json = ticker_dict_fallback[ativo]
                cat_final, desc_final = mapa_regras_json.get(cat_json, ('Não Classificado', 'Verificar manualmente'))
            else:
                # O JSON também falhou. Forçamos a tag "Não Classificado" para não perder o ativo
                cat_final = 'Não Classificado'
                desc_final = 'Não encontrado na API nem no JSON. Verificar ticker manualmente.'
            
            dados_fallback.append({'Ativo': ativo, 'Categoria': cat_final, 'Descricao': desc_final})
            
        df_categorias_fallback = pd.DataFrame(dados_fallback)
        
        # Combina os resultados da API com os do Fallback
        df_categorias_final = pd.concat([df_categorias_api, df_categorias_fallback], ignore_index=True)
        print(f"✅ Fallback concluído. {len(df_categorias_fallback)} ativos processados via JSON.")
        
        # Mostra quais ativos ainda ficaram como "Não Classificado" para sua conferência
        nao_classificados_json = df_categorias_fallback[df_categorias_fallback['Categoria'] == 'Não Classificado']['Ativo'].tolist()
        if nao_classificados_json:
            print(f"⚠️ ATENÇÃO: {len(nao_classificados_json)} ativos NÃO foram encontrados nem na API nem no JSON:")
            print(nao_classificados_json)
        
    except FileNotFoundError:
        print("❌ Arquivo 'tickers-cash-market.json' não encontrado. Baixe-o do GitHub e coloque na mesma pasta.")
        df_categorias_final = df_categorias_api.copy()
else:
    print("✅ Todos os ativos foram classificados com sucesso pela API. Fallback não necessário.")
    df_categorias_final = df_categorias_api.copy()
```

    ⚠️ 23 ativos não classificados pela API. Acionando fallback JSON...
    ✅ Fallback concluído. 23 ativos processados via JSON.


**Unificamos os data frames**


```python
df_categorias_final = df_categorias_final.drop_duplicates(subset=['Ativo'], keep='first')

df_merge = df.merge(df_categorias_final, on='Ativo', how='left')

# Verificação final de saúde dos dados
nao_classificados_final = df_merge[df_merge['Categoria'] == 'Não Classificado']['Ativo'].unique()
if len(nao_classificados_final) > 0:
    print(f"\n⚠️ ATENÇÃO: {len(nao_classificados_final)} ativos ainda não classificados. Verifique manualmente:")
    print(nao_classificados_final[:10])
else:
    print("\n🎉 SUCESSO: 100% dos ativos classificados e prontos para o script dirpf_2026!")

# Exibe um resumo
print("\n📊 Resumo da Classificação:")
display(df_merge['Categoria'].value_counts())
```

    
    🎉 SUCESSO: 100% dos ativos classificados e prontos para o script dirpf_2026!
    
    📊 Resumo da Classificação:



    Categoria
    Ação    4672
    FII      309
    UNIT     309
    BDR       60
    ETF       51
    FIP        4
    Name: count, dtype: int64

