# Thundercat Scan Insights - External Share Link Mockups

**Related Epic:** [AX-1425 - External Share Link - TC Scan Reporting](https://frontgatetickets.atlassian.net/browse/AX-1425)

## Overview

These mockups demonstrate how external clients can access scan insights data from Thundercat without requiring full system access. The solution provides read-only, time-limited access to specific festival scan reports.

## Files Created

### 1. `thundercat-scan-insights-external-mockup.html`
**Purpose:** Basic external share view without authentication

**Features:**
- Clean, branded interface with "External Share" banner
- Festival information header
- Key metrics dashboard (Total Scans, Success Rate, Failed Scans, etc.)
- Interactive date range filters
- Visual charts for scan trends
- Detailed breakdown by ticket group
- Responsive design for mobile/tablet viewing
- No authentication required (suitable for public links with obscure URLs)

**Use Case:** Quick share links for clients who need immediate access without login friction.

### 2. `thundercat-scan-insights-external-with-auth.html`
**Purpose:** Secure external share with token-based authentication

**Features:**
- Token-based access control
- Access token input screen
- Link expiration notice
- Same comprehensive dashboard as version 1
- Session storage for token persistence
- URL parameter support for token (`?token=DEMO-2026-FESTIVAL-56`)

**Demo Token:** `DEMO-2026-FESTIVAL-56`

**Use Case:** More secure sharing for sensitive data, with ability to revoke access by invalidating tokens.

## Key Design Decisions

### 1. **Minimal Branding**
- Kept Thundercat branding subtle (small logo, "Powered by Thundercat" text)
- Focuses on the data, not the platform
- Purple gradient accent colors for visual consistency

### 2. **Read-Only Interface**
- Clear banner indicating external/shared status
- No edit controls or navigation to other Thundercat features
- Self-contained reports with all necessary context

### 3. **Mobile-First Responsive Design**
- Grid layouts that collapse on smaller screens
- Touch-friendly interactive elements
- Readable typography at all sizes

### 4. **Security Considerations**
- Token-based access (version 2)
- Expiration dates displayed prominently
- No sensitive configuration or system data exposed
- Session-based token storage (not persistent)

## Metrics Displayed

The mockups include these key performance indicators:

| Metric | Description |
|--------|-------------|
| **Total Scans** | Aggregate scan count for the period |
| **Successful Scans** | Scans that processed successfully |
| **Failed Scans** | Scans that encountered errors |
| **Peak Scans/Hour** | Highest throughput period |
| **Unique Tickets** | Distinct tickets scanned |
| **Avg Scan Time** | Average processing duration |

## Charts & Visualizations

1. **Scans by Day** - Daily scan volume trends
2. **Scans by Entry Point** - Breakdown by gate/location
3. **Detailed Table** - Group-level statistics with success rates

## Implementation Recommendations

### Backend Requirements

```javascript
// Example API endpoint structure
GET /api/external-share/:shareToken
Response:
{
  "festival": {
    "id": 56,
    "name": "Summer Music Festival 2026",
    "location": "Austin, TX",
    "startDate": "2026-06-11",
    "endDate": "2026-06-14"
  },
  "metrics": {
    "totalScans": 47823,
    "successfulScans": 46891,
    "failedScans": 932,
    "peakScansPerHour": 3247,
    "uniqueTickets": 18456,
    "avgScanTime": 1.2
  },
  "scansByDay": [...],
  "scansByEntryPoint": [...],
  "scansByGroup": [...],
  "expiresAt": "2026-07-31T23:59:59Z"
}
```

### Database Schema Addition

```sql
CREATE TABLE external_share_links (
  id UUID PRIMARY KEY,
  token VARCHAR(255) UNIQUE NOT NULL,
  festival_id INT NOT NULL,
  created_by INT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  expires_at TIMESTAMP NOT NULL,
  is_active BOOLEAN DEFAULT true,
  access_count INT DEFAULT 0,
  last_accessed_at TIMESTAMP,
  allowed_metrics JSON,  -- Optional: restrict what data is visible
  FOREIGN KEY (festival_id) REFERENCES festivals(id),
  FOREIGN KEY (created_by) REFERENCES users(id)
);

CREATE INDEX idx_external_share_token ON external_share_links(token);
CREATE INDEX idx_external_share_festival ON external_share_links(festival_id);
```

### Security Features to Implement

1. **Token Generation**
   - Use cryptographically secure random tokens (e.g., `crypto.randomBytes(32)`)
   - Format: `FEST-{festivalId}-{randomString}`

2. **Access Control**
   - Rate limiting (e.g., 100 requests/hour per token)
   - IP allowlisting (optional)
   - Audit logging of all access

3. **Expiration Handling**
   - Default expiration: 30 days from creation
   - Automatic cleanup of expired tokens
   - Email notifications before expiration

4. **Revocation**
   - Admin ability to revoke tokens immediately
   - Bulk revocation by festival or date range

### Frontend Integration

The external share page should be served from a dedicated route that:
- Does NOT require Thundercat authentication
- Has minimal dependencies (no full app bundle)
- Loads quickly even on slow connections
- Works without JavaScript (progressive enhancement)

**Suggested Route:** `https://thundercat-app.production.fgtix.io/share/{token}`

### URL Structure Options

**Option 1: Token in Path** (Recommended)
```
https://thundercat-app.production.fgtix.io/share/FEST-56-abc123def456
```
- Cleaner URLs
- Easier to copy/paste
- Better for email/SMS sharing

**Option 2: Token in Query Parameter**
```
https://thundercat-app.production.fgtix.io/share?token=FEST-56-abc123def456
```
- Easier to add additional parameters
- Familiar pattern for many users

## User Flow

### Creating a Share Link (Admin)

1. Admin navigates to Scan Insights dashboard
2. Clicks "Share" button (new feature to add)
3. Modal appears with options:
   - Expiration date selector
   - Optional: Restrict specific metrics
   - Optional: Add email recipients
4. System generates unique token
5. Admin receives shareable link
6. Optional: System emails link to recipients

### Accessing a Shared Report (External Client)

**Flow A: Direct Access (No Auth)**
1. Client clicks link
2. Report loads immediately
3. Banner indicates this is a shared/read-only view

**Flow B: With Token Auth**
1. Client clicks link
2. Prompted for access token (if not in URL)
3. Enters token
4. Report loads
5. Token stored in session for convenience

## Testing the Mockups

### Open in Browser

```bash
# Option 1: Direct open
open ~/thundercat-scan-insights-external-mockup.html

# Option 2: With auth screen
open ~/thundercat-scan-insights-external-with-auth.html

# Option 3: With token pre-filled
open ~/thundercat-scan-insights-external-with-auth.html?token=DEMO-2026-FESTIVAL-56
```

### Local Server (for testing with CORS/API)

```bash
# Python 3
cd ~
python3 -m http.server 8000

# Then visit:
# http://localhost:8000/thundercat-scan-insights-external-mockup.html
```

## Next Steps

### Phase 1: Backend API
- [ ] Create `external_share_links` table
- [ ] Implement token generation endpoint
- [ ] Create share link validation middleware
- [ ] Build data aggregation endpoint for shared reports

### Phase 2: Admin UI
- [ ] Add "Share" button to Scan Insights dashboard
- [ ] Create share link management modal
- [ ] Implement link creation form
- [ ] Add link management/revocation interface

### Phase 3: External Share Page
- [ ] Create dedicated route for external shares
- [ ] Implement token validation
- [ ] Build data loading from API
- [ ] Add responsive chart library (Chart.js, Recharts, etc.)
- [ ] Implement real-time updates (optional)

### Phase 4: Additional Features
- [ ] Email/SMS delivery of share links
- [ ] PDF export functionality
- [ ] Custom branding per client (white-label)
- [ ] Multiple report templates
- [ ] Scheduled report delivery

## Design Inspiration

The mockups draw inspiration from:
- **Figma Design:** https://www.figma.com/make/0aWNioMi3MujW9R7XxAtbh/External-Share
- **Existing Dashboard:** https://thundercat-app.production.fgtix.io/dashboards/scan-insights

## Questions & Considerations

1. **Data Refresh Rate:** Should shared reports show live data or snapshots?
   - **Recommendation:** Snapshot at time of share creation, with optional refresh
   
2. **Branding:** Should clients be able to white-label reports?
   - **Recommendation:** Start with standard branding, add custom branding as premium feature
   
3. **Export Formats:** PDF, CSV, Excel?
   - **Recommendation:** Start with HTML view, add PDF export in Phase 4
   
4. **Analytics:** Track which metrics external clients view most?
   - **Recommendation:** Yes, implement view tracking for product insights

## Contact

For questions about this mockup or the implementation:
- **Epic Owner:** Annaliese Walsten
- **Jira Epic:** [AX-1425](https://frontgatetickets.atlassian.net/browse/AX-1425)
