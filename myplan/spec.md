# Air Quality Activity Advice — Requirements

## ADDED Requirements

### Requirement: Parse and clarify activity context (FR1–FR2)
The system SHALL extract location, time, activity, intensity and relevant user group from a user's query and SHALL ask for missing necessary information before deriving a recommendation.

#### Scenario: Missing activity intensity
- **WHEN** the user asks whether tomorrow's outdoor activity is suitable without specifying intensity
- **THEN** the system asks a clarification question and does not invent an intensity level

### Requirement: Acquire attributable air-quality observations (FR3)
The system SHALL associate acquired AQI observations with station identity, source and observation time, and SHALL detect missing or stale data.

#### Scenario: Observation unavailable
- **WHEN** the requested station has no valid recent observation
- **THEN** the system must not fabricate an AQI value and must indicate that valid data cannot be retrieved if no usable official source remains

### Requirement: Forecast future AQI with uncertainty (FR4)
For future target times within 1–24 hours, the system SHALL provide a model-derived AQI estimate and its estimated uncertainty interval, labeled as predictions.

#### Scenario: Request for the following morning
- **WHEN** a valid future time is supplied
- **THEN** the system uses the forecast engine and labels estimate and interval rather than presenting them as observations

### Requirement: Apply official guidance by population and activity (FR5)
The system SHALL derive recommendations from versioned Taiwanese AQI rules based on AQI level, population group and activity intensity.

#### Scenario: Sensitive group and forecast boundary
- **WHEN** the predicted uncertainty upper bound reaches a higher AQI category and the user belongs to a sensitive group
- **THEN** the designed precautionary logic selects the higher category and records the rule ID

### Requirement: Provide traceable evidence (FR6)
The system SHALL include station, observation or prediction time, rule ID and document provenance for every actionable recommendation.

#### Scenario: Traceable recommendation
- **WHEN** a recommendation is shown
- **THEN** its evidence can be reviewed separately from the LLM's explanation

### Requirement: Return controlled output if citations fail (FR8)
The system SHALL check claims and citations against assembled evidence before emitting generated explanations.

#### Scenario: Unsupported generated AQI value
- **WHEN** the generated answer contains a value absent from the supporting evidence
- **THEN** the answer is replaced with a rule-grounded template and does not expose the unsupported value

### Requirement: Offer alternative times (FR7, Should)
The system SHOULD suggest a better time within the next twelve hours when validated forecasts support a meaningful improvement.

#### Scenario: No supported alternative
- **WHEN** no more suitable forecast slot is identified
- **THEN** the system does not invent a better time
