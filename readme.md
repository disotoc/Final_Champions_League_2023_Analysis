# **Análisis final Champions League 2023**

El objetivo de este análisis es verificar las principales estadísticas de la final de la Champions League 2023

Si solo necesitas código del análisis lo puedes encontrar [aquí](/analysis.ipynb)

## Análisis exploratorio

Se partirá de una base con los partidos de Champions League desde el año 2003 hasta el 2022, el dataset principal se obtuvo desde [data.world](https://data.world/brocolidata/uefa-champions-league-matches-2004-2022) y luego de una limpieza de, se compone de lo siguiente:

```python
import pandas as pd
import matplotlib.pyplot as plt
df = pd.read_csv('https://query.data.world/s/b6adganypdgbvdrk5gftvq2dmp5bk7?dws=00000')
df.drop(columns=['_seasonid',
                 'comment',
                 'stadiumid',
                 'bestof',
                 'walkover',
                 'retired',
                 'disqualified',
                 'numberofperiods',
                 'roundname_displaynumber',
                 'result_period',
                 'teams_home__doc',
                 'teams_home_mediumname',
                 'teams_away_name.1',
                 'teams_away_mediumname',
                 'teams_away_nickname',
                 'periods_ot_home',
                 'periods_ot_away',
                 'periods_ap_home',
                 'periods_ap_away'], inplace=True)
df['periods_ft_home'] = df['result_home'] - df['periods_p1_home']
df['periods_ft_away'] = df['result_away'] - df['periods_p1_away']
```
> Se calculó nuevamente algunas columnas por un pequeño error en variables

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 4210 entries, 0 to 4209
Data columns (total 21 columns):
 #   Column                   Non-Null Count  Dtype  
---  ------                   --------------  -----  
 0   match_id                 4210 non-null   int64  
 1   round                    4210 non-null   int64  
 2   roundname__doc           4210 non-null   object 
 3   time_datetime            4210 non-null   object 
 4   teams_home_name          4210 non-null   object 
 5   result_home              4126 non-null   float64
 6   result_away              4126 non-null   float64
 7   teams_away_name          4210 non-null   object 
 8   cuproundmatchnumber      2290 non-null   float64
 9   cuproundnumberofmatches  2290 non-null   float64
 10  week                     4210 non-null   int64  
 11  neutralground            4210 non-null   bool   
 12  roundname_name           4210 non-null   object 
 13  roundname_shortname      2290 non-null   object 
 14  result_winner            3213 non-null   object 
 15  periods_p1_home          4122 non-null   float64
 16  periods_p1_away          4122 non-null   float64
 17  periods_ft_home          4126 non-null   float64
 18  periods_ft_away          4126 non-null   float64
 19  teams_home_abbr          4210 non-null   object 
 20  teams_away_abbr          4210 non-null   object 
dtypes: bool(1), float64(8), int64(3), object(9)
memory usage: 662.0+ KB
```

### Diccionario de variables
- match_id: campo id (no considerar)
- round: campo id (no considerar)
- roundname__doc: indica la fase, se divide en ['cupround', 'tableround']
- time_datetime: fecha partido
- teams_home_name: nombre equipo local
- result_home: goles equipo local
- result_away: goles equipo visitante
- teams_away_name: nombre equipo visitante
- cuproundmatchnumber: número de partidos de encuentro
- cuproundnumberofmatches: número total de partidos de encuentro
- week: semana calendario
- neutralground: booleano que indica si el partido se jugo en un estadio que no corresponde a ningún equipo del encuentro
- roundname_name: indica el nombre de la fase específica, se divide en ['Demi-finales', 'Finale', 'Tour qualificatif 1', 'Tour qualificatif 2', 'Tour qualificatif 3', 'Tour qualificatif 4', '1', '2', '3', '4', '5', '6', '1/8e de finale', 'Quarts de finale', 'Tour 1', 'Tour 2', 'Tour 3']
- roundname_shortname: abreviación de la fase específica, no incluye lo que es del 1 al 8 partido, se divide en ['DF', 'F', 'TQ1', 'TQ2', 'TQ3', 'TQ4', nan, '1/8', 'QF', 'T1', 'T2', 'M3']
- result_winner: ganador, se divide en ['away', nan, 'home']
- periods_p1_home: goles equipo visitante en primer tiempo del partido
- periods_p1_away: goles equipo visitante en primer tiempo del partido
- periods_ft_home: goles equipo visitante en segundo tiempo del partido
- periods_ft_away: goles equipo visitante en segundo tiempo del partido
- teams_home_abbr: nombre abreviado equipo local
- teams_away_abbr: nombre abreviado equipo visitante

Luego se dividió en dataset con solo los equipos finalistas, los resultados son los siguientes:

**Código**
```python
def to_df_individual(team_name):
    """
    Extrae los datos de un solo equipo en particular
    """
    team_home = df.loc[df['teams_home_abbr'] == team_name].copy()
    df.loc[df['teams_home_abbr'] == team_name, 'local'] = True
    team_away = df.loc[df['teams_away_abbr'] == team_name].copy()
    team_away['local'] = False
    df.loc[df['teams_away_abbr'] == team_name, 'local'] = False
    df_team = pd.concat([team_home, team_away], ignore_index=True)
    df_team['local'] = df_team['local'].astype('bool')
    return df_team

df_inter = to_df_individual('INT')
df_city = to_df_individual('MNC')

df_inter_home = df_inter[df_inter['teams_home_abbr'] == 'INT']
df_inter_away = df_inter[df_inter['teams_away_abbr'] == 'INT']
df_city_home = df_city[df_city['teams_home_abbr'] == 'MNC']
df_city_away = df_city[df_city['teams_away_abbr'] == 'MNC']
```
### DF Inter de Milan
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 119 entries, 0 to 118
Data columns (total 22 columns):
 #   Column                   Non-Null Count  Dtype  
---  ------                   --------------  -----  
 0   match_id                 119 non-null    int64  
 1   round                    119 non-null    int64  
 2   roundname__doc           119 non-null    object 
 3   time_datetime            119 non-null    object 
 4   teams_home_name          119 non-null    object 
 5   result_home              114 non-null    float64
 6   result_away              114 non-null    float64
 7   teams_away_name          119 non-null    object 
 8   cuproundmatchnumber      35 non-null     float64
 9   cuproundnumberofmatches  35 non-null     float64
 10  week                     119 non-null    int64  
 11  neutralground            119 non-null    bool   
 12  roundname_name           119 non-null    object 
 13  roundname_shortname      35 non-null     object 
 14  result_winner            87 non-null     object 
 15  periods_p1_home          115 non-null    float64
 16  periods_p1_away          115 non-null    float64
 17  periods_ft_home          114 non-null    float64
 18  periods_ft_away          114 non-null    float64
 19  teams_home_abbr          119 non-null    object 
 20  teams_away_abbr          119 non-null    object 
dtypes: bool(2), float64(8), int64(3), object(9)
memory usage: 19.0+ KB
```

### DF Manchester City
```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 111 entries, 0 to 110
Data columns (total 22 columns):
 #   Column                   Non-Null Count  Dtype  
---  ------                   --------------  -----  
 0   match_id                 111 non-null    int64  
 1   round                    111 non-null    int64  
 2   roundname__doc           111 non-null    object 
 3   time_datetime            111 non-null    object 
 4   teams_home_name          111 non-null    object 
 5   result_home              106 non-null    float64
 6   result_away              106 non-null    float64
 7   teams_away_name          111 non-null    object 
 8   cuproundmatchnumber      39 non-null     float64
 9   cuproundnumberofmatches  39 non-null     float64
 10  week                     111 non-null    int64  
 11  neutralground            111 non-null    bool   
 12  roundname_name           111 non-null    object 
 13  roundname_shortname      39 non-null     object 
 14  result_winner            88 non-null     object 
 15  periods_p1_home          106 non-null    float64
 16  periods_p1_away          106 non-null    float64
 17  periods_ft_home          106 non-null    float64
 18  periods_ft_away          106 non-null    float64
 19  teams_home_abbr          111 non-null    object 
 20  teams_away_abbr          111 non-null    object 
dtypes: bool(2), float64(8), int64(3), object(9)
memory usage: 17.7+ KB
```

---

## Estadísticas a destacar

En base a los datos expuestos, las estadísticas que más podríamos destacar son las siguientes

### **Promedio de goles por partido:**

Esta estadística puede ser interesante para comparar el rendimiento de los equipos finalistas en términos de su capacidad para anotar goles. Se generó el siguiente código

```python
# calcular el promedio de goles por partido para cada equipo
df_inter_goals = pd.concat([df_inter_home['result_home'].fillna(0), df_inter_away['result_away'].fillna(0)])
df_city_goals = pd.concat([df_city_home['result_home'].fillna(0), df_city_away['result_away'].fillna(0)])

# Entrega de resultados
print('\033[1mInter Milan promedio de goles por partido:\033[0m', round(df_inter_goals.mean(), 2))
print('\033[1mManchester City promedio de goles por partido:\033[0m', round(df_city_goals.mean(), 2))

# Gráfico
fig, ax = plt.subplots(figsize=(6, 5))
fig.patch.set_facecolor('#0e173d')  # Fondo oscuro

x = ['Inter de Milán', 'Manchester City']
y = [df_inter_goals.mean(), df_city_goals.mean()]

ax.bar(x, y, color=['#011c98', '#90b7db'])
#ax.set_title('Promedio de goles realizados por partido', fontsize=12, color='white')
ax.set_xlabel('Equipo', color='white')
ax.set_ylabel('Promedio de goles', color='white')

# Cambiar el color de las etiquetas de los ejes (ticks)
ax.tick_params(axis='x', colors='white')
ax.tick_params(axis='y', colors='white')

plt.text(-0.1, -0.15, 'Gráfico creado por Diego Soto C.', ha='left', transform=ax.transAxes, color='white')

plt.savefig('assets/Promedio_goles_bar.png', bbox_inches='tight', dpi=300, transparent=True)
plt.show()
```

Los resultados fueron:

- Inter Milan promedio de goles por partido: 1.29
- Manchester City promedio de goles por partido: 1.96

![Promedio_goles_bar.png](/assets/Promedio_goles_bar.png)

### **Distribución de goles por partido:**

Se podría graficar la distribución de goles por partido para cada equipo finalista. Esto podría ayudar a evaluar la consistencia de cada equipo en términos de anotar goles. Para hacer esto, se creará un gráfico de barras con la frecuencia de goles anotados por partido para cada equipo.

```python
# Filtrar partidos jugados por el Inter de Milán y Manchester City respectivamente
df_inter_goals = pd.concat([df_inter_home['result_home'].fillna(0), df_inter_away['result_away'].fillna(0)])
df_city_goals = pd.concat([df_city_home['result_home'].fillna(0), df_city_away['result_away'].fillna(0)])

# Calcular las frecuencias
inter_freq = df_inter_goals.value_counts().sort_index()
city_freq = df_city_goals.value_counts().sort_index()

# Gráfico
fig, ax = plt.subplots(figsize=(6, 5))
fig.patch.set_facecolor('#0e173d')  # Fondo oscuro

x_inter = inter_freq.index.astype(str)
y_inter = inter_freq.values
x_city = city_freq.index.astype(str)
y_city = city_freq.values

# Dibujar un rectángulo blanco para cubrir la sección de las barras
ax.axvspan(-1, max(city_freq.index) + 0.5, facecolor='white', edgecolor='none', alpha=0.7)

ax.bar(x_inter, y_inter, alpha=0.9, label='Inter de Milán', color='#011c98')
ax.bar(x_city, y_city, alpha=0.7, label='Manchester City', color='#90b7db')

ax.set_title('Frecuencia de goles por partido', fontsize=12, color='white', x=0.5)
ax.set_xlabel('Goles', color='white')
ax.set_ylabel('Frecuencia', color='white')

# Cambiar el color de las etiquetas de los ejes (ticks)
ax.tick_params(axis='x', colors='white')
ax.tick_params(axis='y', colors='white')

plt.text(-0.1, -0.15, 'Gráfico creado por Diego Soto C.', ha='left', transform=ax.transAxes, color='white')

plt.legend()
plt.savefig('assets/Frecuencia_goles.png', bbox_inches='tight', dpi=300, transparent=True)
plt.show()
```

El resultado es el siguiente:

![Frecuencia_goles.png](/assets/Frecuencia_goles.png)

En este gráfico de barras podemos concluir lo siguiente:

- El Inter de Milan tiene una mayor tendencia a hacer menos goles,
- Lo más común es que el Manchester City haga por lo menos 1 gol
- Entre los dos equipos, el Manchester City es el único que ha realizado 6 y 7 goles en algunos partidos

### **Porcentaje de victorias en partidos previos:**

Esta estadística puede ser útil para evaluar el éxito que ha tenido cada equipo en partidos previos. Se calculará el porcentaje de victorias para cada equipo para luego compararlos Para calcular el porcentaje de victorias en partidos previos, se filtran los partidos jugados por cada equipo y luego se cuenta el número de victorias y derrotas para cada equipo, a esto se adicionó el porcentaje de victorias tanto de local como visitante

```python
# Porcentaje de victorias Inter de Milán
# contar el número de victorias y derrotas de local
inter_home_wins = (df_inter['teams_home_abbr'] == 'INT') & (df_inter['result_winner'] == 'home')
inter_home_losses = (df_inter['teams_home_abbr'] == 'INT') & (df_inter['result_winner'] == 'away')
# contar el número de victorias y derrotas de visita
inter_away_wins = (df_inter['teams_away_abbr'] == 'INT') & (df_inter['result_winner'] == 'away')
inter_away_losses = (df_inter['teams_away_abbr'] == 'INT') & (df_inter['result_winner'] == 'home')
# contar el número de victorias y derrotas totales
inter_wins = inter_home_wins | inter_away_wins
inter_losses = inter_home_losses | inter_away_losses
# calcular el porcentaje de victorias de local, visita y total
inter_home_win_pct = round(inter_home_wins.sum() / (inter_home_wins.sum() + inter_home_losses.sum()) * 100, 2)
inter_away_win_pct = round(inter_away_wins.sum() / (inter_away_wins.sum() + inter_away_losses.sum()) * 100, 2)
inter_win_pct = round(inter_wins.sum() / (inter_wins.sum() + inter_losses.sum()) * 100, 2)
# Output

print('La cantidad de partidos ganados del Inter de Milán es de:', inter_wins.sum())
print('La cantidad de partidos perdidos del Inter de Milán es de:', inter_losses.sum())
print('Porcentaje de victorias del Inter de Milán en partidos previos como local:',inter_home_win_pct, '%')
print('Porcentaje de victorias del Inter de Milán en partidos previos como visitante:',inter_away_win_pct, '%')
print('Porcentaje de victorias del Inter de Milán en partidos previos totales:', inter_win_pct, '%')

print("\n")
# Porcentaje de victorias Manchester City
# contar el número de victorias y derrotas de local
city_home_wins = (df_city['teams_home_abbr'] == 'MNC') & (df_city['result_winner'] == 'home')
city_home_losses = (df_city['teams_home_abbr'] == 'MNC') & (df_city['result_winner'] == 'away')
# contar el número de victorias y derrotas de visita
city_away_wins = (df_city['teams_away_abbr'] == 'MNC') & (df_city['result_winner'] == 'away')
city_away_losses = (df_city['teams_away_abbr'] == 'MNC') & (df_city['result_winner'] == 'home')
# contar el número de victorias y derrotas totales
city_wins = city_home_wins | city_away_wins
city_losses = city_home_losses | city_away_losses
# calcular el porcentaje de victorias de local, visita y total
city_home_win_pct = round(city_home_wins.sum() / (city_home_wins.sum() + city_home_losses.sum()) * 100, 2)
city_away_win_pct = round(city_away_wins.sum() / (city_away_wins.sum() + city_away_losses.sum()) * 100, 2)
city_win_pct = round(city_wins.sum() / (city_wins.sum() + city_losses.sum()) * 100, 2)
# Output
print('La cantidad de partidos ganados del Manchester City es de:', city_wins.sum())
print('La cantidad de partidos perdidos del Manchester City es de:', city_losses.sum())
print('Porcentaje de victorias del Manchester City en partidos previos como local:', city_home_win_pct, '%')
print('Porcentaje de victorias del Manchester City en partidos previos como visitante:', city_away_win_pct, '%')
print('Porcentaje de victorias del Manchester City en partidos previos:', city_win_pct, '%')

```

Los resultados fueron:

- La cantidad de partidos ganados del Inter de Milán es de: 51
- La cantidad de partidos perdidos del Inter de Milán es de: 36
- Porcentaje de victorias del Inter de Milán en partidos previos como local: 49.15 %
- Porcentaje de victorias del Inter de Milán en partidos previos como visitante: 36.67 %
- Porcentaje de victorias del Inter de Milán en partidos previos totales: 42.86 %
- La cantidad de partidos ganados del Manchester City es de: 60
- La cantidad de partidos perdidos del Manchester City es de: 28
- Porcentaje de victorias del Manchester City en partidos previos como local: 59.65 %
- Porcentaje de victorias del Manchester City en partidos previos como visitante: 48.15 %
- Porcentaje de victorias del Manchester City en partidos previos: 54.05 %