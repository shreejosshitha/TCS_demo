# AI-Powered Pricing and Combo Offer Support System

Essential working prototype of an **AI-Powered Pricing and Combo Offer Support System** designed for a college canteen.

The system helps a canteen manager:
1. **Discover food combinations** from empirical order baskets using FP-Growth and Apriori association rule mining.
2. **Estimate product demand** dynamically responsive to candidate selling prices, day-of-week, and promotions using Scikit-Learn.
3. **Simulate pricing scenarios** calculating revenue, variable costs (ingredients + packaging), and contribution before fixed costs.
4. **Evaluate candidate combo offers** assessing customer savings, combined margins, student affordability limits, and stock bottlenecks.
5. **Generate transparent recommendations** to optimize contribution while enforcing budget and operational constraints.

---

## 1. Project Architecture

```
TCS/
├── backend/
│   ├── main.py                  # FastAPI server with REST endpoints & static frontend mount
│   ├── data_loader.py           # Loads CSVs and calculates catalog metrics
│   ├── combo_discovery.py       # Association rule mining (Apriori & FP-Growth)
│   ├── demand_model.py          # Random Forest demand estimation & chronological split
│   ├── pricing_engine.py        # Single product candidate price scenario evaluation
│   ├── combo_evaluator.py       # Combo offer economics, savings & bottleneck checks
│   ├── recommendation_engine.py # Constrained optimization & recommendation generation
│   ├── schemas.py               # Pydantic request/response validation schemas
│   ├── test_api.py              # Automated integration tests for all API endpoints
│   └── requirements.txt         # Backend Python dependencies
├── data/
│   ├── products.csv             # 8 products with cost structure, prices & stock
│   ├── transactions.csv         # 8,372 transaction lines across 4,531 order baskets (90 days)
│   └── daily_sales.csv          # 720 product-day sales, prices, promotions & inventory
├── frontend/
│   └── static/
│       └── index.html           # Interactive React 18 + Tailwind CSS + Chart.js dashboard
├── scripts/
│   ├── generate_data.py         # Reproducible synthetic data generation script (seed=42)
│   └── validate_data.py         # Dataset validation & integrity check script
└── README.md
```

---

## 2. Dataset Schema & Assumptions

All datasets are **synthetic** college canteen records created for prototype development.

### A. `products.csv`
- `product_id`: Unique identifier (`P101` to `P108`).
- `product_name`: Food / beverage item (Veg Sandwich, Veg Burger, French Fries, Lemon Juice, Cold Coffee, Samosa, Paneer Roll, Water Bottle).
- `category`: Menu grouping (`Snacks`, `Beverages`, `Sides`).
- `current_price`: Baseline selling price in Indian Rupees (₹).
- `ingredient_cost`: Raw food input cost in ₹.
- `packaging_cost`: Disposables / packaging cost in ₹.
- `portion_size_g`: Weight / volume (grams or ml).
- `available_stock`: Daily standard prepared batch capacity (units).
- `waste_quantity`: Daily average unsold/discarded units under standard operations.

### B. `transactions.csv`
- `transaction_id`: Basket identifier (`TX10001` ...).
- `transaction_date`: Date of purchase (90 days from August 2026 to October 2026).
- `product_id` & `product_name`: Item purchased.
- `quantity`: Quantity in line item.
- `unit_price`: Price charged per unit.
- `discount`: Line-level promotional discount (₹) non-negative and `<= subtotal`.

### C. `daily_sales.csv`
- `date`: Daily observation date.
- `product_id`: Product identifier.
- `selling_price`: Tested price on that day.
- `quantity_sold`: Total units sold.
- `day_of_week`: Day of week (Monday through Sunday).
- `promotion_active`: Binary indicator (`1` = promotion/event day, `0` = normal).
- `stock_available`: Daily available inventory.
- `unsold_quantity`: `stock_available - quantity_sold`.

---

## 3. Financial & Operational Formulas

- **Unit Variable Cost**:
  $$\text{Unit Variable Cost} = \text{Ingredient Cost} + \text{Packaging Cost}$$
- **Revenue**:
  $$\text{Revenue} = \text{Candidate Price} \times \text{Estimated Quantity}$$
- **Contribution**:
  $$\text{Contribution} = \text{Revenue} - (\text{Unit Variable Cost} \times \text{Estimated Quantity})$$
  *(Note: Labeled strictly as contribution before fixed canteen rent, electricity, and staff overheads).*
- **Combo Savings**:
  $$\text{Customer Savings} = \sum(\text{Individual Current Prices}) - \text{Proposed Combo Price}$$
- **Combo Unit Contribution**:
  $$\text{Contribution per Combo} = \text{Proposed Combo Price} - \sum(\text{Component Variable Costs})$$
- **Stock Bottleneck**:
  $$\text{Max Combo Capacity} = \min_{i \in \text{Combo}}(\text{Available Stock}_i)$$

---

## 4. Machine Learning & Discovery Methodology

1. **Basket Association Mining (`mlxtend`)**:
   - Converts transaction orders into binary basket incidence matrices.
   - Discovers frequent itemsets via **FP-Growth** or **Apriori**.
   - Filters rules by configurable **Support**, **Confidence**, and **Lift** ($\ge 1.0$).
2. **Demand Estimation (`scikit-learn`)**:
   - Uses **RandomForestRegressor** trained with a **chronological 80/20 train/test split** (no date leakage).
   - Features: Encoded Product ID, Candidate Price, Encoded Day of Week, Promotion Active, Stock Available.
   - Evaluated against a historical average baseline model using Mean Absolute Error (**MAE**).
   - Dynamically responds to price increases/decreases.

---

## 5. Quickstart & How to Run

### Step 1: Install Dependencies
```bash
pip install fastapi uvicorn pandas numpy scikit-learn mlxtend scipy
```

### Step 2: Regenerate & Validate Data (Optional)
```bash
python3 scripts/generate_data.py
python3 scripts/validate_data.py
```

### Step 3: Run Backend Tests
```bash
PYTHONPATH=. python3 backend/test_api.py
```

### Step 4: Launch Application Server
```bash
PYTHONPATH=. python3 -m uvicorn backend.main:app --host 0.0.0.0 --port 8000 --reload
```

Open your browser to:
- **Interactive UI Dashboard**: [http://localhost:8000/](http://localhost:8000/)
- **Interactive API Documentation (Swagger)**: [http://localhost:8000/docs](http://localhost:8000/docs)

---

## 6. API Endpoints Summary

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/health` | Healthcheck and model status |
| `GET` | `/api/overview` | Total transactions, revenue, contribution, top & slow products |
| `GET` | `/api/products` | Complete product catalog with variable costs and stock |
| `GET` | `/api/combos` | Discovered association rules with default parameters |
| `POST` | `/api/combos/discover` | Discovers combinations with custom support, confidence, lift |
| `GET` | `/api/demand-model/metrics` | Model MAE, baseline comparison, and feature importance |
| `POST` | `/api/pricing/evaluate` | Evaluates price candidates for revenue & contribution deltas |
| `POST` | `/api/combos/evaluate` | Evaluates combo pricing, customer savings, and constraints |
| `POST` | `/api/recommendations` | Suggests optimal prices and combos under budget & stock limits |

---

## 7. Prototype Limitations & Validation Guidance

- **Synthetic Data**: Demand elasticities are based on synthetic simulation. In real operations, menu price adjustments should be rolled out via controlled A/B price experiments.
- **Substitution / Cannibalization**: The combo evaluator assumes partial incrementality; customer cannibalization of single items requires panel tracking data.
- **Decision Support**: The system recommends feasible prices; the canteen manager retains final approval.
