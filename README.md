# CWS

**CSS Web Signatures — "digital signatures" implemented in Pure CSS.**

No JavaScript. No build step. Just CSS doing things CSS was never meant to do.

See [`rfc-cs24.txt`](rfc-cs24.txt) for the full specification.

## Algorithm: CS24

**C**ascading **S**tylesheet **24** — a parody of HS256 that computes
signatures using nothing but CSS selectors, counters, and counter styles.

The CS24 signature is `header_constant ⊕ payload ⊕ key`, where the header constant (`0x5C2834`) is derived by XOR-folding the UTF-8 bytes of `{"alg":"CS24"}` into 3 octets.

```
eyJhbGciOiJDUzI0In0.XXXX.YYYY
\_________________/  \__/  \__/
        |              |     |
   Fixed header     4 chars  4 chars         Total: 29 characters
   (19 chars)       payload  signature       (CWS Compact Serialization)
```

### How it works

| Technique | Purpose |
|---|---|
| `@counter-style base64url` | Maps 6-bit values (0–63) to the Base64URL alphabet |
| `@counter-style ascii` | Maps byte values (32–126) to printable ASCII |
| `counter-increment` on relay `<span>`s | Accumulates bit weights into counters |
| `:has()` selectors on `#cws` | Reads checkbox state to conditionally increment counters |
| XOR via `:has()` pairs | `h⊕p⊕k`: header constant baked into selectors, XOR for h=0 bits, XNOR for h=1 bits |
| 96 mismatch selectors | Any bit mismatch between computed and entered signature → INVALID |
| `content: counter(…, base64url)` | Renders live Base64URL output from accumulated counters |

### Stats

- **72** interactive checkboxes (24 payload + 24 key + 24 verification)
- **48** hidden relay `<span>` elements for counter accumulation
- **24** payload encoding rules
- **24** XOR signature rules (48 selectors)
- **96** verification mismatch selectors
- **0** lines of JavaScript
- **0** bits of security

### Test vector

From [Section 11](rfc-cs24.txt) of the RFC:

| | ASCII | Hex | Binary |
|---|---|---|---|
| Payload | `Hi!` | `48 69 21` | `01001000 01101001 00100001` |
| Key | | `AD E6 54` | `10101101 11100110 01010100` |
| Header constant | | `5C 28 34` | `01011100 00101000 00110100` |
| **Signature** | | `B9 87 41` | `10111001 10000111 01000001` |

Token: `eyJhbGciOiJDUzI0In0.SGkh.uYdB`

## Usage

Open `index.html` in a modern browser (Chrome 105+, Firefox 121+, Safari 15.4+).

1. Toggle **payload bits** to encode data as Base64URL
2. Toggle **key bits** to set your "secret" key
3. The **computed signature** updates live (header ⊕ payload ⊕ key)
4. Enter signature bits in the **verify** section
5. If all 24 bits match the computed signature → ✓ VALID
6. If any single bit differs → ✗ INVALID

## Browser requirements

Requires `:has()` selector support:
- Chrome/Edge 105+
- Firefox 121+
- Safari 15.4+

## Security advisory

CS24 provides exactly **0 bits of security**. The signature algorithm is XOR.
The key is visible on screen. The entire algorithm is in a stylesheet.
Do not use this to protect anything, ever.

## License

MIT — use responsibly (or irresponsibly, given the context).
