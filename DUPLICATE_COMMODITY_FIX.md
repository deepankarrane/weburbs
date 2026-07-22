# Duplicate Commodity Bug Fix

## Problem Description

The application was experiencing a bug where duplicate commodities with the same name could be automatically created in the same site, causing errors when:
- Trying to delete one of the duplicates
- Creating new transmissions
- Performing other operations that expect unique commodity names

## Root Cause Analysis

The issue was caused by:

1. **Race Conditions**: Multiple automatic commodity creation paths could run simultaneously:
   - Site creation (when `DefCommodity.autoadd=True`)
   - Process addition (when referencing non-existent commodities)
   - Storage addition (when referencing non-existent commodities)

2. **Missing Database Constraints**: The `Commodity` model lacked a unique constraint on `(site, name)` combination

3. **Inadequate Error Handling**: The system didn't properly handle cases where commodities already existed

## Solution Implemented

### 1. Database-Level Protection
- **Added unique constraint** on `Commodity` model for `(site, name)` combination
- **Created migration** `0012_add_commodity_unique_constraint.py` to apply the constraint

### 2. Application-Level Improvements
- **Enhanced `add_def_to_project` function** to use `get_or_create()` instead of manual checking
- **Improved race condition handling** in process and storage APIs
- **Added better error handling** for missing default commodities

### 3. Cleanup Functionality
- **Created cleanup endpoint** `/api/project/{project_name}/cleanup-duplicates/` to remove existing duplicates
- **Added URL route** for the cleanup function

## Files Modified

1. `backend/projects/models.py` - Added unique constraint to Commodity model
2. `backend/projects/api/commodity.py` - Improved commodity creation logic and added cleanup function
3. `backend/projects/api/process.py` - Enhanced error handling for commodity creation
4. `backend/projects/api/storage.py` - Enhanced error handling for commodity creation
5. `backend/projects/urls.py` - Added cleanup endpoint URL
6. `backend/projects/migrations/0012_add_commodity_unique_constraint.py` - Database migration

## How to Use the Cleanup Function

If you have existing duplicate commodities in your project, you can clean them up by making a POST request to:

```
POST /api/project/{project_name}/cleanup-duplicates/
```

This will:
- Find all commodities with duplicate names in the same site
- Keep the first occurrence (by ID)
- Remove all other duplicates
- Return a summary of how many duplicates were removed

## Testing

The fix has been tested and verified to:
- ✅ Prevent new duplicate commodities from being created
- ✅ Handle race conditions properly
- ✅ Maintain database integrity with unique constraints
- ✅ Provide cleanup functionality for existing duplicates

## Prevention

With these changes, the system now:
- **Prevents duplicates at the database level** with unique constraints
- **Handles race conditions** using atomic `get_or_create()` operations
- **Provides better error messages** when operations fail
- **Offers cleanup tools** for existing problematic data

The duplicate commodity bug should no longer occur in the application.

## Additional Fix: Transmission Loading Issue

### Problem Description
When loading projects from saved `.urbs` config files, the system was creating hundreds of empty transmissions automatically. This happened because the config loading logic was incorrectly creating transmissions for ALL sites that had commodities with transmission data, rather than only creating the specific transmission relationships defined in the config.

**The Root Cause**: The issue was in the config export/import process:
1. **Export**: When downloading a `.urbs` config file, the system was including a `"transmission": None` field for every commodity that didn't have transmissions
2. **Import**: When loading the config file, the system was checking `if "transmission" in dataCommodity:` which was true even for `None` values
3. **Result**: This caused the system to create transmission objects for every possible site-commodity combination, resulting in hundreds of empty transmissions

### Root Cause
The issue was in `backend/projects/api/configupload.py`:
- The transmission loading loop was iterating through all sites and commodities
- It was checking `if "transmission" in dataCommodity:` which was true even for `None` values
- This caused the system to create transmission objects for every possible site-commodity combination
- The result was hundreds of unnecessary, empty transmission objects

### Solution Implemented
1. **Fixed transmission loading logic** to only create transmissions for specific relationships defined in the config
2. **Added proper validation** to check `if "transmission" in dataCommodity and dataCommodity["transmission"] is not None:`
3. **Added duplicate prevention** by checking if transmissions already exist before creating new ones
4. **Added error handling** to skip transmissions when source sites or commodities don't exist
5. **Created cleanup function** to remove existing invalid transmissions

### New Cleanup Endpoints
- `/api/project/{project_name}/cleanup-duplicates/` - Removes duplicate commodities
- `/api/project/{project_name}/cleanup-transmissions/` - Removes invalid/empty transmissions

### Files Modified for Transmission Fix
- `backend/projects/api/configupload.py` - Fixed transmission loading logic
- `backend/projects/api/commodity.py` - Added transmission cleanup function
- `backend/projects/urls.py` - Added transmission cleanup endpoint

## Summary of All Fixes

The application now handles both:
1. **Duplicate commodity creation** (prevented at database and application level)
2. **Excessive transmission creation** (fixed in config loading logic)
3. **Data cleanup** (tools to remove existing problematic data)

Both issues should now be resolved, allowing you to:
- Save and load projects without creating duplicate commodities
- Load projects without creating excessive empty transmissions
- Clean up any existing problematic data using the cleanup endpoints

## Additional Fix: Transmission Validation

### Problem Description
Even after fixing the config loading logic, empty transmissions were still being created. This was happening because the transmission update/create API was accepting empty or invalid data and creating transmissions with zero values.

### Root Cause
The `update_transmission` function in `backend/projects/api/transmission.py` was not validating the input data, allowing empty transmissions to be created when:
- Required fields were missing or null
- Critical values like efficiency, costs, and capacity were zero
- The frontend was calling the API with incomplete data

### Solution Implemented
1. **Added validation** to the `update_transmission` function to check for required fields
2. **Prevented zero values** for critical transmission parameters (efficiency, costs, capacity)
3. **Disabled problematic duplication** function that had flawed logic causing infinite loops
4. **Enhanced error messages** to help identify validation issues

### Files Modified for Transmission Validation
- `backend/projects/api/transmission.py` - Added validation and disabled problematic duplication

## Summary of All Fixes

The application now handles:
1. **Duplicate commodity creation** (prevented at database and application level)
2. **Excessive transmission creation** (fixed in config loading logic)
3. **Empty transmission creation** (prevented with validation)
4. **Data cleanup** (tools to remove existing problematic data)

All issues should now be resolved, allowing you to:
- Save and load projects without creating duplicate commodities
- Load projects without creating excessive empty transmissions
- Prevent empty transmissions from being created through the API
- Clean up any existing problematic data using the cleanup endpoints





