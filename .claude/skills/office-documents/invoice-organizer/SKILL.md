---
name: invoice-organizer
description: Auto-organize invoices and receipts for tax preparation. Extract vendor, amount, date, and category. Rename files consistently and generate expense summaries for accounting.
---

# Invoice Organizer

Extract, categorize, and organize financial documents for tax preparation and bookkeeping.

---

## 1. Extraction Schema

For every invoice/receipt, extract:

```json
{
  "vendor": "Company name",
  "date": "YYYY-MM-DD",
  "amount": 99.00,
  "currency": "USD",
  "tax_amount": 8.91,
  "category": "Software/Hardware/Travel/Meals/Office/Professional Services/Advertising",
  "description": "Brief description of what was purchased",
  "invoice_number": "INV-001 (if present)",
  "payment_method": "Credit card/Bank transfer/PayPal",
  "deductible": true,
  "original_filename": "receipt.pdf"
}
```

---

## 2. File Naming Convention

```
Format: YYYY-MM-DD_Vendor-Name_Amount-USD_Category.ext

Examples:
2025-01-15_AWS-Inc_247.50-USD_Software.pdf
2025-02-03_Delta-Airlines_412.00-USD_Travel.pdf
2025-03-22_Slack-Technologies_80.00-USD_Software.pdf
2025-04-01_Office-Depot_34.12-USD_Office-Supplies.pdf

Rules:
- Date: ISO format YYYY-MM-DD (sorts chronologically)
- Vendor: capitalize each word, hyphens not spaces
- Amount: include currency code
- Category: from standard list below
- Extension: keep original (.pdf, .png, .jpg)
```

---

## 3. Category Standards

```
Software          — SaaS subscriptions, licenses, cloud services
Hardware          — Computers, peripherals, equipment
Cloud/Hosting     — AWS, GCP, Azure, Vercel, hosting fees
Professional Svcs — Legal, accounting, consulting, contractors
Marketing/Ads     — Google Ads, Meta Ads, sponsorships, PR
Travel            — Flights, hotels, car rental
Meals             — Business meals, coffee meetings
Office Supplies   — Stationery, printer ink, small items
Utilities         — Internet, phone, electricity (home office %)
Education         — Courses, books, conferences, training
Insurance         — Business insurance premiums
Banking           — Bank fees, merchant fees, payment processing
Other             — Anything that doesn't fit above
```

---

## 4. Batch Processing Workflow

```python
import os, json, datetime
from pathlib import Path

def organize_invoices(input_dir: str, output_dir: str):
    """
    Process all PDFs/images in input_dir:
    1. Extract data from each file
    2. Rename to standard format
    3. Move to output_dir/YYYY/Category/
    4. Return summary CSV
    """
    results = []
    for file in Path(input_dir).glob("**/*"):
        if file.suffix.lower() not in ['.pdf', '.png', '.jpg', '.jpeg']:
            continue
        
        # Extract (using Claude vision or PDF parser)
        data = extract_invoice_data(file)
        
        # Build new filename
        new_name = f"{data['date']}_{data['vendor'].replace(' ', '-')}_{data['amount']}-{data['currency']}_{data['category'].replace('/', '-')}{file.suffix}"
        
        # Destination: output/2025/Software/
        dest_dir = Path(output_dir) / data['date'][:4] / data['category']
        dest_dir.mkdir(parents=True, exist_ok=True)
        
        # Move file
        new_path = dest_dir / new_name
        file.rename(new_path)
        
        results.append({**data, 'new_path': str(new_path)})
    
    return results
```

---

## 5. Expense Summary Report

After processing, generate:

```markdown
# Expense Summary: [Year/Quarter]

## Total Expenses: $[X,XXX.XX]

## By Category
| Category | Count | Total | Tax Deductible |
|---|---|---|---|
| Software | 12 | $2,340.00 | Yes (100%) |
| Hardware | 3 | $1,890.00 | Yes (100%) |
| Travel | 8 | $3,420.00 | Yes (50–100%) |
| Meals | 15 | $890.00 | Yes (50%) |
| ...total | 38 | $8,540.00 | |

## By Month
| Month | Total |
|---|---|
| January | $1,200.00 |
| February | $890.00 |
| ...

## Flagged for Review
- 3 receipts with unclear vendor name
- 2 receipts with missing date
- 1 receipt with amount >$500 (needs approval documentation)

## Tax-Deductible Total
Estimated deductible: $7,420.00 (varies by jurisdiction and business use %)
Note: Consult your accountant for final figures.
```

---

## 6. Rules

- **Never delete originals**: Rename and move, always keep backup of original
- **Flag ambiguous items**: Don't guess category for unusual expenses — mark as "Review"
- **Currency conversion**: Record in original currency + USD equivalent at date of purchase
- **Receipts vs invoices**: Receipts = proof of payment; Invoices = request for payment (both needed for audit)
- **Retention**: Keep all tax-related documents for 7 years (US IRS standard)
