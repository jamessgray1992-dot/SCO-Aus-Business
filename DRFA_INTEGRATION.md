# DRFA Intelligence Module — Integration Guide
## For AUS BD & CRM (index.html)

### File Structure (Git repo)
```
your-repo/
├── index.html          ← main AUS BD & CRM app (existing)
├── drfa.html            ← DRFA Intelligence module (NEW)
├── README.md
└── assets/ (if any)
```

### Git Commands
```bash
# From your project directory:
git add drfa.html
git add DRFA_INTEGRATION.md   # this file (optional)
git commit -m "feat: add DRFA Intelligence module with AUS-wide view, scoring engine, and past/present/future timeline"
git push origin main
```

---

## PATCHES TO index.html

### PATCH 1: Add "AUS" to australianStates (add at the TOP of the object, before QLD)

Find this line:
```javascript
const australianStates = {
    QLD: {
```

Replace with:
```javascript
const australianStates = {
    AUS: {
        name: 'Australia Wide',
        code: 'AUS',
        color: '#4F46E5',
        population: 26500000,
        lgaCount: 537,
        zones: [
            { id: 'all', name: 'All States' }
        ],
        regions: []  // AUS aggregates all state regions dynamically
    },
    QLD: {
```

### PATCH 2: Update getCurrentRegions() to support AUS-wide

Find this function:
```javascript
function getCurrentRegions() {
    return getCurrentStateData().regions;
}
```

Replace with:
```javascript
function getCurrentRegions() {
    if (currentState === 'AUS') {
        // Aggregate all regions from all states
        const allRegions = [];
        Object.entries(australianStates).forEach(([code, state]) => {
            if (code === 'AUS') return;
            state.regions.forEach(r => {
                allRegions.push({ ...r, stateCode: code, stateName: state.name });
            });
        });
        return allRegions;
    }
    return getCurrentStateData().regions;
}
```

### PATCH 3: Update switchState() to handle AUS

Find this in switchState():
```javascript
function switchState(stateCode) {
    if (stateCode === currentState) return;
    
    currentState = stateCode;
    currentRegion = null;
```

After `currentRegion = null;` add:
```javascript
    // For AUS-wide: skip Firebase state-specific loading, show aggregated view
    if (stateCode === 'AUS') {
        database.tenders = [];
        database.crmContacts = [];
        updateStateUI();
        renderRegionTabs();
        populateRegionFilters();
        updateAllStats();
        
        // Load tenders from ALL states
        Promise.all(
            Object.keys(australianStates)
                .filter(s => s !== 'AUS')
                .map(s => db.collection('tenders').where('state', '==', s).get())
        ).then(snapshots => {
            database.tenders = snapshots.flatMap(snap => 
                snap.docs.map(d => ({ id: d.id, ...d.data() }))
            );
            // Load CRM from all states
            return Promise.all(
                Object.keys(australianStates)
                    .filter(s => s !== 'AUS')
                    .map(s => db.collection('crmContacts').where('state', '==', s).get())
            );
        }).then(crmSnaps => {
            database.crmContacts = crmSnaps.flatMap(snap =>
                snap.docs.map(d => ({ id: d.id, ...d.data() }))
            );
            updateAllStats();
            if (currentMainTab === 'crm') renderCRMContacts();
            if (currentMainTab === 'analytics') renderAnalytics();
        });
        
        showToast('Switched to Australia Wide view', 'success');
        return;
    }
```

### PATCH 4: Add DRFA feature flag

Find in FEATURE_LABELS:
```javascript
const FEATURE_LABELS = {
    tenders:        { label: 'Tenders & Opportunities',  desc: 'Main tenders tracking tab' },
```

Add this entry:
```javascript
const FEATURE_LABELS = {
    tenders:        { label: 'Tenders & Opportunities',  desc: 'Main tenders tracking tab' },
    drfa:           { label: 'DRFA Intelligence',        desc: 'Disaster Recovery Funding Analysis tool' },
```

And in the BRAND features object:
```javascript
features: {
    tenders:         true,
    drfa:            true,    // ← ADD THIS
    crm:             true,
```

### PATCH 5: Add DRFA nav tab (in the HTML navigation section)

Find the nav pills section (inside the `<nav>` element, where tab buttons are listed).
Add this button alongside the existing tabs:

```html
<button id="tab-drfa" class="nav-pill main-tab px-4 py-2 rounded-lg text-sm font-medium transition-all"
        onclick="window.open('drfa.html?state=' + currentState, '_self')">
    <i class="fas fa-water mr-1.5" style="font-size:11px;"></i> DRFA Intel
</button>
```

### PATCH 6: Add DRFA link in state selector grid

In the `renderStateSelector()` function, after the state grid cards, add a DRFA quick-access card:

```javascript
// After the state cards loop, append:
grid.innerHTML += `
    <div class="bg-gradient-to-br from-indigo-50 to-purple-50 rounded-xl p-6 shadow-sm cursor-pointer hover:shadow-md transition-all border-2 border-indigo-100"
         onclick="window.open('drfa.html','_self')">
        <div class="flex items-center justify-between mb-4">
            <div class="w-14 h-14 rounded-lg flex items-center justify-center text-white font-bold text-xl shadow-lg"
                 style="background:linear-gradient(135deg,#1e3a5f,#4f46e5);">
                DR
            </div>
            <div class="text-right">
                <div class="text-2xl font-bold text-indigo-900">DRFA</div>
                <div class="text-xs text-indigo-500">Intelligence</div>
            </div>
        </div>
        <h3 class="font-bold text-indigo-900 text-lg mb-1">DRFA Intelligence</h3>
        <p class="text-sm text-indigo-500">Flood funding analysis & tender targeting</p>
    </div>
`;
```

### PATCH 7: Deep-link support (drfa.html reads state from AUS BD & CRM)

The drfa.html already supports URL params:
- `drfa.html?state=QLD` — opens filtered to QLD
- `drfa.html?state=VIC&region=Northern%20VIC` — filtered to VIC Northern region
- `drfa.html?lga=24` — opens Campaspe detail drawer

The nav tab button in PATCH 5 automatically passes `?state=currentState` so the DRFA tool opens in context.

---

## ADDITIONAL SUGGESTIONS (my recommendations)

### 1. Firebase Collection for DRFA Data
Add a `drfaLGAs` Firestore collection so DRFA data persists and syncs across users:

```javascript
// In drfa.html — replace DRFA_DATA with Firestore loading:
async function loadDRFAData() {
    const snap = await db.collection('drfaLGAs').get();
    if (snap.empty) {
        // First run: seed from DRFA_DATA
        for (const lga of DRFA_DATA) {
            await db.collection('drfaLGAs').doc(String(lga.id)).set(lga);
        }
        return DRFA_DATA;
    }
    return snap.docs.map(d => ({ id: +d.id, ...d.data() }));
}
```

### 2. Auto-Create Tender from DRFA Opportunity
Add a "Create Tender" button in the DRFA detail drawer that auto-populates a tender in AUS BD & CRM:

```javascript
// In drfa.html drawer:
function createTenderFromDRFA(lga) {
    const params = new URLSearchParams({
        action: 'createTender',
        title: `DRFA Recovery - ${lga.name}`,
        region: lga.regionId,
        state: lga.state,
        value: lga.est_value,
        source: 'DRFA',
        notes: `DRFA opportunity. Score: ${lga.score}. ${lga.notes}`
    });
    window.location.href = `index.html?${params.toString()}`;
}
```

### 3. DRFA Event Alerts in AUS BD & CRM Dashboard
Add a small DRFA alert widget to the main dashboard:

```javascript
// In index.html dashboard rendering:
function renderDRFAAlerts() {
    const activeCount = 12; // Pull from Firestore drfaLGAs where active=true
    return `
        <div class="bg-indigo-50 rounded-xl p-4 border border-indigo-100 cursor-pointer"
             onclick="window.open('drfa.html?active=true','_self')">
            <div class="flex items-center gap-3">
                <div class="w-10 h-10 rounded-lg flex items-center justify-center"
                     style="background:linear-gradient(135deg,#1e3a5f,#4f46e5);">
                    <i class="fas fa-water text-white text-sm"></i>
                </div>
                <div>
                    <div class="text-sm font-bold text-indigo-900">${activeCount} Active DRFA Events</div>
                    <div class="text-xs text-indigo-500">Click to view opportunities</div>
                </div>
            </div>
        </div>
    `;
}
```

### 4. DRFA Scoring in CRM Contacts
Enrich CRM council contacts with their DRFA score:

When displaying a council contact in the CRM, show their DRFA tier:
```javascript
// In renderCRMContacts():
if (contact.category === 'Council') {
    // Look up DRFA score for this council's region
    const drfaMatch = DRFA_DATA.find(l => l.regionId === contact.regions?.[0]);
    if (drfaMatch) {
        cardHTML += `<span class="tier-badge tier-${drfaMatch.tier}">DRFA ${TIERS[drfaMatch.tier].label}</span>`;
    }
}
```

### 5. Kanban Integration
Auto-create Kanban cards from DRFA opportunities:
- When a DRFA event triggers → auto-create a "Lead" card in Kanban
- As the opportunity progresses → drag through Pre-Tender → Tendering → Awarded

### 6. Export to Spreadsheet
Add CSV/XLSX export from the DRFA table so you can share with estimators:
```javascript
function exportDRFACSV() {
    const data = getFiltered();
    const headers = ['Rank','LGA','State','Region','Score','Tier','Phase','Stage','Est Value','Flood Freq','Active'];
    const rows = data.map((l,i) => [i+1, l.name, l.state, l.region, l.score, 'Tier '+l.tier, l.timeline, l.stage, l.est_value, l.flood_freq, l.active]);
    const csv = [headers, ...rows].map(r => r.join(',')).join('\n');
    const blob = new Blob([csv], { type:'text/csv' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url; a.download = 'DRFA_Intelligence_Export.csv'; a.click();
}
```

---

## Summary of what's included

| Feature | Status |
|---------|--------|
| AUS-wide state aggregation | ✅ Ready (patches above) |
| State-by-state filtering | ✅ Built into drfa.html |
| Region filtering | ✅ Built in |
| DRFA scoring engine (0-100) | ✅ Implemented |
| Tier classification (1/2/3) | ✅ Implemented |
| Past / Present / Future timeline | ✅ Built in |
| AI search button (placeholder) | ✅ Wired — needs backend |
| Detail drawer with score breakdown | ✅ Implemented |
| Value for money indicators | ✅ Included |
| AUS BD & CRM region ID linking | ✅ regionId mapped |
| Deep-link URL params | ✅ Implemented |
| Git-ready file structure | ✅ Single HTML file |
| Feature flag integration | ✅ Patch provided |
| Firebase sync (recommended) | 📋 Guide provided |
| Auto-create tenders from DRFA | 📋 Guide provided |
| Dashboard alerts widget | 📋 Guide provided |
| CSV export | 📋 Guide provided |
