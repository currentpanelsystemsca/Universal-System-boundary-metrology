# Direction-aware calibration and flow metadata

## Discovery

This document records a discovery and implementation proposal, not a tested finding.

Direction-aware calibration may matter for bidirectional electrical measurement systems. A single averaged calibration profile can discard information about how a sensor behaves when current flows in opposite directions. In CPS and metrology contexts, it is reasonable to preserve directional context throughout storage, analysis, and explanation rather than reducing it to a scalar average.

Candidate engineering insights:

- Direction-dependent calibration may improve bidirectional measurement accuracy.
- Import and export energy flow may require separate historical baselines, calibration tables, anomaly thresholds, and confidence models.
- Battery charging and battery discharging may represent distinct operating states even when measured current magnitude is similar.
- Sensor drift detection may improve when calibration is evaluated separately for positive and negative current flow directions.

These remain hypotheses until testing validates them.

## Implementation proposal

The corresponding design response is a schema that preserves flow direction, direction-specific calibration, and historical context without asserting a tested conclusion.

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
      "enum": ["import", "export", "unknown"]
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

- Discovery: direction-aware calibration may matter.
- Implementation proposal: the schema models flow direction and direction-specific calibration explicitly.

This matches the repository philosophy of maintaining a clear boundary between hypotheses, tested findings, and proposal work until validation supports a stronger claim.
