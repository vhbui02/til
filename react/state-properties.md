# State properties

1. Local: exactly one component subtree needs it; no other features, routes or pages.
2. Transient: the value is regenerated from scratch every time the component mounts and has no value outside that mount. Navigate way and comeback, a fresh session is created and old values are meaningless. E.g., `firstFrameAt`, `connecting` status. >< persisted state (`localStorage` or Redux)
3. High-churn: the value changes often relative to how often it's read. E.g., a video stream's statistics which gets updated every 2000ms while the video plays.
