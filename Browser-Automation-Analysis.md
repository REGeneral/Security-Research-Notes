# Browser Automation Analysis

This document outlines several HTTP level signals that can be used to detect automated traffic. These signals are based on browser consistency validation, protocol anomalies, and header structure analysis. Modern bot detection systems rarely rely on a single signal. Instead, they combine many weak signals to identify automated behavior.

### Client Hint Consistency

Modern browsers send User-Agent Client Hints such as:

sec-ch-ua  
sec-ch-ua-mobile  
sec-ch-ua-platform  

These headers are typically sent together as a cluster. Automation frameworks frequently include only one or two of these headers, creating inconsistencies that can be used as detection signals. Real browsers normally send these headers together because they are generated automatically by the browser networking stack.

### Casing Signals

Similar to client hints, browsers all use specific casing. For instance, both firefox and chrome send the user agent header as so:

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
The content-length header is the size of the request body in bytes.

Another signal I've come across through my research, a lot of the time automation frameworks will either miscalculate the content-length of a request, send two content-length headers, or they will use a static content-length header.

POST https://test.com/api/endpoint

Content-Length: 38

Request Body:
awidjawd%2ecom

Actual length: 14
