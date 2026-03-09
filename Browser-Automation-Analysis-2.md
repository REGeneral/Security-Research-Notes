# Browser Automation Analysis: Client Environment & Behavioral Signals
This section focuses on signals that can help identify automated browsers by analysing inconsistencies in the client environment and user interaction patterns. Unlike protocol-level signals, which look at HTTP request structure and network behavior, these signals originate from within the browser runtime itself. They are typically derived from browser APIs, rendering behavior and observable user interactions. Automation frameworks such as Playwright, Puppeteer and Selenium attempt to replicate real browser environments, but subtle inconsistencies tend to remain. These inconsistencies can be used as detection signals when trying to decipher whether a client is likely automated.

### WebGL Renderer Consistency
Browsers expose GPU information through the WebGL API, including the renderer and vendor. These values usually come from the user’s hardware and operating system. Because these values arise from the local system environment, they normally remain consistent with other browser signals such as:
```
- Operating System
- Browser family
- User-Agent string
```
Automation frameworks that attempt to imitate browser environments sometimes produce inconsistent combinations of these attributes.

e.g anomaly:
```
User-Agent: Chrome / Windows
WebGL Renderer: Apple M1 GPU
```
In this example, the reported GPU is Apple hardware, whereas the User-Agent claims a windows environment. These types of inconsistencies can occur when automation tools modify the User-Agent string or other browser properties without correctly emulating the underlying system environment. Because legitimate browsers rarely produce these mismatches, WebGL renderer inconsistencies can contribute to weak detection signals within a broader risk-scoring system.

### Viewport / Screen Consistency
Browsers also expose several properties regarding screen size and viewport dimensions. These include values such as:
```
- screen.width
- screen.height
- window.innerWidth
- window.innerHeight
- devicePixelRatio
```
In real browsers, these values usually form consistent patterns that reflect the user's display environment.
e.g anomaly:
```
screen.width = 1920
window.innerWidth = 6000
```
Automation frameworks sometimes generate unrealistic combinations of these values, especially when attempting to randomize browser fingerprints. This can be used as another weak signal in a broader risk-scoring system.
### Navigator Property Consistency

Browsers expose information about the client environment through the JS (JavaScript) navigator object. These properties provide details about the user's system configuration and browser environment.

Some common examples include:
```
- navigator.webdriver
- navigator.languages
- navigator.plugins
- navigator.hardwareConcurrency
- navigator.deviceMemory
```
In real browsers, these values typically reflect the user's real system configuration and remain consistent with other browser signals such as the operating system, hardware capabilities and language settings. Automation frameworks sometimes attempt to mock or override these properties, but inconsistencies may still occur. For example, automation environments may return empty plugin lists, unrealistic hardware values, or unusual language configurations.

Example anomalies may include:
```
navigator.languages = []  
navigator.plugins = []  
navigator.hardwareConcurrency = unrealistic values
navigator.webdriver = true
```
While none of these properties are reliable detection signals on their own, inconsistencies within navigator properties can contribute to a broader set of client environment signals used in automation detection systems.
### Behavioural Signals
Another strong signal that can be used in a risk-scoring system is the mouse movement patterns of the user. Real users tend to have curved paths, variable velocity and pauses between each interaction. Bots typically have linear movement, constant velocity and have instant jumps to interact with the website.

Tracking the user's typing speed is also another strong signal. Real typing usually has a 80ms-250ms keypress interval, whereas automation typically has a 0ms interval or perfectly uniform amounts between each keypress.

User's interaction speeds are also a signal that is commonly used to identify automation. For example, if a page has just loaded but a login is submitted within 50ms, that is a strong signal that automation is likely. 

### Conclusion
Automation detection rarely relies on a single signal. Many of the signals discussed in this document and the previous are weak when evaluated in isolation, but become far more reliable when combined together. Modern bot mitigation systems generally evaluate multiple layers of client behaviour, including protocol-level signals, HTTP request structure, client environment fingerprints and observable interaction patterns. By connecting inconsistencies across these layers, detection systems can build a probabilistic model that helps differentiate real users from automated clients. As automation frameworks continue to evolve and attempt to replicate real browser environments more accurately, detection techniques must also adapt. This is why effective bot detection systems generally rely on combining multiple signals rather than a single one. 
