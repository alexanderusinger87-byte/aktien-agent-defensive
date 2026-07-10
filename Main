import streamlit as st
import yfinance as yf
import pandas as pd

# -----------------------------
# Defensive KPI Scoring Functions (Konservativere Schwellenwerte)
# -----------------------------

def score_roe(roe):
    if roe is None: return 0
    if roe > 18: return 10
    if roe > 12: return 8
    if roe > 8: return 6
    if roe > 4: return 4
    return 1

def score_peg(peg):
    # Bei defensiven Aktien weniger kritisch, aber zu hoch ist spekulativ
    if peg is None: return 0
    if peg < 1.2: return 10
    if peg < 1.8: return 8
    if peg < 2.5: return 5
    return 1

def score_operating_margin(m):
    # Hohe operative Margen bedeuten Preismacht in der Inflation
    if m is None: return 0
    if m > 25: return 10
    if m > 18: return 8
    if m > 10: return 6
    return 2

def score_forward_pe(fpe):
    # Strenge Bestrafung von High-Multiple-Risiken
    if fpe is None: return 0
    if fpe < 12: return 10
    if fpe < 16: return 8
    if fpe < 22: return 5
    return 1

def score_pe(pe):
    if pe is None: return 0
    if pe < 14: return 10
    if pe < 20: return 8
    if pe < 26: return 5
    return 1

def score_eps_growth(g):
    # Defensiv = Stetigkeit vor Hyperwachstum
    if g is None: return 0
    if g > 12: return 10
    if g > 7: return 8
    if g > 3: return 6
    return 2

def score_revenue_growth(g):
    if g is None: return 0
    if g > 10: return 10
    if g > 6: return 8
    if g > 2: return 6
    return 2

def score_profit_margin(m):
    if m is None: return 0
    if m > 20: return 10
    if m > 12: return 8
    if m > 6: return 6
    return 2

def score_debt_to_equity(d):
    # Extrem wichtig für Risiko-Minimierung (Verschuldung)
    if d is None: return 0
    if d < 0.25: return 10  # Kaum Schulden
    if d < 0.50: return 8
    if d < 0.80: return 5
    return 1  # Hohes Risiko im Niedrigzins-Ende

def score_cash_to_debt(c):
    if c is None: return 0
    if c > 2.0: return 10
    if c > 1.2: return 8
    if c > 0.6: return 6
    return 1

def score_fcf_yield(y):
    # FCF schützt vor Bilanzkosmetik
    if y is None: return 0
    if y > 7: return 10
    if y > 5: return 8
    if y > 3: return 6
    return 1

# -----------------------------
# Risiko-Averse Defensive Gewichtung (Summe = 1.0)
# -----------------------------
DEFENSIVE_WEIGHTS = {
    "operating_margin": 0.15,  # Preismacht schützt in Krisen
    "roe": 0.13,               # Solide Substanzrendite
    "forward_pe": 0.13,        # Keine Überbezahlung zukünftiger Gewinne
    "debt_to_equity": 0.12,    # Schutz vor Überschuldung / steigenden Zinsen
    "fcf_yield": 0.12,         # Echte Cash-Generierung
    "pe": 0.10,                # Aktuelle Bewertung
    "cash_to_debt": 0.10,      # Liquiditätspuffer
    "eps_growth": 0.05,        # Geringere Relevanz von Hype-Wachstum
    "revenue_growth": 0.05,    # Geringere Relevanz von Hype-Wachstum
    "profit_margin": 0.05,     # Abgesichert durch Operating Margin
    "peg": 0.00,               # PEG fliegt raus (wird oft durch krasse Wachstums-Prognosen verzerrt)
}

# -----------------------------
# Streamlit UI
# -----------------------------
st.set_page_config(page_title="Defensiver Aktienanalyse Agent", layout="wide")

st.title("🛡️ Defensiver Aktienanalyse Agent – Low-Risk KPI Score")
st.write("Dieses Modell gewichtet finanzielle Stabilität, Cashflows und moderate Bewertungen höher, um risikoarme Qualitätsaktien zu identifizieren.")

tickers_input = st.text_input("Gib mehrere Ticker ein (getrennt durch Komma, z.B. JNJ, PG, PEP, MSFT):")

if tickers_input:
    tickers = [t.strip().upper() for t in tickers_input.split(",")]
    results = []

    with st.spinner("Lade Fundamentaldaten..."):
        for ticker in tickers:
            try:
                stock = yf.Ticker(ticker)
                info = stock.info
                
                if not info or "symbol" not in info:
                    continue

                # Kennzahlen ziehen
                pe = info.get("trailingPE")
                fpe = info.get("forwardPE")
                peg = info.get("pegRatio")
                profit_margin = info.get("profitMargins")
                operating_margin = info.get("operatingMargins")
                roe = info.get("returnOnEquity")
                revenue_growth = info.get("revenueGrowth")
                eps_growth = info.get("earningsQuarterlyGrowth")
                debt_to_equity = info.get("debtToEquity")
                
                total_cash = info.get("totalCash", 0)
                total_debt = info.get("totalDebt", 1)
                cash_to_debt = total_cash / total_debt if total_debt and total_debt > 0 else 2.0

                fcf = info.get("freeCashflow")
                market_cap = info.get("marketCap")
                fcf_yield = (fcf / market_cap) * 100 if fcf and market_cap else None

                # Skalierung
                if profit_margin: profit_margin *= 100
                if operating_margin: operating_margin *= 100
                if roe: roe *= 100
                if revenue_growth: revenue_growth *= 100
                if eps_growth: eps_growth *= 100

                # Score-Berechnung (Defensive Variante)
                score = (
                    score_operating_margin(operating_margin) * DEFENSIVE_WEIGHTS["operating_margin"] +
                    score_roe(roe) * DEFENSIVE_WEIGHTS["roe"] +
                    score_forward_pe(fpe) * DEFENSIVE_WEIGHTS["forward_pe"] +
                    score_debt_to_equity(debt_to_equity) * DEFENSIVE_WEIGHTS["debt_to_equity"] +
                    score_fcf_yield(fcf_yield) * DEFENSIVE_WEIGHTS["fcf_yield"] +
                    score_pe(pe) * DEFENSIVE_WEIGHTS["pe"] +
                    score_cash_to_debt(cash_to_debt) * DEFENSIVE_WEIGHTS["cash_to_debt"] +
                    score_eps_growth(eps_growth) * DEFENSIVE_WEIGHTS["eps_growth"] +
                    score_revenue_growth(revenue_growth) * DEFENSIVE_WEIGHTS["revenue_growth"] +
                    score_profit_margin(profit_margin) * DEFENSIVE_WEIGHTS["profit_margin"]
                )

                results.append({
                    "Ticker": ticker,
                    "Defensiv-Score": round(score * 10, 2),
                    "Operating Margin (%)": round(operating_margin, 2) if operating_margin else None,
                    "ROE (%)": round(roe, 2) if roe else None,
                    "Debt/Equity": round(debt_to_equity, 2) if debt_to_equity else None,
                    "FCF Yield (%)": round(fcf_yield, 2) if fcf_yield else None,
                    "Forward P/E": round(fpe, 2) if fpe else None,
                    "P/E (KGV)": round(pe, 2) if pe else None,
                    "Cash/Debt": round(cash_to_debt, 2),
                    "Revenue Growth (%)": round(revenue_growth, 2) if revenue_growth else None
                })
            except Exception as e:
                st.error(f"Fehler bei {ticker}: {str(e)}")

    if results:
        df = pd.DataFrame(results)
        st.subheader("📊 Auswertung nach Defensiv-Stärke")
        df = df.sort_values(by="Defensiv-Score", ascending=False)
        st.dataframe(df, use_container_width=True)
