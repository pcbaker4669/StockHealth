import requests
import json
import pandas as pd

# Replace with your actual API key
API_KEY = "LVKBwEZrAsQJ5ZKhPuDRY62BQ7zfvoSU"

def read_stock_symbols(file_path):
    with open(file_path, "r") as file:
        return [line.strip() for line in file.readlines()]

def get_fundamental_data(ticker):
    url = f"https://api.polygon.io/vX/reference/financials?ticker={ticker}&apiKey={API_KEY}"
    response = requests.get(url)
    data = response.json()

    if "results" in data and len(data["results"]) > 0:
        financials = data["results"][0].get("financials", {})

        # Extract data safely with defaults
        revenue = financials.get("income_statement", {}).get("revenues", {}).get("value", None)
        net_income = financials.get("income_statement", {}).get("net_income_loss", {}).get("value", None)
        liabilities = financials.get("balance_sheet", {}).get("liabilities", {}).get("value", None)
        equity = financials.get("balance_sheet", {}).get("equity", {}).get("value", None)
        long_term_debt = financials.get("balance_sheet", {}).get("long_term_debt", {}).get("value", None)
        gross_profit = financials.get("income_statement", {}).get("gross_profit", {}).get("value", None)

        # Handle missing data by setting defaults
        if not revenue or not net_income or not equity or not liabilities:
            print(f"⚠️ Warning: Missing key financials for {ticker}. Skipping.")
            return None

        # Calculate derived financial metrics
        net_margin = round((net_income / revenue) * 100, 2) if revenue and net_income else None
        debt_to_equity = round(liabilities / equity, 2) if liabilities and equity else None
        roe = round((net_income / equity) * 100, 2) if equity and net_income else None

        # Print extracted values for debugging
        print(f"\n[Ticker: {ticker}]")
        print(f"Revenue: {revenue}")
        print(f"Net Income: {net_income}")
        print(f"Net Margin: {net_margin}%")
        print(f"Liabilities: {liabilities}")
        print(f"Equity: {equity}")
        print(f"Debt-to-Equity: {debt_to_equity}")
        print(f"ROE: {roe}%")
        print(f"Gross Profit: {gross_profit}")

        return {
            "revenue": revenue,
            "net_margin": net_margin,
            "debt_to_equity": debt_to_equity,
            "roe": roe,
            "gross_profit": gross_profit
        }
    else:
        print(f"⚠️ Warning: No financial data found for {ticker}")
        return None

def calculate_health_score(financials, ticker):
    if not financials:
        return 0

    revenue = financials["revenue"]
    net_margin = financials["net_margin"]
    debt_to_equity = financials["debt_to_equity"]
    roe = financials["roe"]

    # Assign scores (higher is better)
    rev_score = min(max((revenue / 1e9) * 2, 0), 20) if revenue else 0
    margin_score = min(max(net_margin * 1.5, 0), 20) if net_margin else 0
    debt_score = min(20 - min(debt_to_equity * 2, 20), 20) if debt_to_equity else 0
    roe_score = min(roe * 0.5, 20) if roe else 0

    score = rev_score + margin_score + debt_score + roe_score
    return round(score, 2)

def analyze_stocks(input_file, output_file):
    tickers = read_stock_symbols(input_file)
    results = []

    for ticker in tickers:
        print(f"Processing {ticker}...")
        financials = get_fundamental_data(ticker)

        if financials:
            score = calculate_health_score(financials, ticker)
        else:
            score = 0  # Assign zero score for missing data

        results.append({"Ticker": ticker, "Health Score (%)": score})

    df = pd.DataFrame(results)
    df.to_csv(output_file, index=False)
    print(f"\n✅ Analysis complete. Results saved to {output_file}")

# Run the analysis
analyze_stocks("my_stocks.txt", "stock_health_report.csv")