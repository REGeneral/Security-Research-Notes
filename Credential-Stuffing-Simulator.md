# Designing a Sensor-Based Bot Detection System

While working on the credential stuffing simulator, I started to think about how a basic sensor system might work. The idea initially seemed straightforward. I'd make a page that would collect behavioural signals such as mouse movements, typing patterns and device details (plugins, navigator properties, etc) then send that data to an endpoint like /sensor. The server would assess the payload and return a token that must be included in the /login request. The flow might look something like this:
```
User loads login page
|
Browser collects behavioural and device signals
|
POST /sensor
|
Server returns a sensor token
|
POST /login with sensor token
```
At first glance this seems like a reasonable way to ensure that the client interacting with the login endpoint actually executed the page and generated behaviour data. However, once I started thinking about the attacker perspective multiple weaknesses became obvious.

The first issue I thought of with this was token replay. If the /sensor endpoint returns a token that passes validation once, an attacker could simply capture that token and reuse it for many login attempts. This would defeat the purpose of the sensor system. A simple mitigation would be to introduce expiration for the token, or single-use tokens so that a token can't be reused indefinitely. That solves the most obvious replay problem, but it doesn't solve the deeper issue.

Even if tokens expire quickly, an attacker can simply request a new challenge and generate a new token each time. For example:
```
GET /login page

POST /sensor (same behaviour payload)

valid token returned from /sensor

POST /login
```
If the attacker replays the same mouse movement data, keyboard timings and device properties each time, they could generate an unlimited number of valid tokens. At this point it becomes clear that behaviour data alone can't be trusted. Attackers can simply capture a valid behaviour payload and replay it indefinitely.

My next thought was to rate-limit the /sensor endpoint. If the same payload appears repeatedly, the server could refuse to issue new tokens. However, this can lead to user experience issues. Legitimate users may have similar behavioural patterns and aggressive rate limiting could block real traffic. Not only that, but attackers could bypass simple rate-limits by using rotating proxies/IPs.

A more realistic approach is to treat behaviour as one signal among many rather than something that must be perfectly unique. Instead of blocking immediately, the system can track fingerprints. For example:
```
hash(mouse path+keypress timings+interaction delays)
```
If the same fingerprint appears thousands of times, it becomes a strong indication of automation. Humans naturally produce noisy and inconsistent interaction patterns, whereas bots often generate deterministic behaviour. This allows the system to detect patterns over time without relying on strict rules.
