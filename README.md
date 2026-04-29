# Meter Reading Invoice
> A custom Odoo 17 module for utility meter-based billing

---

## What This Does

Standard Odoo invoices require manual quantity input.
This module automates billing for utility companies by
calculating consumption from meter readings automatically.

### New Invoice Line Columns

| Column   | Input  | Description                              |
|----------|--------|------------------------------------------|
| New      | Manual | Meter reading entered this month         |
| Previous | Auto   | Fetched from client's last invoice       |
| Actual   | Auto   | Consumption calculated as New - Previous |

Quantity is automatically set to Actual so the invoice
total reflects real consumption.

---

## Compatibility

| Item           | Version        |
|----------------|----------------|
| Odoo           | 17.0 only      |
| Python         | 3.12           |
| OS             | Linux          |
| Database       | PostgreSQL     |

---

## Submission Structure

```
Odoo Assignment/
│
├── README.md
└── meter_invoice.zip
    │
    └── meter_invoice/
        │
        ├── __init__.py                 # loads models package
        ├── __manifest__.py             # module metadata and dependencies
        │
        ├── models/
        │   ├── __init__.py             # loads account_move_line
        │   └── account_move_line.py   # fields, logic and calculations
        │
        ├── views/
        │   └── account_move_views.xml # adds columns to invoice form
        │
        └── report/
            └── invoice_report.xml     # adds columns to PDF report
```

---

## Installation

### Prerequisites

Run these commands to install required system packages:

```bash
sudo apt update

sudo apt install postgresql python3-dev python3-venv git \
libxml2-dev libxslt1-dev libldap2-dev libsasl2-dev \
wkhtmltopdf -y
```

### Step 1 — Create Odoo System User

```bash
sudo useradd -m -d /opt/odoo -U -r -s /bin/bash odoo
```

### Step 2 — Download Odoo 17

```bash
sudo git clone https://github.com/odoo/odoo \
--depth 1 --branch 17.0 /opt/odoo/odoo
```

### Step 3 — Setup Python Environment

```bash
sudo python3.12 -m venv /opt/odoo/venv

sudo /opt/odoo/venv/bin/pip install \
-r /opt/odoo/odoo/requirements.txt
```

### Step 4 — Setup Database

```bash
sudo -u postgres createuser -s odoo
```

### Step 5 — Copy Module

```bash
sudo cp -r meter_invoice /opt/odoo/custom-addons/
```

### Step 6 — Install Module

```bash
sudo -u odoo /opt/odoo/venv/bin/python \
/opt/odoo/odoo/odoo-bin -c /opt/odoo/odoo.conf \
-d your_database -i meter_invoice
```

Replace `your_database` with your actual database name.

### Step 7 — Open Odoo

```
http://localhost:8069
```

---

## Testing

Follow these steps to verify the module works correctly:

**Test 1 — First Invoice (New Client)**
1. Go to Invoicing → Customers → Invoices → New
2. Select any customer
3. Add a product line
4. Enter `1000` in the New field
5. Confirm — Previous should be 0, Actual should be 1000

**Test 2 — Second Invoice (Auto-fill)**
1. Create another invoice for the same customer
2. Add the same product
3. Enter `1500` in the New field
4. Previous should auto-fill as `1000`
5. Actual should calculate as `500`
6. Quantity should sync to `500`
7. Confirm — all values should persist

**Test 3 — PDF Report**
1. On any confirmed invoice click Print → Invoices
2. New, Previous and Actual columns should appear in the PDF

---

## How It Works

```
User enters New reading
        ↓
System searches for client's last posted invoice
        ↓
Previous = last invoice's New value (0 if none exists)
        ↓
Actual = New - Previous
        ↓
Quantity = Actual
        ↓
Total = Quantity × Unit Price
```

---

## Notes

- Previous is read-only — never enter this manually
- Actual is read-only — calculated automatically
- New client with no history → Previous defaults to 0
- Module tested on Odoo 17 only
- Other Odoo versions are not guaranteed to work
