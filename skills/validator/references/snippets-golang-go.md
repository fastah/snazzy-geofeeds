# Code examples for Go

- Use Go's standard library [`net/netip`](https://pkg.go.dev/net/netip) package for IP address and prefix parsing. It is more efficient and type-safe than the older `net` package.
- Be intentional about IPv4 vs IPv6 address parsing—they are not the same for professional network engineers.
- Remember that a subnet can contain a single host: use `/128` for IPv6 and `/32` for IPv4.

## IP address and subnet parsing

Use [`netip.ParseAddr`](https://pkg.go.dev/net/netip#ParseAddr) for single addresses and [`netip.ParsePrefix`](https://pkg.go.dev/net/netip#ParsePrefix) for subnets:

```go
package main

import (
	"fmt"
	"net/netip"
)

func main() {
	// Parse a single IPv4 address
	addr4, err := netip.ParseAddr("192.168.0.1")
	if err != nil {
		fmt.Println("Invalid IPv4 address:", err)
	}
	fmt.Println("IPv4:", addr4, "Is4:", addr4.Is4())

	// Parse a single IPv6 address
	addr6, err := netip.ParseAddr("2001:db8::1")
	if err != nil {
		fmt.Println("Invalid IPv6 address:", err)
	}
	fmt.Println("IPv6:", addr6, "Is6:", addr6.Is6())

	// Parse an IPv4 prefix (subnet)
	prefix4, err := netip.ParsePrefix("192.168.0.0/28")
	if err != nil {
		fmt.Println("Invalid IPv4 prefix:", err)
	}
	fmt.Println("Prefix:", prefix4, "Bits:", prefix4.Bits())

	// Parse an IPv6 prefix (subnet)
	prefix6, err := netip.ParsePrefix("2001:db8::/32")
	if err != nil {
		fmt.Println("Invalid IPv6 prefix:", err)
	}
	fmt.Println("Prefix:", prefix6, "Bits:", prefix6.Bits())
}
```

### Strict validation: detect host bits set

Unlike Python's `strict=True`, Go's `netip.ParsePrefix` does not reject prefixes with host bits set. You must check manually:

```go
func parseStrictPrefix(s string) (netip.Prefix, error) {
	prefix, err := netip.ParsePrefix(s)
	if err != nil {
		return netip.Prefix{}, err
	}
	// Check if the address equals the masked (network) address
	if prefix.Addr() != prefix.Masked().Addr() {
		return netip.Prefix{}, fmt.Errorf("%s has host bits set", s)
	}
	return prefix, nil
}
```

Example usage:

```go
// This will fail: "192.168.0.1/30 has host bits set"
prefix, err := parseStrictPrefix("192.168.0.1/30")
if err != nil {
	fmt.Println("Error:", err)
}
```

## Map of IP prefixes to geolocation data

Use `netip.Prefix` as a map key since it is comparable:

```go
type GeoLocation struct {
	CountryCode string // ISO 3166-1 alpha-2 (e.g., "US")
	Region      string // ISO 3166-2 (e.g., "US-CA")
	City        string
}

// Map from prefix to geolocation
geoData := make(map[netip.Prefix]GeoLocation)

prefix, _ := netip.ParsePrefix("192.168.0.0/24")
geoData[prefix] = GeoLocation{
	CountryCode: "US",
	Region:      "US-CA",
	City:        "Los Angeles",
}
```

## ISO 3166-1 country code validation

Read valid ISO 2-letter country codes from [assets/iso3166-1.json](../assets/iso3166-1.json):

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type Country struct {
	Alpha2 string `json:"alpha_2"`
	Alpha3 string `json:"alpha_3"`
	Name   string `json:"name"`
	Flag   string `json:"flag"`
}

type ISO31661Data struct {
	Countries []Country `json:"3166-1"`
}

func loadValidCountryCodes(path string) (map[string]bool, error) {
	data, err := os.ReadFile(path)
	if err != nil {
		return nil, err
	}

	var iso ISO31661Data
	if err := json.Unmarshal(data, &iso); err != nil {
		return nil, err
	}

	codes := make(map[string]bool, len(iso.Countries))
	for _, c := range iso.Countries {
		codes[c.Alpha2] = true
	}
	return codes, nil
}

func main() {
	validCodes, err := loadValidCountryCodes("assets/iso3166-1.json")
	if err != nil {
		fmt.Println("Failed to load country codes:", err)
		return
	}

	// Validate a country code
	testCode := "US"
	if validCodes[testCode] {
		fmt.Println(testCode, "is valid")
	} else {
		fmt.Println(testCode, "is NOT valid")
	}
}
```

## ISO 3166-2 region code validation

Read valid region codes from [assets/iso3166-2.json](../assets/iso3166-2.json):

```go
type Subdivision struct {
	Code   string `json:"code"`
	Name   string `json:"name"`
	Type   string `json:"type"`
	Parent string `json:"parent,omitempty"`
}

type ISO31662Data struct {
	Subdivisions []Subdivision `json:"3166-2"`
}

func loadValidRegionCodes(path string) (map[string]bool, error) {
	data, err := os.ReadFile(path)
	if err != nil {
		return nil, err
	}

	var iso ISO31662Data
	if err := json.Unmarshal(data, &iso); err != nil {
		return nil, err
	}

	codes := make(map[string]bool, len(iso.Subdivisions))
	for _, s := range iso.Subdivisions {
		codes[s.Code] = true
	}
	return codes, nil
}
```
