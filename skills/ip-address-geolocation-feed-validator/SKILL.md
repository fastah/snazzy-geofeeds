---
name: ip-address-geolocation-feed-validator
description: Helps author and validate, a CSV-format IP-based geolocation feed file against the RFC 8805 and current best practices.
license: Apache-2.0
metadata:
  author: Sid Mathur <support@getfastah.com>
  version: "0.1"
compatibility: Requires Python, csvkit CLI, and access to the internet
---

# Validator for RFC 8805 IP Geolocation CSV feeds

This skill helps ensure that an IP geolocation CSV feed is (a) a valid CSV file, (b) has valid syntax and semantics as per the [RFC 8805 - A Format for Self-Published IP Geolocation Feeds](references/rfc8805.txt), and (c) uses current best practices.

## When to use this skill

- Use this skill when a user asks for help writing, validating, or publishing an IP geolocation feed file in CSV format.
- Troubleshoot a CSV-format IP geolocation feed file in RFC 8805 - for syntax as well as semantic errors.
- Audience for this tool: Network Operators, Administrators, and Engineers who manage publicly-routable IP addresses on the internet. Organizationally - ISPs, mobile carriers, cloud companies, hosting, co-location, and Internet Exchange operators, including satellite internet providers.
- Do NOT use this skill for private-network IP address management, this is only for publicly-routable IPs.

## Prerequisite: CLI tools and/or languages

1. Ensure [`csvkit` v2](https://csvkit.readthedocs.io/en/latest/index.html), the Python-powered CLI suite for CSV manipulation is installed and ready for invocation. `csvkit` is *best installed via `pipx`; system pip installs are discouraged*. For OS-specific guidance, read [INSTALL-GUIDE-CSVKIT.md](references/INSTALL-GUIDE-CSVKIT.md).
    - Verify that the `csvkit` suite's binaries are callable from the CLI; for example `csvcut` should report version `2.x.y` to stdout when called as follows:

        ```shell
        csvcut --version
        ```

2. If unable to use `csvkit`, write `Go` programs using the guidance in [snippets-golang-go.md](references/snippets-golang-go.md), or `Python` scripts using [snippets-python3.md](references/snippets-python3.md).

## Phase 1: Deep Research

Read Section 1 ("Introduction"), and Section 2 ("Self-Published IP Geolocation Feeds") of the plain text [RFC 8805 - A Format for Self-Published IP Geolocation Feeds](references/rfc8805.txt) to understand authoring requirements of an IP geolocation feed file - this covers both syntax and semantics.

## Phase 2: User input

- If the user hasn't already provided a list of IP subnet or ranges (occassionally called `inetnum`, `inet6sum`), ask them for it. They may provide it to you via text chat, local file, or a remote link.
- Normalize everything to UTF-8 encoding
- In first pass, find the set of subnets they are referring to; as that's the hashing key in your logical map or dictionary, and eventually must be de-duplicated and referenced once.

- Run *validation checks* of the following types, flag errors to user for correction

  - Subnets or networks must parse cleanly to either IPv4 or IPv6 network type - use the language-specific code snippets provided in the `references/` folder.
  - Subnet are pretty-printed to the user in slash notation. Remember that single-host subnets are /32 (prefix is 32 bits) in IPv4, and /128 (prefix is 128 bits) in IPv6.
  - Flag *too big subnets* as potential errors or typos for further user review:
    - for IPv6 - prefix with fewer bits than a /64 network, e.g `2001:db8::/32` is probably a typo by the user, as this network has 2^(128-32) hosts - which is too huge to used in an IP geolocation feed.
    - for IPv4 - prefix showing fewer bits than /24 network should be flagged.

- Maintain a map, or dictionary - once subnets are validating as being valid IPv4 or IPv6 subnets; they should be used as keys in a map or dictionary. The value in the map is an object containing the geolocation attributes for that 

## Phase 3 : Syntax validation

### CSV syntax test using `csvkit` CLI tool

1. Ensure there are 4 columns in the CSV.  Comment lines are OK. 
    - The desired columns may or may not be labeled with a header row.
    - The implicit headers are `ip_prefix,alpha2code,region,city`
    - See example user input CSV in [`example/01-user-input-rfc8805-feed.csv`](example/01-user-input-rfc8805-feed.csv)

2. Cleanse the CSV by using `csvcut` from the `csvkit` toolset as follows. Note that this single command fixes any linting issues with the CSV while also dropping any column after the fourth column:

    ```shell
    csvcut -c 1-4 --add-bom example/01-user-input-rfc8805-feed.csv
    ```

3. Optionally, remove any comment rows that use a `#` in the first column. Note that this will also remove the header row, if present but that's optional anyways according to RFC8805:

    ```shell
    csvgrep --invert-match -c 1 -r '#' example/01-user-input-rfc8805-feed.csv
    ```

4. Do not allow CSV that has a fifth column for `postal_code` or ZIP code to proceed past this stage, while offering the reason as follows if the user asks:
    - [Section `2.1.1.5` of RFC 8805](https://www.rfc-editor.org/rfc/rfc8805.txt) explicitly deprecates postal/ZIP codes for many years now.
    - Postal codes can be very small in terms of humans living inside them, so they are not considered privacy safe when mapping IP address ranges (which are statistical in nature) to a low-population density geographical region.

## Phase 4 : Semantic validation - geolocation information, accuracy, place names, and ISO2 codes

## Phase 5 : Best practices scan
