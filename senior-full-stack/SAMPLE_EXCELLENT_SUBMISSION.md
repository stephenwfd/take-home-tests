# Sample Excellent Submission
## What a ~80-point candidate looks like

This document shows the **structure and thinking** of a strong submission. Not the actual code (too long), but the approach and rationale.

---

## Directory Structure (Excellent Candidate)

```
renewal-risk-system/
├── backend/
│   ├── src/
│   │   ├── api/
│   │   │   ├── routes/
│   │   │   │   ├── renewal-risk.ts
│   │   │   │   └── index.ts
│   │   │   ├── middleware/
│   │   │   │   └── error-handler.ts
│   │   │   └── controllers/
│   │   │       └── renewal-risk.controller.ts
│   │   ├── db/
│   │   │   ├── connection.ts
│   │   │   └── migrations/
│   │   │       ├── 001_renewal_risk_tables.sql
│   │   │       └── 002_webhook_state_tables.sql
│   │   ├── services/
│   │   │   ├── renewal-risk.service.ts
│   │   │   └── webhook-delivery.service.ts
│   │   ├── webhooks/
│   │   │   ├── delivery.ts
│   │   │   ├── retry.ts
│   │   │   └── idempotency.ts
│   │   ├── types/
│   │   │   └── index.ts
│   │   ├── config.ts
│   │   └── index.ts
│   ├── seed/
│   │   └── seed.ts
│   ├── package.json
│   ├── tsconfig.json
│   ├── .env.example
│   └── README.md
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── RenewalRiskDashboard.tsx
│   │   │   ├── RiskTable.tsx
│   │   │   └── RiskSignalDetail.tsx
│   │   ├── pages/
│   │   │   └── PropertyDashboard.tsx
│   │   ├── api/
│   │   │   └── client.ts
│   │   ├── types/
│   │   │   └── index.ts
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── package.json
│   ├── vite.config.ts
│   └── README.md
├── docker-compose.yml
├── .gitignore
└── README.md (root)
```

**Rationale for structure:**
- Clear separation of concerns (API, services, webhooks)
- Frontend components are small and focused
- Database migrations are version-controlled
- Configuration is external (env files)
- Type definitions are centralized

---

## Root README (Excellent)

```markdown
# Renewal Risk Detection System for ROP

## Quick Start

### Prerequisites
- Docker & Docker Compose
- Node.js 18+

### Setup & Run (5 minutes)

1. **Start database:**
   ```bash
   docker-compose up -d postgres
   sleep 5
   ```

2. **Set up backend:**
   ```bash
   cd backend
   npm install
   npm run migrate
   npm run seed
   npm run dev
   ```
   Should output: `Server running on http://localhost:3000`

3. **Set up frontend (new terminal):**
   ```bash
   cd frontend
   npm install
   npm run dev
   ```
   Opens http://localhost:5173

### Test the Feature

**1. Calculate renewal risk:**
```bash
# Get a propertyId from seed output first
PROPERTY_ID="..." 

curl -X POST http://localhost:3000/api/v1/properties/$PROPERTY_ID/renewal-risk/calculate \
  -H "Content-Type: application/json" \
  -d '{"propertyId": "'$PROPERTY_ID'", "asOfDate": "2025-01-02"}'
```

Expected: 200 OK with risk scores for residents

**2. Open dashboard:**
Open http://localhost:5173/properties/$PROPERTY_ID/renewal-risk

Should show a table of at-risk residents with risk scores and reasons.

**3. Test webhook delivery:**
Option A (quick):
- Use https://webhook.site
- Copy your URL
- Set `RMS_ENDPOINT=https://webhook.site/... npm run dev` in backend
- Click "Trigger Renewal Event" on a resident
- Check webhook.site for delivery

Option B (local testing):
- The repo includes a mock RMS in `backend/mock-rms.js`
- Run `node backend/mock-rms.js` (listens on :3001)
- Configure `RMS_ENDPOINT=http://localhost:3001/webhook`

## Architecture Decisions

### Schema Design

**Renewal Risk Scores Table:**
```sql
renewal_risk_scores (
  id, property_id, resident_id, risk_score, risk_tier,
  days_to_expiry, payment_delinquent, no_renewal_offer,
  rent_above_market, calculated_at, created_at
)
```

**Rationale:**
- Denormalized slightly (includes signals) to avoid join when displaying dashboard
- Indexed on (property_id, calculated_at) for efficient "get latest risks" queries
- Signals stored atomically so we know exactly what factors contributed to each score
- Calculated_at lets us track freshness (can identify stale scores)

**Webhook Delivery State Table:**
```sql
webhook_delivery_state (
  id, event_id (idempotency key), property_id, resident_id,
  status, attempt_count, last_attempt_at, next_retry_at,
  rms_response, created_at, updated_at
)

webhook_dead_letter_queue (
  id, webhook_delivery_state_id, reason, created_at
)
```

**Rationale:**
- Event_id is the idempotency key (prevents duplicate deliveries)
- Status tracks state machine: pending → delivered | dlq
- Exponential backoff calculated in next_retry_at (1s, 2s, 4s, 8s, 16s)
- DLQ is separate table so failed webhooks are visible for support/debugging
- Indexed on (status, next_retry_at) for efficient retry job queries

### Risk Scoring Algorithm

**Calculation (0-100 scale):**
```
days_to_expiry_score = min(days_to_expiry / 180 * 100, 100)  # 180 days = max risk
payment_score = delinquent ? 25 : 0
renewal_offer_score = no_offer ? 20 : 0
market_rent_score = (market_rent - actual_rent) / actual_rent > 0.05 ? 15 : 0

total = days_to_expiry_score * 0.40 + payment_score * 0.25 + renewal_offer_score * 0.20 + market_rent_score * 0.15
```

**Rationale:**
- Days to expiry is the strongest signal (40%). Close expiration = high risk.
- Payment history matters (25%). Delinquent residents are less likely to renew.
- Renewal offers matter (20%). If we haven't offered renewal, residents may leave.
- Market rent matters (15%). If market rent >> their rent, they may shop around.

**Edge cases handled:**
- Month-to-month residents: days_to_expiry = 30 (treated as "at risk")
- No market rent data: assume no gap (market_rent_score = 0)
- Lease already expired: status != 'active', not included in calculation

### Webhook Delivery System

**Happy path (2-second requirement):**
1. API receives "Trigger Renewal Event"
2. Create webhook_delivery_state with status='pending'
3. POST to RMS endpoint asynchronously (don't wait for response)
4. When response arrives, update status and attempt_count
5. If successful (2xx), status='delivered' ✓
6. If failed, calculate next_retry_at and save for retry job

**Retry job (runs every 30 seconds):**
- Query for all webhooks with status='pending' AND next_retry_at < NOW()
- Retry each one (POST with same event_id for idempotency)
- Update attempt_count
- If attempt_count >= 5, move to DLQ
- If successful, mark as delivered

**Idempotency:**
- Event ID is UUID (unique per resident/event combination)
- RMS should use event_id to deduplicate (idempotency key)
- If we see the same event_id twice, we don't retry (we check if delivered first)

**Request signing:**
- HMAC-SHA256 of payload with shared secret
- RMS can validate that webhook came from us (no tampering)
- Header: `X-Webhook-Signature: sha256=...`

**p95 latency:**
- Async delivery means API responds immediately (< 100ms)
- Retry job spreads load (doesn't block API)
- p95 webhook delivery = first attempt latency (< 2s to RMS endpoint)

### Query Performance

**Risk calculation query:**
```sql
-- Efficient query for scoring 5000 residents in one pass
WITH resident_data AS (
  SELECT 
    r.id, r.property_id, r.unit_id,
    l.lease_end_date,
    CURRENT_DATE - l.lease_end_date as days_past_expiry,
    CASE WHEN l.lease_end_date <= CURRENT_DATE THEN 30 
         ELSE LEAST((l.lease_end_date - CURRENT_DATE), 180) END as days_to_expiry,
    COALESCE(MAX(CASE WHEN rl.transaction_type = 'charge' AND rl.transaction_date > CURRENT_DATE - INTERVAL '90 days' 
                  AND NOT EXISTS (SELECT 1 FROM resident_ledger WHERE resident_id = r.id AND transaction_type = 'payment' 
                                  AND transaction_date > rl.transaction_date - INTERVAL '7 days')
                  THEN 1 ELSE 0 END), 0) as payment_delinquent,
    CASE WHEN NOT EXISTS (SELECT 1 FROM renewal_offers WHERE resident_id = r.id AND status = 'pending') 
         THEN 1 ELSE 0 END as no_renewal_offer,
    COALESCE(up.market_rent, 0) - COALESCE(l.monthly_rent, 0) as rent_gap
  FROM residents r
  JOIN leases l ON r.id = l.resident_id AND l.status = 'active'
  JOIN units u ON r.unit_id = u.id
  LEFT JOIN resident_ledger rl ON r.id = rl.resident_id
  LEFT JOIN unit_pricing up ON u.id = up.unit_id AND up.effective_date = CURRENT_DATE
  WHERE r.property_id = $1
  GROUP BY r.id, l.id, up.market_rent
)
SELECT 
  id, property_id,
  LEAST(
    (days_to_expiry::DECIMAL / 180 * 100) * 0.40 +
    (payment_delinquent * 25) * 0.25 +
    (no_renewal_offer * 20) * 0.20 +
    (CASE WHEN rent_gap > l.monthly_rent * 0.05 THEN 15 ELSE 0 END) * 0.15,
    100
  ) as risk_score
FROM resident_data
```

**Rationale:**
- Single query (no N+1)
- Aggregations in CTE to avoid subqueries
- Indexed on property_id and lease dates for fast filtering
- Completes in < 1s even for 5000 residents

### React Component Structure

**RenewalRiskDashboard.tsx (page):**
- Fetches risk scores from API
- Manages loading/error state
- Maps data to RiskTable component

**RiskTable.tsx (component):**
- Displays list of residents
- Color-codes by risk tier (high=red, medium=yellow, low=green)
- Expandable rows show risk signals
- "Trigger Renewal Event" button (disabled while loading)

**RiskSignalDetail.tsx (component):**
- Shows why a resident is at risk
- Displays each signal as a bullet point
- Friendly language ("45 days until lease expires" vs "days_to_expiry: 45")

**UX Decisions:**
- Table-based layout (familiar to property managers)
- Color coding is secondary (text labels are primary, for accessibility)
- Clickable rows expand to show signals (no separate modal)
- "Trigger Event" button is obvious (large, distinct color)
- Errors are shown inline (toast notification)

## Testing

### Automated
```bash
npm run test  # Run test suite (if you have time)
```

### Manual
```bash
# See TESTING_GUIDE.md in this repo
```

## Known Limitations & Future Work

### Phase 1 (what's here)
- ✓ Risk scoring based on simple rules
- ✓ Webhook retry with exponential backoff
- ✓ Idempotency (no duplicate deliveries)
- ✓ React dashboard (minimal, functional)

### Phase 2 (future)
- [ ] ML-based risk scoring (instead of rules)
- [ ] Pagination for large resident lists
- [ ] Filters by risk tier, days to expiry
- [ ] Bulk actions ("Trigger event for all high-risk residents")
- [ ] Webhook signature verification (RMS validates request)
- [ ] Metrics/observability (track delivery success rate)
- [ ] Unit tests for scoring logic

### What I'd do differently with more time
1. Add integration tests (currently just manual testing)
2. Move webhook retry to a background job (Bull queue or similar)
3. Add pagination to dashboard (50 residents per page)
4. Implement request signing for webhook security
5. Add metrics (delivery latency, success rate, DLQ size)
6. Database connection pooling (PgBouncer or similar)

## Agentic Development

**Parts generated with Claude (Cursor):**
- Seed script boilerplate (repetitive data generation)
- React component styling (Tailwind classes)
- Error handling middleware (standard pattern)

**Parts I wrote manually:**
- Renewal risk scoring algorithm (custom business logic)
- Webhook retry logic (critical reliability code)
- SQL queries (need to optimize for multi-tenancy)
- API endpoint design (represents our assumptions about the feature)

**Why this split:**
- AI is great at boilerplate and standard patterns
- I wrote the code that matters most (business logic, reliability)
- This ensures I understand every critical path

## Questions?

This is a minimal but functional feature. It demonstrates:
- ✓ Full-stack shipping (API + React + SQL)
- ✓ Scalability thinking (webhooks, retries, idempotency)
- ✓ Pragmatism under time pressure (80% in 2 hours)
- ✓ Clear code and decision-making

Happy to discuss any questions in Round 3.
```

---

## Sample API Implementation (Key Routes)

**renewal-risk.controller.ts (highlights):**
```typescript
export async function calculateRenewalRisk(req: Request, res: Response) {
  const { propertyId, asOfDate } = req.body;
  
  // Validate
  if (!propertyId || !asOfDate) {
    return res.status(400).json({ error: 'propertyId and asOfDate required' });
  }
  
  try {
    const scores = await RenewalRiskService.calculateForProperty(propertyId, asOfDate);
    
    res.json({
      propertyId,
      calculatedAt: asOfDate,
      totalResidents: scores.total,
      flaggedCount: scores.flagged.length,
      riskTiers: { high: scores.high, medium: scores.medium, low: scores.low },
      flags: scores.flagged
    });
  } catch (error) {
    res.status(500).json({ error: 'Failed to calculate risk' });
  }
}

export async function triggerRenewalEvent(req: Request, res: Response) {
  const { propertyId, residentId } = req.body;
  
  try {
    // Create webhook_delivery_state record
    const webhookState = await db.query(`
      INSERT INTO webhook_delivery_state (event_id, property_id, resident_id, status, attempt_count)
      VALUES ($1, $2, $3, 'pending', 0)
      RETURNING *
    `, [generateEventId(), propertyId, residentId]);
    
    // Trigger async delivery (don't wait)
    WebhookDeliveryService.deliverAsync(webhookState.rows[0]);
    
    res.json({ status: 'scheduled', webhookId: webhookState.rows[0].id });
  } catch (error) {
    res.status(500).json({ error: 'Failed to trigger event' });
  }
}
```

**webhook-delivery.service.ts (highlights):**
```typescript
export class WebhookDeliveryService {
  static async deliverAsync(webhookState: WebhookState) {
    try {
      const response = await fetch(process.env.RMS_ENDPOINT, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'X-Webhook-Signature': generateSignature(webhookState.payload),
          'X-Event-ID': webhookState.event_id
        },
        body: JSON.stringify(webhookState.payload)
      });
      
      if (response.ok) {
        await db.query('UPDATE webhook_delivery_state SET status = $1 WHERE id = $2', 
          ['delivered', webhookState.id]);
      } else {
        await this.scheduleRetry(webhookState);
      }
    } catch (error) {
      await this.scheduleRetry(webhookState);
    }
  }
  
  static async scheduleRetry(webhookState: WebhookState) {
    const nextRetry = this.calculateBackoff(webhookState.attempt_count);
    await db.query(`
      UPDATE webhook_delivery_state 
      SET status = $1, attempt_count = $2, next_retry_at = NOW() + INTERVAL '1 second' * $3
      WHERE id = $4
    `, ['pending', webhookState.attempt_count + 1, nextRetry, webhookState.id]);
    
    if (webhookState.attempt_count + 1 >= 5) {
      await db.query(`
        INSERT INTO webhook_dead_letter_queue (webhook_delivery_state_id, reason)
        VALUES ($1, $2)
      `, [webhookState.id, 'max_retries_exceeded']);
    }
  }
  
  static calculateBackoff(attemptCount: number): number {
    // 1s, 2s, 4s, 8s, 16s
    return Math.pow(2, attemptCount);
  }
}
```

---

## Sample React Component (Key Parts)

**RenewalRiskDashboard.tsx:**
```typescript
export const RenewalRiskDashboard: React.FC<{ propertyId: string }> = ({ propertyId }) => {
  const [risks, setRisks] = useState<RiskScore[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const loadRisks = async () => {
      try {
        const response = await fetch(`/api/v1/properties/${propertyId}/renewal-risk/calculate`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ propertyId, asOfDate: new Date().toISOString().split('T')[0] })
        });
        
        if (!response.ok) throw new Error('Failed to load risks');
        
        const data = await response.json();
        setRisks(data.flags);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    loadRisks();
  }, [propertyId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div style={{ color: 'red' }}>Error: {error}</div>;

  return (
    <div>
      <h1>At-Risk Renewals</h1>
      <RiskTable risks={risks} propertyId={propertyId} />
    </div>
  );
};
```

---

## Sample Schema (Key Tables)

```sql
-- Renewal Risk Scores (denormalized for performance)
CREATE TABLE renewal_risk_scores (
  id UUID PRIMARY KEY,
  property_id UUID NOT NULL REFERENCES properties(id),
  resident_id UUID NOT NULL REFERENCES residents(id),
  risk_score DECIMAL(5,2) NOT NULL CHECK (risk_score >= 0 AND risk_score <= 100),
  risk_tier VARCHAR(20) NOT NULL, -- high, medium, low
  days_to_expiry INT,
  payment_delinquent BOOLEAN,
  no_renewal_offer BOOLEAN,
  rent_above_market BOOLEAN,
  calculated_at DATE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(property_id, resident_id, calculated_at)
);

CREATE INDEX idx_renewal_risk_scores_property_id_calculated_at 
  ON renewal_risk_scores(property_id, calculated_at DESC);

-- Webhook Delivery State
CREATE TABLE webhook_delivery_state (
  id UUID PRIMARY KEY,
  event_id UUID NOT NULL UNIQUE, -- Idempotency key
  property_id UUID NOT NULL REFERENCES properties(id),
  resident_id UUID NOT NULL REFERENCES residents(id),
  status VARCHAR(20) NOT NULL, -- pending, delivered, dlq
  attempt_count INT DEFAULT 0,
  last_attempt_at TIMESTAMP,
  next_retry_at TIMESTAMP,
  rms_response TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_webhook_delivery_state_status_next_retry 
  ON webhook_delivery_state(status, next_retry_at) 
  WHERE status = 'pending';

-- Dead Letter Queue
CREATE TABLE webhook_dead_letter_queue (
  id UUID PRIMARY KEY,
  webhook_delivery_state_id UUID NOT NULL REFERENCES webhook_delivery_state(id),
  reason VARCHAR(255),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Summary: What Makes This "Excellent"

1. **Complete**: Backend API, database schema, React dashboard, webhooks all work
2. **Thoughtful**: Risk scoring is well-designed, schema is normalized+denormalized thoughtfully
3. **Reliable**: Webhooks have retry logic, idempotency, DLQ
4. **Performant**: Queries are optimized (no N+1), indexes are strategic
5. **Clear**: Code is organized, decisions are documented, README is comprehensive
6. **Pragmatic**: Delivered 80% in 2 hours, clearly identified future work

**Scoring: 80-85 points**

---

This is what you should see in a strong submission. Not perfect (no tests, some rough edges), but solid shipping engineering.
