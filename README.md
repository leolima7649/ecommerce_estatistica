# Dashboard — Ecommerce Estatística

Este projeto é uma aplicação Dash + Plotly para visualização de dados do arquivo ecommerce_estatistica.csv.

## Como rodar
pip install -r requirements.txt
python app.py

A aplicação abrirá em:
http://127.0.0.1:8050
import pandas as pd
from dash import Dash, html, dcc, Input, Output
import plotly.express as px

df = pd.read_csv("ecommerce_estatistica.csv")

app = Dash(__name__)

num_cols = df.select_dtypes(include='number').columns
cat_cols = df.select_dtypes(exclude='number').columns

app.layout = html.Div([
    html.H1("Dashboard — Ecommerce Estatística"),

    dcc.Dropdown(num_cols, id="num_hist", placeholder="Selecione coluna numérica (Histograma)"),
    dcc.Graph(id="hist"),

    dcc.Dropdown(num_cols, id="num_scatter_x", placeholder="Eixo X (Dispersão)"),
    dcc.Dropdown(num_cols, id="num_scatter_y", placeholder="Eixo Y (Dispersão)"),
    dcc.Graph(id="scatter"),

    dcc.Dropdown(id="num_heat", options=[{"label": c, "value": c} for c in num_cols], placeholder="Mapa de calor"),
    dcc.Graph(id="heatmap"),

    dcc.Dropdown(cat_cols, id="cat_bar", placeholder="Categoria (Barra e Pizza)"),
    dcc.Graph(id="bar"),
    dcc.Graph(id="pie"),
])

@app.callback(Output("hist", "figure"), Input("num_hist", "value"))
def update_hist(col):
    if col is None: return {}
    return px.histogram(df, x=col, title=f"Histograma — {col}")

@app.callback(Output("scatter", "figure"), [Input("num_scatter_x", "value"), Input("num_scatter_y", "value")])
def update_scatter(x, y):
    if x is None or y is None: return {}
    return px.scatter(df, x=x, y=y, title=f"Dispersão — {x} x {y}")

@app.callback(Output("heatmap", "figure"), Input("num_heat", "value"))
def update_heat(col):
    if col is None: return {}
    corr = df.corr()
    return px.imshow(corr, text_auto=True, title="Mapa de Calor — Correlação")

@app.callback([Output("bar", "figure"), Output("pie", "figure")], Input("cat_bar", "value"))
def update_cat(cat):
    if cat is None: return {}, {}
    top = df[cat].value_counts().nlargest(10).reset_index()
    top.columns = [cat, 'count']
    bar = px.bar(top, x=cat, y="count", title=f"Top 10 — {cat}")
    pie = px.pie(top, names=cat, values="count", title=f"Pizza — {cat}")
    return bar, pie

if __name__ == "__main__":
    app.run_server(debug=True)
