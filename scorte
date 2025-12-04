import streamlit as st
import pandas as pd
from datetime import datetime
from fpdf import FPDF

# --- CONFIGURAZIONE PAGINA ---
st.set_page_config(
    page_title="Gestione Magazzino",
    layout="wide",
    initial_sidebar_state="expanded"
)

# --- CSS STILE BUSINESS (CLEAN & PROFESSIONAL) ---
st.markdown("""
    <style>
    /* Sfondo e Font */
    .stApp {
        background-color: #ffffff;
        font-family: 'Segoe UI', Helvetica, Arial, sans-serif;
        color: #333333;
    }
    
    /* Sidebar */
    [data-testid="stSidebar"] {
        background-color: #f8f9fa;
        border-right: 1px solid #e9ecef;
    }
    
    /* Intestazioni */
    h1, h2, h3 {
        color: #1e293b !important; /* Slate Blue Scuro */
        font-weight: 600;
        letter-spacing: -0.5px;
    }
    
    /* KPI Cards */
    div[data-testid="metric-container"] {
        background-color: #ffffff;
        border: 1px solid #e2e8f0;
        padding: 15px;
        border-radius: 6px;
        box-shadow: 0 2px 4px rgba(0,0,0,0.02);
    }
    
    /* Bottoni */
    .stButton>button {
        background-color: #1e293b;
        color: white;
        border: none;
        border-radius: 4px;
        padding: 0.6rem 1.2rem;
        font-weight: 500;
        width: 100%;
        transition: background 0.2s;
    }
    .stButton>button:hover {
        background-color: #334155;
    }
    
    /* Tabella */
    div[data-testid="stDataEditor"] {
        border: 1px solid #e2e8f0;
        border-radius: 4px;
    }
    
    /* Alert */
    .stAlert {
        background-color: #fff1f2;
        color: #9f1239;
        border: 1px solid #fecdd3;
    }
    </style>
    """, unsafe_allow_html=True)

# --- STATO (DATABASE SIMULATO) ---
if 'inventory' not in st.session_state:
    # Dati iniziali di esempio
    st.session_state.inventory = pd.DataFrame([
        {"Codice": "A001", "Prodotto": "Laptop Aziendale 15", "Categoria": "Elettronica", "Posizione": "A-12", "Giacenza": 5, "Minimo": 2, "Prezzo Unitario": 850.00},
        {"Codice": "C023", "Prodotto": "Carta A4 (Pacchi)", "Categoria": "Ufficio", "Posizione": "B-05", "Giacenza": 12, "Minimo": 20, "Prezzo Unitario": 4.50},
        {"Codice": "M104", "Prodotto": "Filtri Olio Motore", "Categoria": "Ricambi", "Posizione": "C-01", "Giacenza": 8, "Minimo": 10, "Prezzo Unitario": 12.00},
        {"Codice": "K990", "Prodotto": "Kit Primo Soccorso", "Categoria": "Sicurezza", "Posizione": "Ingresso", "Giacenza": 2, "Minimo": 1, "Prezzo Unitario": 45.00},
    ])

# --- FUNZIONE PDF REPORT ---
def generate_pdf(dataframe):
    pdf = FPDF(orientation='L', unit='mm', format='A4')
    pdf.add_page()
    
    # Intestazione
    pdf.set_font("Helvetica", 'B', 16)
    pdf.set_text_color(30, 41, 59)
    pdf.cell(0, 10, "INVENTARIO MAGAZZINO", 0, 1, 'L')
    
    pdf.set_font("Helvetica", size=10)
    pdf.set_text_color(100, 100, 100)
    pdf.cell(0, 8, f"Report generato il: {datetime.now().strftime('%d/%m/%Y')}", 0, 1, 'L')
    pdf.ln(5)
    
    # Configurazione Colonne
    headers = ["Codice", "Prodotto", "Posizione", "Qta", "Min", "Prezzo", "Valore Tot."]
    widths = [25, 85, 30, 20, 20, 30, 35]
    
    # Header Tabella
    pdf.set_fill_color(241, 245, 249) # Grigio molto chiaro
    pdf.set_text_color(0, 0, 0)
    pdf.set_font("Helvetica", 'B', 10)
    
    for i, h in enumerate(headers):
        pdf.cell(widths[i], 10, h, 1, 0, 'L', True)
    pdf.ln()
    
    # Righe
    pdf.set_font("Helvetica", size=10)
    total_inventory_value = 0
    
    for _, row in dataframe.iterrows():
        # Calcoli
        qty = row["Giacenza"]
        min_qty = row["Minimo"]
        price = row["Prezzo Unitario"]
        val = qty * price
        total_inventory_value += val
        
        # Encoding stringhe
        code = str(row["Codice"]).encode('latin-1', 'replace').decode('latin-1')
        prod = str(row["Prodotto"]).encode('latin-1', 'replace').decode('latin-1')
        pos = str(row["Posizione"]).encode('latin-1', 'replace').decode('latin-1')
        
        # Logica Sottoscorta (Rosso nel PDF)
        is_low = qty < min_qty
        
        pdf.cell(widths[0], 10, code, 1)
        
        if is_low:
            pdf.set_text_color(220, 38, 38) # Rosso
            pdf.set_font("Helvetica", 'B', 10)
            pdf.cell(widths[1], 10, f"{prod} (!)", 1)
            pdf.set_text_color(0, 0, 0)
            pdf.set_font("Helvetica", size=10)
        else:
            pdf.cell(widths[1], 10, prod, 1)
            
        pdf.cell(widths[2], 10, pos, 1)
        pdf.cell(widths[3], 10, str(qty), 1)
        pdf.cell(widths[4], 10, str(min_qty), 1)
        pdf.cell(widths[5], 10, f"{price:.2f}", 1)
        pdf.cell(widths[6], 10, f"{val:.2f}", 1)
        pdf.ln()
        
    # Totale Finale
    pdf.ln(5)
    pdf.set_font("Helvetica", 'B', 12)
    pdf.cell(0, 10, f"VALORE TOTALE: EUR {total_inventory_value:,.2f}", 0, 1, 'R')
    
    return pdf.output(dest='S').encode('latin-1')

# --- SIDEBAR: INSERIMENTO ---
with st.sidebar:
    st.header("Nuovo Articolo")
    st.caption("Aggiungi referenze al magazzino.")
    
    with st.form("add_item", clear_on_submit=True):
        n_code = st.text_input("Codice (SKU)")
        n_prod = st.text_input("Nome Prodotto")
        n_cat = st.selectbox("Categoria", ["Generale", "Materia Prima", "Prodotti Finiti", "Ricambi", "Ufficio"])
        n_pos = st.text_input("Posizione (Scaffale/Ripiano)")
        
        col_s1, col_s2 = st.columns(2)
        with col_s1:
            n_qty = st.number_input("Giacenza", 0, 10000, 0)
        with col_s2:
            n_min = st.number_input("Soglia Minima", 0, 1000, 5)
            
        n_price = st.number_input("Prezzo Unitario (€)", 0.0, 100000.0, 0.0, step=0.5)
        
        if st.form_submit_button("Inserisci"):
            if n_prod:
                new_data = pd.DataFrame([{
                    "Codice": n_code, 
                    "Prodotto": n_prod, 
                    "Categoria": n_cat,
                    "Posizione": n_pos,
                    "Giacenza": n_qty, 
                    "Minimo": n_min, 
                    "Prezzo Unitario": n_price
                }])
                st.session_state.inventory = pd.concat([st.session_state.inventory, new_data], ignore_index=True)
                st.success("Articolo aggiunto.")
            else:
                st.warning("Il nome del prodotto è obbligatorio.")

# --- MAIN DASHBOARD ---
st.title("Gestione Magazzino")
st.markdown("Monitoraggio giacenze, valore e riordini.")

df = st.session_state.inventory

# Calcoli KPI
total_refs = len(df)
total_value = (df["Giacenza"] * df["Prezzo Unitario"]).sum()
# Filtro Sottoscorta
low_stock_df = df[df["Giacenza"] < df["Minimo"]]
low_stock_count = len(low_stock_df)

# 1. METRICHE
col1, col2, col3 = st.columns(3)
col1.metric("Referenze Totali", total_refs)
col2.metric("Valore Magazzino", f"€ {total_value:,.2f}")
col3.metric("Articoli Sottoscorta", low_stock_count, delta_color="inverse")

st.markdown("---")

# 2. TABELLA MODIFICABILE
st.subheader("Inventario Operativo")

if low_stock_count > 0:
    st.warning(f"⚠️ Attenzione: {low_stock_count} articoli sono sotto la soglia minima. Sono evidenziati nel PDF.")

# Configurazione Editor Tabella
column_config = {
    "Codice": st.column_config.TextColumn("Codice", width="small"),
    "Prodotto": st.column_config.TextColumn("Prodotto", width="medium"),
    "Posizione": st.column_config.TextColumn("Posizione", width="small"),
    "Giacenza": st.column_config.NumberColumn("Giacenza", min_value=0, step=1, help="Doppio click per modificare la quantità"),
    "Minimo": st.column_config.NumberColumn("Minimo", min_value=0, help="Soglia di allarme"),
    "Prezzo Unitario": st.column_config.NumberColumn("Prezzo (€)", format="€ %.2f", min_value=0.0),
}

edited_df = st.data_editor(
    df,
    column_config=column_config,
    use_container_width=True,
    num_rows="dynamic", # Permette di aggiungere/cancellare righe
    key="inventory_editor",
    height=500
)

# Aggiornamento automatico se l'utente modifica la tabella
if not edited_df.equals(st.session_state.inventory):
    st.session_state.inventory = edited_df
    st.rerun()

# 3. EXPORT
st.markdown("### Reportistica")
col_pdf, _ = st.columns([1, 4])

with col_pdf:
    try:
        pdf_bytes = generate_pdf(edited_df)
        st.download_button(
            label="Scarica Inventario PDF",
            data=pdf_bytes,
            file_name="Inventario_Magazzino.pdf",
            mime="application/pdf",
            use_container_width=True
        )
    except Exception as e:
        st.error(f"Errore generazione PDF: {e}")
