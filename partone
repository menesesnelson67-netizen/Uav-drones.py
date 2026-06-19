import streamlit as st
import numpy as np
import matplotlib.pyplot as plt
from scipy.optimize import linprog

# Configuración inicial
st.set_page_config(page_title="Drone Mission Control", layout="wide")
st.title("🛸 Optimizador de Flota de Drones con Backhaul Satelital")

# Sidebar para parámetros
with st.sidebar:
    st.header("⚙️ Parámetros Globales")
    limit_energia = st.number_input("Límite Energía (Wh)", value=180)
    limit_presupuesto = st.number_input("Límite Presupuesto (USD/h)", value=120)
    limit_backhaul = st.number_input("Límite Satélite (Mbps)", value=150)
    
    st.divider()
    z_fijo = st.slider("Valor de z (Drones Tipo C) para visualización", 0, 10, 1)

# Layout principal
col1, col2 = st.columns([1, 1])

with col1:
    st.subheader("📝 Matriz de Consumo")
    # Definimos los consumos del problema
    data = {
        'A': [30, 10, 25],
        'B': [20, 20, 15],
        'C': [10, 15, 10]
    }
    
    # Botón de ejecución
    ejecutar = st.button("🚀 Ejecutar Optimización y Generar Gráfico", use_container_width=True)

if ejecutar:
    # 1. Optimización Numérica (Maximizamos x + y + z)
    c = [-1, -1, -1]
    A = [
        [data['A'][0], data['B'][0], data['C'][0]],
        [data['A'][1], data['B'][1], data['C'][1]],
        [data['A'][2], data['B'][2], data['C'][2]]
    ]
    b = [limit_energia, limit_presupuesto, limit_backhaul]
    
    res = linprog(c, A_ub=A, b_ub=b, bounds=(0, None), method='highs')
    
    with col1:
        if res.success:
            st.success("¡Solución óptima encontrada!")
            st.metric("Drones Tipo A (x)", round(res.x[0], 2))
            st.metric("Drones Tipo B (y)", round(res.x[1], 2))
            st.metric("Drones Tipo C (z)", round(res.x[2], 2))
        else:
            st.error("No se pudo hallar una solución factible.")

    # 2. Representación Gráfica (Proyección z = fijo)
    with col2:
        st.subheader(f"📊 Región de Factibilidad (Proyección z = {z_fijo})")
        
        x = np.linspace(0, 10, 400)
        
        # Inecuaciones despejadas para y:
        # 30x + 20y + 10z <= 180 -> y <= (180 - 10z - 30x) / 20
        y1 = (limit_energia - data['C'][0]*z_fijo - data['A'][0]*x) / data['B'][0]
        # 10x + 20y + 15z <= 120 -> y <= (120 - 15z - 10x) / 20
        y2 = (limit_presupuesto - data['C'][1]*z_fijo - data['A'][1]*x) / data['B'][1]
        # 25x + 15y + 10z <= 150 -> y <= (150 - 10z - 25x) / 15
        y3 = (limit_backhaul - data['C'][2]*z_fijo - data['A'][2]*x) / data['B'][2]

        fig, ax = plt.subplots()
        ax.plot(x, y1, label='Energía', color='red', linestyle='--')
        ax.plot(x, y2, label='Presupuesto', color='blue', linestyle='--')
        ax.plot(x, y3, label='Backhaul', color='green', linestyle='--')
        
        # Pintar la región de factibilidad
        y_min = np.minimum(np.maximum(0, y1), np.minimum(np.maximum(0, y2), np.maximum(0, y3)))
        ax.fill_between(x, 0, y_min, color='gray', alpha=0.3, label='Región Factible')
        
        ax.set_xlim(0, 7)
        ax.set_ylim(0, 7)
        ax.set_xlabel('Drones Tipo A (x)')
        ax.set_ylabel('Drones Tipo B (y)')
        ax.legend()
        ax.grid(True, alpha=0.3)
        
        st.pyplot(fig)

        # 3. Cálculo de Vértices para la proyección
        st.subheader("📍 Vértices del Polígono (Proyección)")
        st.info("Puntos de corte con ejes y entre restricciones para el plano z actual.")
        # Lógica simplificada de visualización de puntos críticos
        st.write("- Origen: (0, 0)")
        st.write(f"- Corte Eje Y (Presupuesto): (0, {max(0, (limit_presupuesto - 15*z_fijo)/20)})")
        st.write(f"- Corte Eje X (Energía): ({max(0, (limit_energia - 10*z_fijo)/30)}, 0)")
