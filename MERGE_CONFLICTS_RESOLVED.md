# Merge Conflicts and Resolutions

This document describes the manual merge performed from `/Users/deepankarrane/Desktop/weburbs-cursor` to `/Users/deepankarrane/Desktop/weburbs-ens/weburbs`.

## Important Note
This was a **manual merge**, not a git merge, so there were no traditional merge conflict markers (<<<<<<, =======, >>>>>>). Instead, differences between the two codebases were manually resolved.

## Key Conflicts/Decisions Made

### 1. **security/views.py** - MAJOR DIFFERENCE

**Conflict:**
- **Source (weburbs-cursor)**: Simple registration with NO email verification (76 lines)
- **Target (weburbs-ens)**: Complex registration WITH email verification (196 lines)

**Resolution:** 
- ✅ **KEPT target's email verification features** (as requested)
- ✅ **ADDED error handling improvements** from source:
  - Added try/except for email sending failures
  - Added auto-verify in development mode when email not configured
  - Added better error logging

**Code Location:**
- Lines 93-121: Enhanced error handling in registration
- Lines 164-172: Enhanced error handling in resend_mail

### 2. **backend/projects/models.py**

**Conflict:**
- **Source**: TransmissionType enum has `pipe = 2`
- **Target**: TransmissionType enum only has `hvac = 1`

**Resolution:**
- ✅ **ADDED `pipe = 2`** to TransmissionType enum

**Code:**
```python
class TransmissionType(IntEnum):
    hvac = 1
    pipe = 2  # ← Added from source
```

### 3. **backend/projects/api/commodity.py**

**Conflict:**
- **Source**: Uses `get_or_create()` to prevent race conditions
- **Target**: Uses direct `Commodity()` creation which can cause duplicates

**Resolution:**
- ✅ **REPLACED** with `get_or_create()` approach from source
- ✅ **ADDED** duplicate prevention logic
- ✅ **ADDED** cleanup functions

### 4. **backend/projects/api/configupload.py**

**Conflict:**
- **Source**: Checks `if "transmission" in dataCommodity and dataCommodity["transmission"] is not None:`
- **Target**: Only checks `if "transmission" in dataCommodity:`

**Resolution:**
- ✅ **FIXED** to check for `None` values to prevent creating empty transmissions
- ✅ **ADDED** duplicate prevention when loading transmissions
- ✅ **ADDED** error handling for missing sites/commodities

### 5. **backend/projects/migrations/**

**Conflict:**
- **Source**: Has migrations `0010_alter_transmission_type.py` and `0011_add_commodity_unique_constraint.py`
- **Target**: Has migration `0010_defproject_defprojectloaded.py`

**Resolution:**
- ✅ **CREATED** migrations as `0011_alter_transmission_type.py` (after target's 0010)
- ✅ **CREATED** migrations as `0012_add_commodity_unique_constraint.py` (after 0011)

### 6. **Frontend Components - Missing Duplicate Functionality**

**Conflict:**
- **Source**: Has duplicate buttons/functionality for ALL entity types (commodity, process, storage, transmission)
- **Target**: Has NO duplicate functionality at all

**Resolution:**
- ✅ **ADDED** duplicate functionality to ALL components:
  - CommodityOverviewComponent
  - ProcessOverviewComponent
  - StorageOverviewComponent
  - TransmissionConfig
- ✅ **ADDED** duplicate API functions:
  - useDuplicateCommodity
  - useDuplicateProcess
  - useDuplicateStorage
  - useDuplicateTransmission
- ✅ **UPDATED** TransformerComponent to show copy buttons

### 7. **Backend API Endpoints - Missing Duplicate Endpoints**

**Conflict:**
- **Source**: Has duplicate endpoints for process and storage
- **Target**: Missing duplicate endpoints for process and storage

**Resolution:**
- ✅ **ADDED** `duplicate_process()` function to process.py
- ✅ **ADDED** `duplicate_storage()` function to storage.py
- ✅ **ADDED** URL routes for both endpoints

### 8. **docker-compose.yaml**

**Conflict:**
- User explicitly requested: **DO NOT CHANGE docker-compose.yaml**

**Resolution:**
- ✅ **LEFT UNCHANGED** as requested (even though source had differences)

## Files Modified (22 files, +710 insertions, -53 deletions)

### Backend:
1. `backend/backendurbs/settings.py` - Added email configuration
2. `backend/projects/models.py` - Added pipe type to TransmissionType
3. `backend/projects/api/commodity.py` - Added duplicate prevention, cleanup functions
4. `backend/projects/api/configupload.py` - Fixed transmission loading
5. `backend/projects/api/process.py` - Added duplicate_process function
6. `backend/projects/api/storage.py` - Added duplicate_storage function
7. `backend/projects/api/transmission.py` - Added validation, debug endpoint, duplicate function
8. `backend/projects/urls.py` - Added cleanup and duplicate endpoints
9. `backend/security/views.py` - Enhanced error handling (kept email verification)
10. `backend/projects/migrations/0011_alter_transmission_type.py` - NEW
11. `backend/projects/migrations/0012_add_commodity_unique_constraint.py` - NEW

### Frontend:
12. `frontend/src/backend/commodities.ts` - Added useDuplicateCommodity
13. `frontend/src/backend/processes.ts` - Added useDuplicateProcess
14. `frontend/src/backend/storage.ts` - Added useDuplicateStorage
15. `frontend/src/backend/transmission.ts` - Added useDuplicateTransmission
16. `frontend/src/components/CommodityOverviewComponent.vue` - Added duplicate events
17. `frontend/src/components/ProcessOverviewComponent.vue` - Added duplicate events
18. `frontend/src/components/StorageOverviewComponent.vue` - Added duplicate events
19. `frontend/src/components/TransformerComponent.vue` - Added copy button
20. `frontend/src/pages/projectConfigs/CommodityConfig.vue` - Added duplicate handler
21. `frontend/src/pages/projectConfigs/ProcessConfig.vue` - Added duplicate handler
22. `frontend/src/pages/projectConfigs/StorageConfig.vue` - Added duplicate handler
23. `frontend/src/pages/projectConfigs/TransmissionConfig.vue` - Added duplicate handler

### Documentation:
24. `DUPLICATE_COMMODITY_FIX.md` - NEW documentation file

## Migration Conflicts Resolved

**Problem:** Target had migration 0010 for `defproject_defprojectloaded`, but source had different 0010 for `alter_transmission_type`.

**Resolution:** Renumbered source migrations to 0011 and 0012 to avoid conflicts.

## Database Conflicts Resolved

**Problem:** Migration `0012_add_commodity_unique_constraint` failed because existing duplicate commodities in database.

**Resolution:** 
1. Ran cleanup script to remove 14 duplicate commodities
2. Successfully applied migration

## Summary

All conflicts were resolved by:
- Keeping target project's advanced features (email verification, project presets)
- Adding source project's bug fixes and improvements (duplicate prevention, validation)
- Merging frontend duplicate functionality from source
- Adding missing backend endpoints from source
- Preserving user's request to not change docker-compose.yaml





