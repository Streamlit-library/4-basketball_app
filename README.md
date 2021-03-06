# Basketball app

1. [Importar librerías ](#schema1)
2. [Títulos](#schema2)
3. [Selectbox](#schema3)
4. [Web scraping de NBA](#schema4)
5. [Barra lateral, selección del equipo](#schema5)
6. [Barra lateral - selección de la posición](#schema6)
7. [Filtrado de los datos](#schema7)
8. [Título del dataframe obtenido con los datos del usuario](#schema8)
9. [Descargar los datos](#schema9)
10. [Heatmap](#schema10)

<hr>

<a name="schema1"></a>

# 1. Importar librerías
~~~python
import streamlit as st
import pandas as pd
import base64
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
~~~
<hr>

<a name="schema2"></a>

# 2. Títulos
~~~python
st.title('NBA Player Stats Explorer')

st.markdown("""
This app performs simple webscraping of NBA player stats data!
* **Python libraries:** base64, pandas, streamlit
* **Data source:** [Basketball-reference.com](https://www.basketball-reference.com/).
""")

st.sidebar.header('User Input Features')
~~~
<hr>

<a name="schema3"></a>

# 3. Selectbox
Podemos seleccionar el año entre 1950 y 2020
~~~python
selected_year = st.sidebar.selectbox('Year', list(reversed(range(1950,2020))))
~~~
<hr>

<a name="schema4"></a>

# 4. Web scraping de NBA 
~~~python
@st.cache
def load_data(year):
    url = "https://www.basketball-reference.com/leagues/NBA_" + str(year) + "_per_game.html"
    html = pd.read_html(url, header = 0)
    df = html[0]
    raw = df.drop(df[df.Age == 'Age'].index) # Deletes repeating headers in content
    raw = raw.fillna(0)
    playerstats = raw.drop(['Rk'], axis=1)
    return playerstats
playerstats = load_data(selected_year)
~~~

<hr>

<a name="schema5"></a>

# 5. Barra lateral, selección del equipo

~~~python
sorted_unique_team = sorted(playerstats.Tm.unique())
selected_team = st.sidebar.multiselect('Team', sorted_unique_team, sorted_unique_team)
~~~

<hr>

<a name="schema6"></a>

# 6. Barra lateral - selección de la posición
~~~python
unique_pos = ['C','PF','SF','PG','SG']
selected_pos = st.sidebar.multiselect('Position', unique_pos, unique_pos)
~~~

<hr>

<a name="schema7"></a>

# 7. Filtrado de los datos
~~~python
df_selected_team = playerstats[(playerstats.Tm.isin(selected_team)) & (playerstats.Pos.isin(selected_pos))]
~~~
<hr>

<a name="schema8"></a>

# 8. Título del dataframe obtenido con los datos del usuario
~~~python
st.header('Display Player Stats of Selected Team(s)')
st.write('Data Dimension: ' + str(df_selected_team.shape[0]) + ' rows and ' + str(df_selected_team.shape[1]) + ' columns.')
st.dataframe(df_selected_team)
~~~

<hr>

<a name="schema9"></a>

# 9. Descargar los datos
~~~python
# https://discuss.streamlit.io/t/how-to-download-file-in-streamlit/1806
def filedownload(df):
    csv = df.to_csv(index=False)
    b64 = base64.b64encode(csv.encode()).decode()  # strings <-> bytes conversions
    href = f'<a href="data:file/csv;base64,{b64}" download="playerstats.csv">Download CSV File</a>'
    return href

st.markdown(filedownload(df_selected_team), unsafe_allow_html=True)
~~~
<hr>

<a name="schema10"></a>

# 10. Heatmap
~~~python

if st.button('Intercorrelation Heatmap'):
    st.header('Intercorrelation Matrix Heatmap')
    df_selected_team.to_csv('output.csv',index=False)
    df = pd.read_csv('output.csv')

    corr = df.corr()
    mask = np.zeros_like(corr)
    mask[np.triu_indices_from(mask)] = True
    with sns.axes_style("white"):
        f, ax = plt.subplots(figsize=(7, 5))
        ax = sns.heatmap(corr, mask=mask, vmax=1, square=True)
    st.pyplot()

~~~