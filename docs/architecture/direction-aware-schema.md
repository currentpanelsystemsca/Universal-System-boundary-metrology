# Direction-aware calibration and flow metadata

## Research Observations

### Direction-Aware Calibration
Date: 2026-10-30
Status: Research observation (not validated in CPS)

Published Hall-sensor and motor-control calibration examples indicate that
forward and reverse operation may not always produce identical measured
behavior. Some systems perform separate calibration passes for each
direction and preserve direction-specific correction information rather
than averaging both directions into a single model.

Potential CPS relevance:

- Power-flow direction may be a first-class property of a measurement.
- Import and export energy flow may require separate historical baselines.
- Battery charging and battery discharging may require separate operating
  models even when current magnitude is identical.
- Direction-dependent behavior should be preserved until testing shows
  that directional differences are insignificant.

Working CPS principle:

"Do not average away directional behavior until equivalent behavior has
been demonstrated."

Design implications for future investigation:

- Direction-aware measurement packets
- Direction-specific calibration profiles
- Direction-specific anomaly thresholds
- Direction-specific historical baselines
- Direction-aware explanation generation within Buddy

Examples:

IMPORT
  Utility -> Building

EXPORT
  Building -> Utility

CHARGING
  Energy -> Battery

DISCHARGING
  Battery -> Load

What this does not prove:

This observation does not establish that CPS sensors will exhibit
direction-dependent calibration effects. It only suggests that
direction-dependent behavior should be preserved and available for
analysis until testing determines whether separate calibration models
provide meaningful benefit.

Evidence level:

Research observation only.
No CPS hardware testing has been performed.
Requires validation against reference instrumentation.

## Implementation proposal

The JSON Schema below is a design response to this observation. It is not
proof of the observation and it does not claim that separate calibration
models are required. It is a way to preserve flow direction, directional
calibration context, and historical context so the system can test whether
those distinctions are useful.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "MeasurementRecord",
  "type": "object",
  "required": ["timestamp", "measurementType", "flowDirection"],
  "properties": {
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "measurementType": {
      "type": "string",
      "enum": ["current", "voltage", "power", "energy", "temperature"]
    },
    "flowDirection": {
      "type": "string",
      "enum": ["import", "export", "charging", "discharging", "unknown"]
    },
    "value": {
      "type": "number"
    },
    "units": {
      "type": "string"
    },
    "directionSpecificCalibration": {
      "type": "object",
      "properties": {
        "import": { "type": "object" },
        "export": { "type": "object" },
        "charging": { "type": "object" },
        "discharging": { "type": "object" },
        "notes": { "type": "string" }
      },
      "additionalProperties": true
    },
    "historicalContext": {
      "type": "object",
      "properties": {
        "baselineId": { "type": "string" },
        "previousSampleTimestamp": { "type": "string", "format": "date-time" },
        "driftScore": { "type": "number" },
        "confidence": { "type": "number", "minimum": 0, "maximum": 1 }
      },
      "additionalProperties": true
    }
  },
  "additionalProperties": true
}
```

## Why this fits the repository

This design keeps the discovery and the implementation separate:

- Discovery: direction itself may carry relevant information.
- Implementation: the schema preserves that information for analysis.

This is consistent with the repository rule that observations, hypotheses,
and validated findings should remain distinct until evidence supports a
stronger conclusion.
