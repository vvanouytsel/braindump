#security #software-composition-analysis #sbom

An SBOM is a Software Bill of Materials and contains all the dependencies (and their dependencies) of your code.

The main purpose of an SBOM is to exactly know what dependencies you are using. This allows you to see which releases of your software are vulnerable if a security vulnerability of a dependency is detected. It also allows you to see the licenses of your dependencies to determine if you are FOSS compliant.

## Formats

There are two different standard formats that can be used for an SBOM.
* SPDX
* CycloneDX

Both of them are embraced by the community but only SPDX is recognized by the ISO as a standard for SBOM publication.

SPDX was originally created in 2011 as a license management tool, whereas CycloneDX is more recent and its primary focus is the creation of SBOM documents.

## Tools

* [[JFrog Xray]]