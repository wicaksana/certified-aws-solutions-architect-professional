# ECR

## Scan images for vulnerabilities
1. Basic scanning
  * Using common CVE database to scan for OS vulnerabilities
  * Options:
      - scan on push
      - manual scanning
2. Enhanced scanning
  * integrates with Amazon Inspector
  * Automated, continuous scanning of your repos
  * The container images are scanned for both operating systems and programming language package vulnerabilities
  * As new vulnerabilities appear, the scan results are updated and Amazon Inspector emits an event to EventBridge to notify you
  * Options:
      - scan on push
      - continuous scanning
