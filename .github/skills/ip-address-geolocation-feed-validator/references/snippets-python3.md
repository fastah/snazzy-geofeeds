# Code examples for Python 3

- Use Python 3's built in [`ipaddress` pacakge](https://docs.python.org/3/library/ipaddress.html), with `strict=True` passed to constructors where available.
- Be intentional about IPv4 or IPv6 address parsing, they are not the same for professional network engineers! Use the strongest type/class available.
- Remember that it's OK for a subnet to have a single host too, just use mask `/128` for IPv6, and `/32` for IPv4.

## IP address and Subnet parsing

- Use the [convenience factory functions in `ipaddress`](https://docs.python.org/3/library/ipaddress.html#convenience-factory-functions).

    The following `ipaddress.ip_address(textAddress)` examples parse the text into an `IPv4Address` and `IPv6Address` object respectively.

    ```python
    ipaddress.ip_address('192.168.0.1')
    ipaddress.ip_address('2001:db8::')
    ```

    The following `ipaddress.ip_network(address, strict=True)` examples parse a subnet string input and return an `IPv4Network` or `IPv6Network` object, with hard fail on invalid input.

    ```python
    ipaddress.ip_network('192.168.0.0/28', strict=True)
    ```

    The following strict mode subnet parsing call fails (correctly), because `"ValueError: 192.168.0.1/30 has host bits set"`. Ask the user to fix these errors; don't imagine answers. 

    ```python
     ipaddress.ip_network('192.168.0.1/30', strict=True);
     ```

- Use the `strict` form parser [`ipaddress.ip_network(address, strict=True)`](https://docs.python.org/3/library/ipaddress.html#ipaddress.ip_network)

## Dictionary of IP subnets

Use a Python dictionary to keep track of subnets and their associated geolocation properties and/or user instructions ; all of `IPv4Network`, `IPv6Network`, `IPv4Address` and `IPv6Address` are hashable, so they can be used as keys in dictionaries.
