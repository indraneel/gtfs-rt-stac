# GTFS-RT STAC Extension

This repository contains a [STAC](https://stacspec.org/) (SpatioTemporal Asset Catalog) extension specification for [GTFS-RT](https://gtfs.org/realtime/reference/) (General Transit Feed Specification - Realtime) data.

## Overview

GTFS-RT provides realtime transit information including:
- **Vehicle Positions**: Current location of transit vehicles (lat/lon, speed, bearing)
- **Trip Updates**: Delays, cancellations, and schedule deviations
- **Service Alerts**: Disruptions, stop closures, and other notices

This STAC extension enables cataloging and discovery of GTFS-RT data archives and snapshots, mapping transit feeds to the STAC data model:

| GTFS-RT Concept | STAC Concept | Description |
|-----------------|--------------|-------------|
| Transit Agency Feed | **Collection** | A transit agency's GTFS-RT feed archive |
| Feed Snapshot | **Item** | A single timestamped capture of the feed |
| Protobuf/JSON file | **Asset** | The actual data file(s) |

## Extension Specification

- **Extension ID**: `gtfs-rt`
- **Extension Prefix**: `gtfs-rt:`
- **Scope**: Item, Collection
- **[JSON Schema](json-schema/schema.json)**

## Files

```
gtfs-rt-stac/
├── README.md                    # Extension documentation (this file)
├── PLAN.md                      # Implementation plan
├── json-schema/
│   └── schema.json             # JSON Schema for validation
└── examples/
    ├── collection.json         # Example collection (transit agency)
    ├── item.json               # Example item (feed snapshot)
    └── item-vehicle-focus.json # Example with vehicle statistics
```

## Usage

Include the extension in your STAC Item or Collection:

```json
{
  "stac_extensions": [
    "https://example.com/stac-extensions/gtfs-rt/v1.0.0/schema.json"
  ]
}
```

## Properties

### Collection-Level Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `gtfs-rt:agency_id` | string | **Yes** | Unique identifier for the transit agency |
| `gtfs-rt:agency_name` | string | **Yes** | Human-readable name of the transit agency |
| `gtfs-rt:feed_url` | string (URI) | **Yes** | URL of the live GTFS-RT feed |
| `gtfs-rt:static_feed_url` | string (URI) | No | URL of the associated static GTFS feed |
| `gtfs-rt:feed_types` | string[] | **Yes** | Entity types: `vehicle_positions`, `trip_updates`, `service_alerts` |
| `gtfs-rt:gtfs_rt_version` | string | No | GTFS-RT specification version (e.g., "2.0") |
| `gtfs-rt:update_frequency_seconds` | number | No | Expected feed update interval in seconds |

### Item-Level Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `gtfs-rt:feed_timestamp` | integer | **Yes** | POSIX timestamp from FeedHeader |
| `gtfs-rt:feed_version` | string | No | Matches `feed_info.feed_version` from static GTFS |
| `gtfs-rt:incrementality` | string | **Yes** | `FULL_DATASET` or `DIFFERENTIAL` |
| `gtfs-rt:entity_count` | integer | **Yes** | Total number of FeedEntity objects |
| `gtfs-rt:vehicle_count` | integer | No | Number of VehiclePosition entities |
| `gtfs-rt:trip_update_count` | integer | No | Number of TripUpdate entities |
| `gtfs-rt:alert_count` | integer | No | Number of Alert entities |

### Optional Vehicle Statistics

| Property | Type | Description |
|----------|------|-------------|
| `gtfs-rt:vehicles_in_transit` | integer | Vehicles with status IN_TRANSIT_TO |
| `gtfs-rt:vehicles_stopped` | integer | Vehicles with status STOPPED_AT |
| `gtfs-rt:avg_speed_mps` | number | Average vehicle speed (meters/second) |
| `gtfs-rt:congestion_severe_count` | integer | Vehicles reporting SEVERE_CONGESTION |

### Optional Alert Statistics

| Property | Type | Description |
|----------|------|-------------|
| `gtfs-rt:alerts_by_cause` | object | Count of alerts by cause type |
| `gtfs-rt:alerts_by_effect` | object | Count of alerts by effect type |
| `gtfs-rt:alerts_severe_count` | integer | Alerts with SEVERE severity level |

## Asset Types

| Asset Key | Media Type | Description |
|-----------|------------|-------------|
| `feed` | `application/x-protobuf` | Original protobuf GTFS-RT feed |
| `feed-json` | `application/json` | JSON representation of feed |
| `vehicle-positions` | `application/geo+json` | GeoJSON of vehicle positions |
| `trip-updates` | `application/json` | Extracted trip updates |
| `alerts` | `application/json` | Extracted service alerts |
| `thumbnail` | `image/png` | Map visualization of vehicles |

## Geometry Handling

- **Collections**: Bounding box of the transit system's service area (can be derived from static GTFS `stops.txt`)
- **Items**: Convex hull or bounding box of vehicle positions in the snapshot. Falls back to the collection service area if no vehicle positions are present.

## Link Relations

Items should use `derived_from` link relation to reference the associated static GTFS feed.

## License

CC0-1.0
