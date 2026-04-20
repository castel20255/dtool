# App ID Consolidation - Summary

## Objective
Consolidate all app IDs to use **113831 (Staging)** across all environments and remove all other app IDs (121856, 114784).

## Changes Made

### 1. Configuration File Updates
**File**: `/src/components/shared/utils/config/config.ts`

#### APP_IDS Object
All environment IDs now map to 113831:
```typescript
export const APP_IDS = {
    LOCALHOST: 113831,          // Changed from 121856
    TMP_STAGING: 113831,        // Already was 113831
    STAGING: 113831,            // Already was 113831
    STAGING_BE: 113831,         // Already was 113831
    STAGING_ME: 113831,         // Already was 113831
    PRODUCTION: 113831,         // Changed from 121856
    PRODUCTION_BE: 113831,      // Changed from 114784
    PRODUCTION_ME: 113831,      // Changed from 114784
    VERCEL: 113831,             // Changed from 121856
};
```

#### Domain App IDs
```typescript
'profithubtool.vercel.app': '113831'  // Changed from '121856'
```

#### OAuth URL Generation
```typescript
// Vercel Staging OAuth configuration
if (hostname === 'profithubtool.vercel.app') {
    const lang = window.localStorage.getItem('lang') || 'EN';
    return `https://oauth.deriv.com/oauth2/authorize?app_id=113831&l=${lang}&brand=deriv`;
}
```

### 2. Login Module Updates
**File**: `/src/components/shared/utils/login/login.ts`

#### Vercel OAuth Endpoint
```typescript
// Changed Vercel Production reference to Vercel Staging
if (window.location.hostname === 'profithubtool.vercel.app') {
    return `https://oauth.deriv.com/oauth2/authorize?app_id=113831&l=${language}&brand=deriv`;
}
```

### 3. API Reference Documentation
**File**: `/DERIV_API_REFERENCE.md`

Updated all documentation references to reflect single app ID:
- Removed multiple app ID entries
- Updated connection examples to use 113831
- Clarified all environments use Staging API (113831)

### 4. Mobile Responsiveness Fixes (Previously Applied)
**Files Modified**:
- `/src/pages/smart-trading/components/scp-tab.scss`
- `/src/pages/free-bots/free-bots-tab.scss`
- `/src/pages/signals/signals-tab.scss`
- `/src/pages/auto-trader/auto-trader.scss`

**Changes**:
- Added responsive media queries for tablet (768px) and mobile (480px) breakpoints
- Scaled fonts, padding, and grid layouts for smaller screens
- Ensured all components fit properly on mobile devices
- Fixed header sizes and spacing issues

## Verification

All instances of old app IDs have been removed:
- ✅ 121856 removed (Localhost/Production/Vercel)
- ✅ 114784 removed (Production Backend/Middle-End)
- ✅ All references now use 113831 (Staging)

**Grep verification results**:
```
src/components/shared/utils/config/config.ts:    LOCALHOST: 113831
src/components/shared/utils/config/config.ts:    STAGING: 113831
src/components/shared/utils/config/config.ts:    PRODUCTION: 113831
src/components/shared/utils/config/config.ts:    PRODUCTION_BE: 113831
src/components/shared/utils/config/config.ts:    PRODUCTION_ME: 113831
src/components/shared/utils/config/config.ts:    VERCEL: 113831
src/components/shared/utils/config/config.ts:    'profithubtool.vercel.app': '113831'
src/components/shared/utils/login/login.ts:    app_id=113831
```

## Deriv API Configuration

### WebSocket Connection
All connections now use:
```
wss://ws.derivws.com/websockets/v3?app_id=113831
```

### OAuth Flow
All OAuth redirects use:
```
https://oauth.deriv.com/oauth2/authorize?app_id=113831&l={language}&brand=deriv
```

## Impact

- **All environments** (localhost, staging, production, vercel) now connect to Deriv's staging API with app ID 113831
- **Single source of truth** for app ID configuration
- **Simplified deployment** - no environment-specific app ID logic needed
- **Consistent OAuth experience** across all deployment domains

## Notes

- The app ID configuration is dynamically resolved through `getAppId()` function
- Environment variables (VITE_APP_ID, REACT_APP_Deriv_APP_ID) take priority if set
- LocalStorage overrides available for testing (`config.app_id`)
- OAuth domain selection still respects domain-specific configurations (deriv.com, deriv.me, deriv.be)

## Files Modified

1. ✅ `/src/components/shared/utils/config/config.ts` - APP_IDS, domain_app_ids, OAuth URLs
2. ✅ `/src/components/shared/utils/login/login.ts` - Vercel OAuth endpoint
3. ✅ `/DERIV_API_REFERENCE.md` - Documentation updates
4. ✅ Mobile responsiveness fixes in SCSS files

## Testing Recommendations

1. Test OAuth login on all deployment domains
2. Verify WebSocket connection to staging API
3. Confirm authorization flow with staging app ID
4. Test on mobile devices (tablet and mobile sizes)
5. Verify responsive layouts on all tabs

---

**Consolidation Status**: ✅ Complete
**Date**: 2026-04-20
