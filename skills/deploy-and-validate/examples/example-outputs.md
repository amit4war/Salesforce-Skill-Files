# Example Outputs

## Example Deployment Summary

- Target org: `uat-org`
- Scope: `manifest/package.xml`
- Risk: High
- Validation: Passed (`RunLocalTests`)
- Deployment: Passed (job ID captured)
- Post-deploy checks: smoke tests passed
- Rollback: disable impacted feature toggle + revert commit + redeploy previous package
