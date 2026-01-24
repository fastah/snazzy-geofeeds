---
name: csv-rfc8805-validation
description: Validates CSV-format IP geofeed files against RFC 8805 syntax standards. Use when processing or debugging IP geolocation feeds to ensure valid CSV structure and RFC 8805 compliance.
license: Apache-2.0
metadata:
  author: Sid Mathur <support@getfastah.com>
  version: "0.1"
---

# RFC 8805 IP Geofeed CSV Syntax Validation

This skill helps ensure that an IP geolocation CSV feed is (a) a valid CSV file, and (b) formatted according to the minimum requirements of the [RFC 8805 internet standard - A Format for Self-Published IP Geolocation Feeds](https://www.rfc-editor.org/rfc/rfc8805.txt).

## When to use this skill

- As a first-pass syntax validation step while accepting/processing a CSV-formatted IP geolocation file or columnar text.
- Debug problems with its RFC 8805 syntax, but not its semantics.

## Running syntax tests

### CSV syntax test using `csvkit` CLI tool

1. Prerequisite: Ensure [`csvkit`](https://csvkit.readthedocs.io/en/latest/index.html), the Python3 powered CLI is installed and ready for invocation. `csvkit` is *best installed via `pipx`; system pip installs are discouraged*.

    - Install and verify csvkit v2.x.y as a CLI tool on Linux/macOS/Windows using the safest modern method.

    - Requirements: Python ≥ 3.10, avoid system Python modifications, prefer pipx, include PATH fixes, and provide OS-specific commands.

        | Platform               | Best default            | Acceptable alternative | Discouraged        |
        |------------------------|-------------------------|------------------------|--------------------|
        | Linux (Ubuntu/Debian)  | `pipx install csvkit`   | Python virtualenv      | `sudo pip install` |
        | Linux (RHEL/CentOS)    | `pipx install csvkit`   | Python virtualenv      | system `pip`       |
        | macOS                  | `pipx install csvkit`   | `brew install csvkit`  | system `pip`       |
        | Windows                | `pipx install csvkit`   | Python virtualenv      | global `pip`       |

    - End by running a version check as below, and explain how to debug if it fails or does not report a version `2.x.y`:

        ```shell
        csvcut --version
        ```

2. Ensure there are 4 columns in the CSV.  Comment lines are OK. 
    - The desired columns may or may not be labeled with a header row.
    - The implicit headers are `ip_prefix,alpha2code,region,city`
    - See example user input CSV in [`example/01-user-input-rfc8805-feed.csv`](example/01-user-input-rfc8805-feed.csv)

3. Cleanse the CSV by using `csvcut` from the `csvkit` toolset as follows. Note that this single command fixes any linting issues with the CSV while also dropping any column after the fourth column:

    ```shell
    csvcut -c 1-4 --add-bom example/01-user-input-rfc8805-feed.csv
    ```

4. Optionally, remove any comment rows that use a `#` in the first column. Note that this will also remove the header row, if present but that's optional anyways according to RFC8805:

    ```shell
    csvgrep --invert-match -c 1 -r '#' example/01-user-input-rfc8805-feed.csv
    ```

5. Do not allow CSV that has a fifth column for `postal_code` or ZIP code to proceed past this stage, while offering the reason as follows if the user asks:
    - [Section `2.1.1.5` of RFC 8805](https://www.rfc-editor.org/rfc/rfc8805.txt) explicitly deprecates postal/ZIP codes for many years now.
    - Postal codes can be very small in terms of humans living inside them, so they are not considered privacy safe when mapping IP address ranges (which are statistical in nature) to a low-population density geographical region.
