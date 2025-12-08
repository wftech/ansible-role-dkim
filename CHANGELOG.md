# DKIM Role Changelog

## Updates for DKIM Optimization Feature

### Changes Made

This role has been updated to support the DKIM optimization feature with the following enhancements:

#### 1. Custom Selector Support
- **Previous behavior**: Only supported the "default" selector
- **New behavior**: Supports custom selectors per domain
- **Data format change**: `dkim_domains` now expects a list of dictionaries with `domain` and `selector` keys

Example:
```yaml
dkim_domains:
  - domain: example.com
    selector: default
  - domain: another.com
    selector: mail
```

#### 2. Key File Validation (Requirement 8.2)
- Added validation to check if existing DKIM key files are readable
- Automatically fixes file permissions if they are incorrect
- Uses `failed_when: false` to prevent failures on permission fixes

#### 3. Missing Key Recovery (Requirement 8.3)
- Added recovery logic to generate missing keys for domains with DKIM enabled
- Uses the `creates` parameter to ensure idempotency
- Will regenerate keys if they are accidentally deleted

#### 4. Idempotency (Requirement 8.1, 1.2)
- Role now properly validates existing keys without regenerating them
- Uses `creates` parameter in key generation to skip if keys already exist
- File modification timestamps remain unchanged on subsequent runs

#### 5. OpenDKIM Configuration Updates (Requirement 8.4)
- Updated SigningTable template to use custom selectors
- Updated KeyTable template to use custom selectors
- Configuration files are regenerated when `dkim_domains` changes

### Template Changes

#### SigningTable.j2
- Now uses `{{ item.selector }}` instead of hardcoded "default"
- Format: `domain selector._domainkey.domain`

#### KeyTable.j2
- Now uses `{{ item.selector }}` instead of hardcoded "default"
- Format: `selector._domainkey.domain domain:selector:/path/to/key`

### Backward Compatibility

**Breaking Change**: The `dkim_domains` variable format has changed from a simple list to a list of dictionaries.

**Migration**: If you were using:
```yaml
dkim_domains:
  - example.com
  - another.com
```

You must now use:
```yaml
dkim_domains:
  - domain: example.com
    selector: default
  - domain: another.com
    selector: default
```

### Testing

To test the role:
1. Ensure the inventory script provides `dkim_domains` in the correct format
2. Run the playbook with the DKIM tag: `ansible-playbook site.yml --tags dkim`
3. Verify that existing keys are not regenerated (check file timestamps)
4. Delete a key file and re-run to verify recovery logic works
5. Check that OpenDKIM configuration files are updated correctly

### Requirements Addressed

- **Requirement 1.2**: DKIM Role validates existing keys without regenerating them
- **Requirement 8.1**: Role remains idempotent when re-run
- **Requirement 8.2**: Validates key files are readable
- **Requirement 8.3**: Generates missing keys for active DKIM domains
- **Requirement 8.4**: Updates OpenDKIM signing table and key table configuration
