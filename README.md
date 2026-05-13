# n8n-nodes-erpnext-selling

Planned n8n community node package for ERPNext Selling.

Planned resources:

- Customer
- Lead
- Opportunity
- Quotation
- Sales Order
- Sales Partner
- Contact
- Address

## Node Identity

All `n8n2erpnext` module nodes use the same ERPNext-style logo shape. Each module changes only the main background color.

Selling uses commerce teal `#00A6A6` because the module represents customer-facing pipeline, quotations, sales orders, and revenue movement.

Full module color map:

| Module | Color | Hex |
| --- | --- | --- |
| Core | ERPNext blue | `#2490EF` |
| HRMS | People green | `#2E7D5F` |
| Accounting | Finance orange-red | `#D94A2B` |
| Buying | Procurement amber | `#C47F00` |
| Selling | Commerce teal | `#00A6A6` |
| Stock | Frappe black | `#171717` |

When building this module, copy the HRMS/Accounting SVG structure and change only the main background fill to `#00A6A6`.
