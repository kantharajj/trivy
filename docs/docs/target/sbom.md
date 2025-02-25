# SBOM scanning
Trivy can take the following SBOM formats as an input and scan for vulnerabilities.

- CycloneDX
- SPDX
- SPDX JSON
- CycloneDX-type attestation

To scan SBOM, you can use the `sbom` subcommand and pass the path to the SBOM.
The input format is automatically detected.

```bash
$ trivy sbom /path/to/sbom_file
```

!!! note
    Passing SBOMs generated by tool other than Trivy may result in inaccurate detection 
    because Trivy relies on custom properties in SBOM for accurate scanning.

## CycloneDX
Trivy supports CycloneDX as an input.

!!! note
    CycloneDX XML is not supported at the moment.


```bash
$ trivy sbom /path/to/cyclonedx.json
```

!!! note
    If you want to generate a CycloneDX report from a CycloneDX input, please be aware that the output stores references to your original CycloneDX report and contains only detected vulnerabilities, not components.
    The report is called [BOV](https://cyclonedx.org/capabilities/sbom/).

## SPDX
Trivy supports the SPDX SBOM as an input.

The following SPDX formats are supported:

- Tag-value (`--format spdx`)
- JSON (`--format spdx-json`)

```bash
$ trivy image --format spdx-json --output spdx.json alpine:3.16.0
$ trivy sbom spdx.json
```

<details>
<summary>Result</summary>

```
2022-09-15T21:32:27.168+0300    INFO    Vulnerability scanning is enabled
2022-09-15T21:32:27.169+0300    INFO    Detected SBOM format: spdx-json
2022-09-15T21:32:27.210+0300    INFO    Detected OS: alpine
2022-09-15T21:32:27.210+0300    INFO    Detecting Alpine vulnerabilities...
2022-09-15T21:32:27.211+0300    INFO    Number of language-specific files: 0

spdx.json (alpine 3.16.0)
=========================
Total: 5 (UNKNOWN: 0, LOW: 0, MEDIUM: 2, HIGH: 2, CRITICAL: 1)

┌──────────────┬────────────────┬──────────┬───────────────────┬───────────────┬────────────────────────────────────────────────────────────┐
│   Library    │ Vulnerability  │ Severity │ Installed Version │ Fixed Version │                           Title                            │
├──────────────┼────────────────┼──────────┼───────────────────┼───────────────┼────────────────────────────────────────────────────────────┤
│ busybox      │ CVE-2022-30065 │ HIGH     │ 1.35.0-r13        │ 1.35.0-r15    │ busybox: A use-after-free in Busybox's awk applet leads to │
│              │                │          │                   │               │ denial of service...                                       │
│              │                │          │                   │               │ https://avd.aquasec.com/nvd/cve-2022-30065                 │
├──────────────┼────────────────┼──────────┼───────────────────┼───────────────┼────────────────────────────────────────────────────────────┤
│ libcrypto1.1 │ CVE-2022-2097  │ MEDIUM   │ 1.1.1o-r0         │ 1.1.1q-r0     │ openssl: AES OCB fails to encrypt some bytes               │
│              │                │          │                   │               │ https://avd.aquasec.com/nvd/cve-2022-2097                  │
├──────────────┤                │          │                   │               │                                                            │
│ libssl1.1    │                │          │                   │               │                                                            │
│              │                │          │                   │               │                                                            │
├──────────────┼────────────────┼──────────┼───────────────────┼───────────────┼────────────────────────────────────────────────────────────┤
│ ssl_client   │ CVE-2022-30065 │ HIGH     │ 1.35.0-r13        │ 1.35.0-r15    │ busybox: A use-after-free in Busybox's awk applet leads to │
│              │                │          │                   │               │ denial of service...                                       │
│              │                │          │                   │               │ https://avd.aquasec.com/nvd/cve-2022-30065                 │
├──────────────┼────────────────┼──────────┼───────────────────┼───────────────┼────────────────────────────────────────────────────────────┤
│ zlib         │ CVE-2022-37434 │ CRITICAL │ 1.2.12-r1         │ 1.2.12-r2     │ zlib: a heap-based buffer over-read or buffer overflow in  │
│              │                │          │                   │               │ inflate in inflate.c...                                    │
│              │                │          │                   │               │ https://avd.aquasec.com/nvd/cve-2022-37434                 │
└──────────────┴────────────────┴──────────┴───────────────────┴───────────────┴────────────────────────────────────────────────────────────┘
```

</details>

## SBOM attestation

You can also scan an SBOM attestation.
In the following example, [Cosign](https://github.com/sigstore/cosign) gets an attestation and Trivy scans it.
You must create CycloneDX-type attestation before trying the example.
To learn more about how to create an CycloneDX-Type attestation and attach it to an image, see the [SBOM attestation page](../supply-chain/attestation/sbom.md#sign-with-a-local-key-pair).

```bash
$ cosign verify-attestation --key /path/to/cosign.pub --type cyclonedx <IMAGE> > sbom.cdx.intoto.jsonl
$ trivy sbom ./sbom.cdx.intoto.jsonl

sbom.cdx.intoto.jsonl (alpine 3.7.3)
=========================
Total: 2 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 2)

┌────────────┬────────────────┬──────────┬───────────────────┬───────────────┬──────────────────────────────────────────────────────────┐
│  Library   │ Vulnerability  │ Severity │ Installed Version │ Fixed Version │                          Title                           │
├────────────┼────────────────┼──────────┼───────────────────┼───────────────┼──────────────────────────────────────────────────────────┤
│ musl       │ CVE-2019-14697 │ CRITICAL │ 1.1.18-r3         │ 1.1.18-r4     │ musl libc through 1.1.23 has an x87 floating-point stack │
│            │                │          │                   │               │ adjustment im ......                                     │
│            │                │          │                   │               │ https://avd.aquasec.com/nvd/cve-2019-14697               │
├────────────┤                │          │                   │               │                                                          │
│ musl-utils │                │          │                   │               │                                                          │
│            │                │          │                   │               │                                                          │
│            │                │          │                   │               │                                                          │
└────────────┴────────────────┴──────────┴───────────────────┴───────────────┴──────────────────────────────────────────────────────────┘
```
