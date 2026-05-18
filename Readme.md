![AI Trading Agent screenshot](./Screenshot.png)

# Schwab API — AI Agent

A Python application that connects an **AI agent** to the Charles Schwab brokerage API via [schwabdev](https://github.com/tylerebowers/Schwabdev), served through a **NiceGUI** web interface. The agent supports natural-language queries about market data, account information, and trade execution using a ReAct tool-call loop across multiple LLM providers.

## Disclaimer

This project is provided **as-is** for **educational and personal experimentation only**. It is **not** financial, investment, legal, or tax advice. Trading and investing involve risk of loss; you are solely responsible for your decisions, your account activity, and for complying with Charles Schwab's API terms of service, your brokerage agreements, and applicable laws and regulations.

The authors and contributors make **no warranty** regarding accuracy, availability, or fitness for a particular purpose and **disclaim liability** for any losses or damages arising from use of this software. Outputs from the AI agent may be incorrect, incomplete, or outdated — **verify everything** before acting, and do not rely on the agent alone to place or manage trades.

---

## AI Agent

The agent (`src/agent.py`) runs a **ReAct loop**: it sends your message to the LLM, receives tool calls, executes them against the Schwab API, feeds the results back, and repeats until a final answer is produced.

### Supported LLM providers

Configure **one** API key in `.env` and select the provider in the sidebar:

| Provider | Env variable | Notes |
|---|---|---|
| Google Gemini | `GEMINI_API_KEY` | Default; models: `gemini-3.1-pro-preview`, `gemini-3-flash-preview`, `gemini-3.1-flash-lite-preview` |
| OpenAI | `OPENAI_API_KEY` | Models: `gpt-5.5`, `gpt-5.4`, `gpt-5.4-mini` |
| Anthropic Claude | `ANTHROPIC_API_KEY` | Models: `claude-opus-4-7`, `claude-sonnet-4-6`, `claude-haiku-4-5-20251001` |

### Agent tools

Tools are grouped into three categories. Each can be **enabled or disabled individually** in the sidebar before or during a conversation.

#### Market Data
| Tool | Description |
|---|---|
| `get_quote` | Real-time quote for a single symbol |
| `get_quotes` | Real-time quotes for multiple symbols |
| `get_option_chain` | Option chain filtered by type, strike count, date range, or DTE |
| `get_option_expirations` | All available expiration dates for a ticker |
| `get_price_history` | Historical OHLCV candles (minute to monthly) |
| `get_movers` | Top movers for an index ($SPX, $DJI, NASDAQ, and more) |
| `get_market_hours` | Trading hours for equity, option, bond, future, or forex markets |
| `search_instruments` | Search by symbol or description |
| `get_instrument_by_cusip` | Instrument lookup by CUSIP |
| `create_stock_chart` | Render an interactive candlestick + volume chart in the UI |

#### Account Info
| Tool | Description |
|---|---|
| `get_linked_accounts` | All linked account numbers and hashes |
| `get_account_details` | Balances (and optionally positions) for the primary account |
| `get_all_account_details` | Balances for all linked accounts |
| `get_positions` | All open positions |
| `get_account_orders` | Order history for the primary account |
| `get_all_orders` | Order history across all accounts |
| `get_order_details` | Full details for a specific order ID |
| `get_transactions` | Trades, dividends, ACH, and other transactions |
| `get_transaction_details` | Full details for a specific transaction ID |
| `get_preferences` | User preferences and streaming configuration |
| `preview_order` | Preview an order without placing it |

#### Trading *(disabled by default — enable explicitly in the sidebar)*
| Tool | Description |
|---|---|
| `buy_stock` | Place a limit buy order for equities |
| `sell_stock` | Place a limit sell order for equities |
| `cancel_order` | Cancel a pending order by ID |

### Skills (quick actions)

Skills are pre-built prompt templates surfaced as sidebar buttons for common tasks:

| Skill | Action |
|---|---|
| **Chart Stock** | Candlestick + volume chart for any symbol |
| **Get Quote** | Current price for a symbol |
| **My Positions** | All open positions |
| **Recent Orders** | Orders placed in the account |
| **Buy Stock** | Guided limit-buy prompt |

---

## Setup

### Requirements

- Python 3.10+
- A Schwab developer app with completed OAuth/token flow (via `schwabdev`)
- At least one LLM API key (Gemini, OpenAI, or Anthropic)

### Install

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### Environment variables

Create a `.env` file in the project root:

```dotenv
# Schwab credentials
APP_KEY=your_schwab_app_key
APP_SECRET=your_schwab_app_secret

# Web UI
APP_PASSWORD=your_ui_login_password
NICEGUI_STORAGE_SECRET=a_long_random_string
NICEGUI_PORT=8080            # optional, default 8080

# LLM — set at least one
GEMINI_API_KEY=...
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...
```

OAuth tokens are stored in **`.nicegui/schwab_tokens.db`**. On first run, `schwabdev` will prompt you to complete the OAuth flow in a browser.

---

## Run

```bash
python app.py
```

Open **http://localhost:8080**, sign in with `APP_PASSWORD`, and navigate to **AI Agent** in the sidebar drawer.

---

## Project layout

```
app.py                       # entry point
nicegui_app.py               # NiceGUI + FastAPI app, auth middleware, routes
src/
  agent.py                   # ReAct agent loop (Claude / OpenAI / Gemini)
  agent_tools.py             # ALL_TOOLS registry — Schwab API tool definitions
  skills.py                  # SKILLS registry — quick-action shortcuts
  client.py                  # schwabdev client singleton + token management
  account.py                 # account info helpers
  orders.py                  # order submission helpers
  spx_chart.py               # chart rendering
  app_logging_config.py      # rotating file logger
ui/
  nicegui_agent.py           # AI Agent chat page
  nicegui_account.py         # Account page
  nicegui_option_chains.py   # Option chains page
  nicegui_chart.py           # Chart page
```

---

## References

- schwabdev: [https://github.com/tylerebowers/Schwabdev](https://github.com/tylerebowers/Schwabdev)
- NiceGUI: [https://nicegui.io/documentation](https://nicegui.io/documentation)
- Earlier Streamlit-oriented walkthrough (may not match current UI): [YouTube](https://youtu.be/J-8m5j3Zshs)
