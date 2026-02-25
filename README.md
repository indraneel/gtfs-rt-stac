# GTFS-RT STAC Extension

This repository contains a STAC (SpatioTemporal Asset Catalog) extension specification for GTFS-RT (General Transit Feed Specification - Realtime) data.

## Overview

GTFS-RT provides realtime transit information including:
- **Vehicle Positions**: Current location of transit vehicles
- **Trip Updates**: Delays, cancellations, and schedule deviations
- **Service Alerts**: Disruptions, stop closures, and other notices

This STAC extension enables cataloging and discovery of GTFS-RT data archives and snapshots.

## Extension Specification

- **Extension ID**: `gtfs-rt`
- **Extension Prefix**: `gtfs-rt:`
- **Scope**: Item, Collection

## Files

- `extension.json` - JSON Schema for the extension
- `collection.json` - Example STAC Collection for a transit agency
- `item.json` - Example STAC Item for a GTFS-RT snapshot

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

| Property | Type | Description |
|----------|------|-------------|
| `gtfs-rt:feed_type` | string[] | Types of data in feed: `vehicle_positions`, `trip_updates`, `service_alerts` |
| `gtfs-rt:agency_id` | string | Unique identifier for the transit agency |
| `gtfs-rt:agency_name` | string | Human-readable name of the transit agency |
| `gtfs-rt:feed_url` | string | URL of the live GTFS-RT feed |
| `gtfs-rt:static_feed_url` | string | URL of the associated static GTFS feed |
| `gtfs-rt:vehicle_count` | integer | Number of vehicles in snapshot |
| `gtfs-rt:trip_count` | integer | Number of trips with updates |
| `gtfs-rt:alert_count` | integer | Number of active service alerts |
| `gtfs-rt:gtfs_rt_version` | string | GTFS-RT specification version |
| `gtfs-rt:update_frequency` | number | Expected update frequency in seconds |

## License

CC0-1.0
