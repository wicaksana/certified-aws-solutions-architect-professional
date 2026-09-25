# Multi-Factor Authentication (MFA)

- **Factor**: different piece of evidence which proves the identity
- Factors:
    - **Knowledge**: something we as users know: username, password
    - **Possession**: something we as users have: bank card, MFA device/app
    - **Inherent**: something we are, example: fingerprint, face, voice, iris
    - **Location**: a location (physical) or which network we are connected to (corporate wifi)
- More factors means more security, harder to bypass by an intruder
- Three primary mechanisms to enforce MFA:
    * AWS IAM Identity Center (recommended)
    * IAM policy-based enforcement (for local IAM users) --> `"aws:MultiFactorAuthPresent": "false"`
    * SCPs via AWS Organizations -- deploy an SCP to the organization root or specific OUs --> `"aws:MultiFactorAuthPresent": "false"`
