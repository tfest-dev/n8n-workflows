# Sample output: daily collections summary

```markdown
# Daily Collections Summary

## Overview
- 3 overdue invoices eligible for action.
- Total overdue: USD 32,800.
- 2 invoices prepared for escalation.
- Overall workflow status: success.

## Buckets
- 7-14 days: 1 invoices (USD 4,200)
- 15-30 days: 0 invoices (USD 0)
- 31-60 days: 1 invoices (USD 9,800)
- 60+ days: 1 invoices (USD 18,800)

## Newly escalated
1. **Beta Inc – inv_1002**
   - Amount: USD 9,800
   - Days overdue: 45
   - Owner: tom@yourcompany.com
   - Action: Account owner follow-up and billing-system note update.
2. **Contoso Ltd – inv_1003**
   - Amount: USD 18,800
   - Days overdue: 66
   - Owner: maya@yourcompany.com
   - Action: Finance lead review and account-owner call task.
```

The final workflow output also includes an `audit_record` with billing-source status, excluded invoices, action results, idempotency keys, finance notification payload and optional LLM prompt metadata.
