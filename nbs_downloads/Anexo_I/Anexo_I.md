
# Mineração de Dados Geoespaciais - Anexo I

Fernando Ferraz Ribeiro

Beth Leite Soares

O objetivo deste notebook é juntar dados de um shapefile com um mapa do Open Street Maps(OSM)

## Testando o ambiente de trabalho (conda environment)
Mostrando o caminho que o servidor Jupyter carrega o interpretador Python (python.exe) e carregas o anbiente (\envs)


```python
import os
import sys
print(sys.executable)
pathFix = sys.prefix
print(pathFix)
```

    C:\Users\ffrib\AppData\Local\conda\conda\envs\Geodata_2018_12_08\python.exe
    C:\Users\ffrib\AppData\Local\conda\conda\envs\Geodata_2018_12_08
    

## Lidando com erros do ambiente (WINDOWS ONLY)
Algumas bibliotecas importadas apresentam um erro apenas na plataforma Windows.

Caso esteja rodando no windows e tenha problemas com o sistema de coordenadas de referência (crs), rode a linha de comando abaixo:



```python
pathFix = pathFix .replace('\\', '/')
print(pathFix)
os.environ["PROJ_LIB"] =   pathFix + "/Library/share"
```

    C:/Users/ffrib/AppData/Local/conda/conda/envs/Geodata_2018_12_08
    


mais informações no [Post sobre o Erro](https://github.com/geopandas/geopandas/issues/830)

## Importando Bibliotecas


```python
# Biblioteca basica de programação científica em python
import numpy as np
# biblioteca de análise de dados
import pandas as pd
# biblioteca de gráficos
import matplotlib.pyplot as plt
import matplotlib.cm as cm
import matplotlib.colors as colors
# Bibliotecas geopandas
import geopandas as gpd
# biblioteca de redes complexas
import networkx as nx
# biblioteca para acessar dados do Open sreet maps
import osmnx as ox
# ler dados remotamente da internet
```


```python
%matplotlib inline
ox.config(use_cache=True, log_console=True)
ox.__version__
```




    '0.8.2'



## Criando coordenadas de recorte


```python
# coordenada inicial x
xSC = 555000
# variação da coordenada x
deltaX = 2000
# coordenada inicial y
ySC = 8570000
# variação da coordenada y
deltaY = 2000
```

## Limites da importaçâo


```python
from shapely.geometry import Polygon

recorte = gpd.GeoSeries([ Polygon([(xSC,ySC), (xSC + deltaX , ySC), (xSC + deltaX, ySC + deltaY ), (xSC, ySC + deltaY )]) ])
```

A geometria criada pela linha de comando acima, embora tenha as corrdenadas relativas ao sistema de projeção Sigras 2000, não tem nenhuma informação georreferenciada. É preciso informar qual o sistema de coordinadas de referência utilizado ( coordenates reference sistem - crs). O Bloco de código abaxo informa que as coordenadas do recorte devem ser tratadas com o sistema Sigras 2000. com unidades em metro.


```python
# colocando em coordenaadas SIRGAS 2000
recorte.crs = {'proj': 'utm', 'zone': 24, 'south': True, 'ellps': 'aust_SA', 'units': 'm', 'no_defs': True}

```

## Carregando shapes


```python
# importando Shape dos bairros - polígonos
bairros = gpd.read_file('../shapefiles/BaseSSA/Limites/bairros_fim.shp', bbox = recorte )

# importando Shape das edificaçõs - polilinhas
edf = gpd.read_file('../shapefiles/BaseSSA/edificacoes_polyline.shp', bbox = recorte)

# importando shape de pontos
edf_pt = gpd.read_file('../shapefiles/BaseSSA/edificacoes_point.shp', bbox = recorte)
```

## Mostrando os dados dos shapes

### dados do shape bairros


```python
bairros.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OBJECTID</th>
      <th>BR_</th>
      <th>BR_ID</th>
      <th>NM_BAIRROS</th>
      <th>Shape_Leng</th>
      <th>Shape_Area</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>89</td>
      <td>2</td>
      <td>1</td>
      <td>Lobato</td>
      <td>10290.734146</td>
      <td>1.508163e+06</td>
      <td>POLYGON ((556643.6323074758 8573040.904571733,...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>91</td>
      <td>2</td>
      <td>1</td>
      <td>Massaranduba</td>
      <td>5027.152427</td>
      <td>5.301521e+05</td>
      <td>POLYGON ((555124.2668471506 8570956.484697679,...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>92</td>
      <td>2</td>
      <td>1</td>
      <td>Santa Luzia</td>
      <td>5579.648112</td>
      <td>3.957189e+05</td>
      <td>POLYGON ((556131.141931782 8571133.633667247, ...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>98</td>
      <td>2</td>
      <td>1</td>
      <td>Alto do Cabrito</td>
      <td>5031.429303</td>
      <td>1.112943e+06</td>
      <td>POLYGON ((557394.0547874847 8572993.633136973,...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>99</td>
      <td>2</td>
      <td>1</td>
      <td>Capelinha</td>
      <td>3337.251376</td>
      <td>4.201418e+05</td>
      <td>POLYGON ((556500.9559292701 8570974.556498181,...</td>
    </tr>
  </tbody>
</table>
</div>



### Dados do shape edificacoes


```python
edf.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>357236</td>
      <td>LINESTRING (555000.0316226622 8570905.59166144...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>357238</td>
      <td>LINESTRING (554999.0173801129 8570926.86740669...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>357239</td>
      <td>LINESTRING (555000.4686540046 8570930.90659893...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>357240</td>
      <td>LINESTRING (555002.9094328225 8570934.45588914...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>357241</td>
      <td>LINESTRING (555009.4978864561 8570930.67664492...</td>
    </tr>
  </tbody>
</table>
</div>




```python
edf_pt.head()

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>367175</td>
      <td>(POINT (556360.5184374301 8570865.76962509))</td>
    </tr>
    <tr>
      <th>1</th>
      <td>367176</td>
      <td>(POINT (556360.5184374301 8570865.76962509))</td>
    </tr>
    <tr>
      <th>2</th>
      <td>367177</td>
      <td>(POINT (556360.5184374301 8570865.76962509))</td>
    </tr>
    <tr>
      <th>3</th>
      <td>367178</td>
      <td>(POINT (556497.9771634268 8571237.37531103))</td>
    </tr>
    <tr>
      <th>4</th>
      <td>367179</td>
      <td>(POINT (556360.5184374301 8570865.76962509))</td>
    </tr>
  </tbody>
</table>
</div>



## Plotando imagens

### limites do recorte


```python
fig, layers = plt.subplots(figsize=(10,10)
                            #,dpi=30
                           )
# pltando a imagem de fundo
recorte.plot(ax = layers, color = 'blue', alpha = 1)
# Plotando os bairros

# layers - quadro o qual poderá ser alterado os valores de xmin, xmax, título e etc

# Plotando os bairros
bairros.plot(ax= layers, color='yellowgreen', edgecolor='black', linestyle="--", lw= 1.5, alpha=1) 

# Plotando as edificações com a cor vermelha
edf.plot(ax= layers, color='darkred', lw= 0.7)

try:
    edf_pt.plot(ax= layers, color='orange', lw= 0.7, )
except:
    pass
# titulo da figura
fig.suptitle('Imagem Teste', fontsize=16)

# limites do gráfico
layers.set_xlim(xSC, xSC + deltaX )
layers.set_ylim(ySC,  ySC + deltaY )

#fig.show()
```




    (8570000, 8572000)




![png](output_27_1.png)


## Lendo Arquivos do banco de dados espaciais online Open Street maps

### Através de arquivo baixado manualmente pelo site


```python
gFile = ox.graph_from_file('../OSM/map.osm')
```


```python
gFile
```




    <networkx.classes.multidigraph.MultiDiGraph at 0x1df6fd9f4e0>




```python
fig, ax = ox.plot_graph(gFile)
plt.tight_layout()
```


![png](output_32_0.png)



    <Figure size 432x288 with 0 Axes>


### Baixando arquivos por coordenadas limite

O open street maps trabalha com coordenadas WGS84, definidas pelo código epsg 4326. O sistema WGS84 utiliza coordenadas em graus de latitude e longitude.


```python
recorte.bounds
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>minx</th>
      <th>miny</th>
      <th>maxx</th>
      <th>maxy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>555000.0</td>
      <td>8570000.0</td>
      <td>557000.0</td>
      <td>8572000.0</td>
    </tr>
  </tbody>
</table>
</div>



Para utilizar a mesma geometria limite da utilizada para a importação do Shapefile, é preciso converter o sistema de coordenadas de referência de Sigras 2000 para WGS84.


```python
recorte_LL = recorte.to_crs({'init': 'epsg:4326'})
recorte_LL.bounds
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>minx</th>
      <th>miny</th>
      <th>maxx</th>
      <th>maxy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-38.49299</td>
      <td>-12.934936</td>
      <td>-38.474516</td>
      <td>-12.916815</td>
    </tr>
  </tbody>
</table>
</div>



O elemento de índice 0 da coluna geometry é um polígono da biblioteca shapely.


```python
 type(recorte_LL.geometry[0])
```




    shapely.geometry.polygon.Polygon



segundo a [documentação do comando ox.graph_from_polygon](https://osmnx.readthedocs.io/en/stable/osmnx.html?highlight=graph_from_place#osmnx.core.graph_from_polygon) ele recebe como priméiro parâmetro um polígono ou multi-polígono da biblioteca citada.

O download do multi-grafo depende da conexão com a internet


```python
gLatLon =  ox.graph_from_polygon(
                                   recorte_LL.geometry[0]
                                 , network_type = 'all_private'
                                 , truncate_by_edge = True
                                 , retain_all = True
                            )
```


```python
gLatLon
```




    <networkx.classes.multidigraph.MultiDiGraph at 0x1df752f64a8>




```python
ox.plot_graph(ox.project_graph(gLatLon))
```


![png](output_43_0.png)





    (<Figure size 449.029x432 with 1 Axes>,
     <matplotlib.axes._subplots.AxesSubplot at 0x1df75462320>)



### Através do endereço (geocoding)


```python
gCode = ox.graph_from_place('Salvador, Bahia, Brazil', network_type="drive")
```

O download do multi-grafo depende da conexão com a internet


```python
gCode
```




    <networkx.classes.multidigraph.MultiDiGraph at 0x1df752e8438>




```python
ox.plot_graph(ox.project_graph(gCode))
```


![png](output_48_0.png)





    (<Figure size 422.933x432 with 1 Axes>,
     <matplotlib.axes._subplots.AxesSubplot at 0x1df731987f0>)



Centralidade é uma medida da importância de um vértice em um grafo. Os critérios de medida da importância variam. Os quatro mais utilizados são:
    1. grau
    2. intermediação
    3. proximidade
    4. vetor próprio
    
O código abaixo calcula a centralidade de proximidade da malha viária da localidade gravada no grafo do OSM

### Antenção!
O custo computacional do cálculo da centralidade cresce exponencialmente de acordo com o número de vértices do grafo(a execução do código abaixo pode demorar muitas horas)


```python
# edge closeness centrality: convert graph to line graph so edges become nodes and vice versa
edge_centrality = nx.closeness_centrality(nx.line_graph(gCode))
```

O código abaixo desenha um gráfico mostrando, em escala de cores a varição da medida de centralidade da malha viária. 


```python
# list of edge values for the orginal graph
ev = [edge_centrality[edge + (0,)] for edge in gCode.edges()]

# color scale converted to list of colors for graph edges
norm = colors.Normalize(vmin=min(ev)*0.8, vmax=max(ev))
cmap = cm.ScalarMappable(norm=norm, cmap=cm.inferno)
ec = [cmap.to_rgba(cl) for cl in ev]

# color the edges in the original graph with closeness centralities in the line graph
fig, ax = ox.plot_graph(gCode, bgcolor=(0.1,0.3,0.0,0.0), axis_off=True, node_size=0, 
                        edge_color=ec, edge_linewidth=1.5, edge_alpha=1)
```


![png](output_53_0.png)


## Juntando o arquivo do OSM com o Arquivo SHP

### Trasnformando de Grafo OSM para geopandas Data frame


```python
osm_pontos, osm_linhas = ox.save_load.graph_to_gdfs(gLatLon)
```


```python
osm_linhas.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>bridge</th>
      <th>geometry</th>
      <th>highway</th>
      <th>key</th>
      <th>lanes</th>
      <th>length</th>
      <th>maxspeed</th>
      <th>name</th>
      <th>oneway</th>
      <th>osmid</th>
      <th>service</th>
      <th>u</th>
      <th>v</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>LINESTRING (-38.4817005 -12.9226106, -38.48173...</td>
      <td>footway</td>
      <td>0</td>
      <td>NaN</td>
      <td>5.818</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>False</td>
      <td>431452585</td>
      <td>NaN</td>
      <td>592377792</td>
      <td>4306484891</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>LINESTRING (-38.4817005 -12.9226106, -38.48143...</td>
      <td>primary</td>
      <td>0</td>
      <td>2</td>
      <td>293.516</td>
      <td>60</td>
      <td>Avenida Afrânio Peixoto</td>
      <td>True</td>
      <td>258852462</td>
      <td>NaN</td>
      <td>592377792</td>
      <td>3944737722</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>LINESTRING (-38.4806693 -12.9168449, -38.48061...</td>
      <td>secondary_link</td>
      <td>0</td>
      <td>NaN</td>
      <td>6.314</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>True</td>
      <td>86577754</td>
      <td>NaN</td>
      <td>592378346</td>
      <td>4306607916</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>LINESTRING (-38.4806693 -12.9168449, -38.48063...</td>
      <td>primary</td>
      <td>0</td>
      <td>2</td>
      <td>69.680</td>
      <td>60</td>
      <td>Avenida Afrânio Peixoto</td>
      <td>True</td>
      <td>406250331</td>
      <td>NaN</td>
      <td>592378346</td>
      <td>592378351</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>LINESTRING (-38.4806036 -12.9174681, -38.48123...</td>
      <td>residential</td>
      <td>0</td>
      <td>NaN</td>
      <td>233.994</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>False</td>
      <td>88627556</td>
      <td>NaN</td>
      <td>592378351</td>
      <td>1029167862</td>
    </tr>
  </tbody>
</table>
</div>



## Sistemas de coordenadas
### trasnformando para o sistema Sigras 2000


```python
# Mudando o sistema de coordenadas de referência

# Para o Shape de Pontos
osm_pontos.to_crs(bairros.crs, inplace = True)
# Para o Shape de Bairros
osm_linhas.to_crs(bairros.crs, inplace = True)
```

Odenando as colunas do shape das linhas


```python
# Ordenando as colunas das linhas para que a coluna de geometria fique por último
osm_linhas_cols = osm_linhas.columns.tolist()
osm_linhas_cols = [x for x in osm_linhas_cols if x != 'geometry']
osm_linhas_cols.append('geometry')
osm_linhas = osm_linhas[osm_linhas_cols]
```


```python
osm_linhas.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>bridge</th>
      <th>highway</th>
      <th>key</th>
      <th>lanes</th>
      <th>length</th>
      <th>maxspeed</th>
      <th>name</th>
      <th>oneway</th>
      <th>osmid</th>
      <th>service</th>
      <th>u</th>
      <th>v</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>footway</td>
      <td>0</td>
      <td>NaN</td>
      <td>5.818</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>False</td>
      <td>431452585</td>
      <td>NaN</td>
      <td>592377792</td>
      <td>4306484891</td>
      <td>LINESTRING (556223.3656601022 8571360.65885263...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>primary</td>
      <td>0</td>
      <td>2</td>
      <td>293.516</td>
      <td>60</td>
      <td>Avenida Afrânio Peixoto</td>
      <td>True</td>
      <td>258852462</td>
      <td>NaN</td>
      <td>592377792</td>
      <td>3944737722</td>
      <td>LINESTRING (556223.3656601022 8571360.65885263...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>secondary_link</td>
      <td>0</td>
      <td>NaN</td>
      <td>6.314</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>True</td>
      <td>86577754</td>
      <td>NaN</td>
      <td>592378346</td>
      <td>4306607916</td>
      <td>LINESTRING (556336.5217465418 8571998.06138254...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>primary</td>
      <td>0</td>
      <td>2</td>
      <td>69.680</td>
      <td>60</td>
      <td>Avenida Afrânio Peixoto</td>
      <td>True</td>
      <td>406250331</td>
      <td>NaN</td>
      <td>592378346</td>
      <td>592378351</td>
      <td>LINESTRING (556336.5217465418 8571998.06138254...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>residential</td>
      <td>0</td>
      <td>NaN</td>
      <td>233.994</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>False</td>
      <td>88627556</td>
      <td>NaN</td>
      <td>592378351</td>
      <td>1029167862</td>
      <td>LINESTRING (556343.5093335714 8571929.12723277...</td>
    </tr>
  </tbody>
</table>
</div>




```python
osm_pontos.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>highway</th>
      <th>osmid</th>
      <th>x</th>
      <th>y</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>592377792</th>
      <td>NaN</td>
      <td>592377792</td>
      <td>-38.4817</td>
      <td>-12.9226</td>
      <td>POINT (556223.3656601022 8571360.658852631)</td>
    </tr>
    <tr>
      <th>592377803</th>
      <td>NaN</td>
      <td>592377803</td>
      <td>-38.4805</td>
      <td>-12.9169</td>
      <td>POINT (556350.339344732 8571996.562528457)</td>
    </tr>
    <tr>
      <th>592378346</th>
      <td>NaN</td>
      <td>592378346</td>
      <td>-38.4807</td>
      <td>-12.9168</td>
      <td>POINT (556336.5217465418 8571998.061382545)</td>
    </tr>
    <tr>
      <th>592378351</th>
      <td>NaN</td>
      <td>592378351</td>
      <td>-38.4806</td>
      <td>-12.9175</td>
      <td>POINT (556343.5093335714 8571929.127232777)</td>
    </tr>
    <tr>
      <th>592378362</th>
      <td>NaN</td>
      <td>592378362</td>
      <td>-38.4815</td>
      <td>-12.9222</td>
      <td>POINT (556243.6960290928 8571404.168260001)</td>
    </tr>
  </tbody>
</table>
</div>



### Estatística descritivas do DF osm_linhas


```python
osm_linhas.describe(include='all')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>bridge</th>
      <th>highway</th>
      <th>key</th>
      <th>lanes</th>
      <th>length</th>
      <th>maxspeed</th>
      <th>name</th>
      <th>oneway</th>
      <th>service</th>
      <th>u</th>
      <th>v</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>6</td>
      <td>2054</td>
      <td>2054.000000</td>
      <td>33</td>
      <td>2054.000000</td>
      <td>33</td>
      <td>232</td>
      <td>2054</td>
      <td>118</td>
      <td>2.054000e+03</td>
      <td>2.054000e+03</td>
      <td>2054</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>1</td>
      <td>14</td>
      <td>NaN</td>
      <td>1</td>
      <td>NaN</td>
      <td>1</td>
      <td>27</td>
      <td>2</td>
      <td>2</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2054</td>
    </tr>
    <tr>
      <th>top</th>
      <td>yes</td>
      <td>residential</td>
      <td>NaN</td>
      <td>2</td>
      <td>NaN</td>
      <td>60</td>
      <td>Rua Padre Antônio Vieira</td>
      <td>False</td>
      <td>driveway</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>LINESTRING (556536.1844486846 8571385.22782885...</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>6</td>
      <td>1713</td>
      <td>NaN</td>
      <td>33</td>
      <td>NaN</td>
      <td>33</td>
      <td>46</td>
      <td>2008</td>
      <td>62</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.006816</td>
      <td>NaN</td>
      <td>74.858000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.708460e+09</td>
      <td>1.708382e+09</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>std</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.082297</td>
      <td>NaN</td>
      <td>74.955946</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.335574e+09</td>
      <td>1.335211e+09</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>min</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>2.301000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>5.923778e+08</td>
      <td>5.923778e+08</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>32.251750</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.005901e+09</td>
      <td>1.005901e+09</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>53.966000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.005902e+09</td>
      <td>1.005902e+09</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>95.081250</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.583408e+09</td>
      <td>1.583408e+09</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>max</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.000000</td>
      <td>NaN</td>
      <td>1139.594000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>5.769358e+09</td>
      <td>5.769358e+09</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
fig2, layers2 = plt.subplots(figsize=(10,10)
                            #,dpi=30
                           )

recorte.plot(ax = layers2, color = 'blue', alpha = 1)
# Plotando os bairros
# Limites preto
# alpha - transparência
# layers - quadro o qual poderá ser alterado os valores de xmin, xmax, título e etc


bairros.plot(ax= layers2, color='yellowgreen', edgecolor='black', linestyle="--", lw= 1.5, alpha=1) 

# Plotando as edificações com a cor vermelha
edf.plot(ax= layers2, color='darkred', lw= 0.7)

try:
    edf_pt.plot(ax= layers2, color='orange', lw= 0.7, )
except:
    pass

# importados do OSM
osm_linhas.plot(ax= layers2, color='teal', lw= 0.8, )
osm_pontos.plot(ax= layers2, color='indigo', lw= 0.4, )

# titulo da figura
fig2.suptitle('Imagem Teste', fontsize=16)

# limites do gráfico
layers2.set_xlim(xSC, xSC + deltaX )
layers2.set_ylim(ySC,  ySC + deltaY )

```




    (8570000, 8572000)




![png](output_66_1.png)



```python
osm_linhas.dtypes
```




    bridge       object
    highway      object
    key           int32
    lanes        object
    length      float64
    maxspeed     object
    name         object
    oneway        int32
    osmid        object
    service      object
    u             int64
    v             int64
    geometry     object
    dtype: object



Devido a incompatibilidades entre as estruturas de dados do shapefile e do OSM algumas colunas precisam ser tratadas


```python
osm_linhas.oneway = osm_linhas.oneway.astype(int)
osm_linhas.key = osm_linhas.key.astype(int)
osm_linhas.name = osm_linhas.name.astype(str)
osm_linhas.bridge = osm_linhas.bridge.astype(str)

```


```python
osm_select = osm_linhas.filter( items= [ 'name', 'oneway','key','length', 'bridge' ,'geometry' ]) 
```


```python
osm_select.to_file('./output/teste01.shp', driver='ESRI Shapefile')
```
