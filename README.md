import streamlit as st
import pandas as pd
import numpy as np
import plotly.express as px
from datetime import datetime, timedelta

def main():
    st.set_page_config(page_title="Dashboard Genómico", layout="wide")

    st.title("Dashboard de Visualización Genómica")

    st.sidebar.header("Opciones de filtrado")
    selected_gene = st.sidebar.selectbox("Selecciona un gen:", ["BRCA1", "TP53", "EGFR", "KRAS", "MYC"])
    selected_date_range = st.sidebar.date_input("Rango de fechas:", [datetime.now() - timedelta(days=30), datetime.now()])

    data = generate_mock_data()

    filtered_data = filter_data(data, selected_gene, selected_date_range)

    display_kpis(filtered_data)

    display_charts(filtered_data, selected_gene)

    st.subheader("Datos filtrados")
    st.dataframe(filtered_data)

    st.sidebar.write("**Última actualización:**")
    st.sidebar.write(datetime.now().strftime("%Y-%m-%d %H:%M:%S"))

def generate_mock_data():
    np.random.seed(42)
    return pd.DataFrame({
        "Fecha": pd.date_range(start="2024-01-01", periods=100),
        "Gen": np.random.choice(["BRCA1", "TP53", "EGFR", "KRAS", "MYC"], 100),
        "Expresión": np.random.rand(100) * 100,
        "Mutaciones": np.random.randint(0, 10, 100),
        "Población afectada (%)": np.random.rand(100) * 20
    })

def filter_data(data, selected_gene, selected_date_range):
    return data[(data["Gen"] == selected_gene) &
                (data["Fecha"] >= pd.to_datetime(selected_date_range[0])) &
                (data["Fecha"] <= pd.to_datetime(selected_date_range[1]))]

def display_kpis(filtered_data):
    st.subheader("Indicadores Clave (KPIs)")
    col1, col2, col3, col4 = st.columns(4)
    with col1:
        st.metric("Prom. Expresión", f"{filtered_data['Expresión'].mean():.2f}")
    with col2:
        st.metric("Total Mutaciones", int(filtered_data["Mutaciones"].sum()))
    with col3:
        st.metric("Máx. Población afectada (%)", f"{filtered_data['Población afectada (%)'].max():.2f}%")
    with col4:
        st.metric("Días analizados", len(filtered_data["Fecha"].unique()))

def display_charts(filtered_data, selected_gene):
    st.subheader(f"Visualización de datos para el gen {selected_gene}")

    fig_line = px.line(filtered_data, x="Fecha", y="Expresión", title="Expresión Genómica a lo largo del tiempo")
    st.plotly_chart(fig_line, use_container_width=True)

    fig_bar = px.bar(filtered_data, x="Fecha", y="Mutaciones", title="Mutaciones por Fecha")
    st.plotly_chart(fig_bar, use_container_width=True)

    fig_scatter = px.scatter(filtered_data, x="Expresión", y="Población afectada (%)",
                             size="Mutaciones", color="Fecha",
                             title="Relación entre Expresión y Población afectada")
    st.plotly_chart(fig_scatter, use_container_width=True)

if __name__ == "__main__":
    main()
