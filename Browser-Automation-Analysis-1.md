# Browser Automation Analysis: Protocol & Request-Level Signals

This document outlines several HTTP level signals that can be used to detect automated traffic. These signals are based on browser consistency validation, protocol anomalies, and header structure analysis. Modern bot detection systems rarely rely on a single signal. Instead, they combine many weak signals to identify automated behavior.

### Client Hint Consistency

Modern browsers send User-Agent Client Hints such as:

sec-ch-ua  
sec-ch-ua-mobile  
sec-ch-ua-platform  

These headers are typically sent together as a cluster. Automation frameworks frequently include only one or two of these headers, creating inconsistencies that can be used as detection signals. Real browsers normally send these headers together because they are generated automatically by the browser networking stack.

### Casing Signals

Similar to client hints, browsers all use specific casing. For instance, both Firefox and Chrome send the User-Agent header as so:

User-Agent

Commonly, automation tools may send headers with different casing.

e.g:

USER-AGENT

Although HTTP header names are technically case-insensitive, browsers tend to use consistent casing patterns. Deviations from these patterns can be used as weak detection signals.

### Accept Language Consistency

The accept-language header follows the same syntax almost every time.

e.g:

en-GB,en-US;q=0.9,en;q=0.8

Automation frameworks sometimes omit this header entirely or generate unrealistic language lists whether that's malformed language codes, excessive language entries, or uncommon language combinations. These inconsistencies can contribute to risk scoring signals in automation detection systems.

### User Agent Mismatch

In Chromium based browsers, the sec-ch-ua header is used to identify the browsers version. Through my research, I've encountered numerous cases where automation frameworks will generate inconsistencies between the User-Agent header, and the Sec-Ch-Ua header. 

e.g:

Sec-Ch-Ua: "Not:A-Brand";v="99", "Google Chrome";v="145", "Chromium";v="145"

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36

Real browsers generate these values internally, so they remain consistent. Any mismatches between these headers can be used as a strong automation signal.

### Content-Length Mismatch
The content-length header is the size of the request body in bytes. Browsers automatically calculate this header when sending request. Through my research, I've encountered numerous automation frameworks that will either miscalculate the content-length of a request, send two content-length headers, or they will use a static content-length header. A miscalculated Content-Length can be used as a strong automation signal.

POST https://test.com/api/endpoint

Content-Length: 38

Request Body:
awidjawd%2ecom

Actual length: 14

### Encoding and Path Normalization

Attackers frequently manipulate URL paths using encoding techniques in order to bypass input filters or Web Application Firewall (WAF) rules.

Common examples include percent-encoded characters such as:
```
/  -> %2F  
\  -> %5C  
.  -> %2E  
```
However, encoding is not limited to structural characters. In some cases attackers may encode normal characters as well:
```
o -> %6F
```
While this does not change the path structure, it can expose inconsistencies in how different layers normalize requests. For example, a filter may compare the raw encoded string while the backend application decodes it before processing. This can lead to situations where encoded characters bypass simple pattern matching rules.

Attackers often combine encoding with other techniques such as:

- path traversal sequences (`../`)
- double encoding (`%252F`, `%252E`)
- mixed case paths
- additional path separators
- Overlong UTF-8 Encodings

These techniques attempt to exploit differences in how requests are normalized across different components of the system.

### TLS Fingerprint Consistency
Transport Layer Security (TLS) is a cryptographic protocol that ensures an encrypted link between a web server and a user's browser. When a browser establishes a TLS connection, it sends a ClientHello containing:
```
- Cipher Suites
- TLS Extensions
- Extension Ordering
- ALPN Values
- Supported Groups
```
The exact combination of these fields creates what is known as a TLS Fingerprint. Common fingerprinting methods include things like:
```
- JA3
- JA4
```
Each browser version tends to produce a consistent TLS handshake fingerprint. For example:
```
Chrome v133
TLS Fingerprint -> Chrome 133 Pattern
```
Automation frameworks often generate mismatches because they:
```
- Use different TLS libraries
- Use HTTP clients instead of real browsers
- Modify headers but not TLS behaviour
```
An example of an inconsistency:
```
User-Agent: Chrome/133
TLS Fingerprint: Chrome 110 pattern
```
Many automation tools rely on HTTP clients or modified TLS libraries that do not perfectly reproduce the handshake behavior of modern browsers. This can lead to inconsistencies between the TLS fingerprint and the browser version claimed in HTTP headers. Real browsers normally keep these values aligned because both are generated by the browser networking stack. As a result, UA vs TLS mismatches can serve as strong automation signals.
### Header Order Consistency
Browsers tend to send headers in a predictable order pattern. An example of a Chromium request header order may look like:
```
Host
Connection
sec-ch-ua
sec-ch-ua-mobile
sec-ch-ua-platform
Upgrade-Insecure-Requests
User-Agent
Accept
Sec-Fetch-Site
Sec-Fetch-Mode
Sec-Fetch-Dest
Referer
Accept-Encoding
Accept-Language
```
Automation frameworks often generate headers differently because they:
```
- Construct headers manually
- Use generic HTTP libraries
- Randomize header order incorrectly
```
An example of an anomaly for our example would be:
```
User-Agent
Accept
Connection
Host
...
```
This may seem minor, but real browser stacks produce very stable header ordering. Because of that, unusual header order can contribute to a weak detection signal. Using header-order alone as a detection signal is very uncommon and would be fragile, but adding this to a multi-signal scoring system can be very effective.
