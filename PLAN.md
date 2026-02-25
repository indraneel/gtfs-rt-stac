# GTFS-RT STAC Extension - Implementation Plan
## Overview
Create a STAC extension to catalog GTFS-RT (General Transit Feed Specification - Realtime) data, enabling discovery and archival of realtime transit feeds.
## Data Model Mapping
### GTFS-RT Feed Types → STAC Concepts
| GTFS-RT Concept | STAC Concept | Description |
|-----------------|--------------|-------------|
| Transit Agency Feed | **Collection** | A transit agency's GTFS-RT feed archive |
| Feed Snapshot | **Item** | A single timestamped capture of the feed |
| Protobuf/JSON file | **Asset** | The actual data file(s) |
### GTFS-RT Entity Types
The spec defines three main feed entity types that can be cataloged:
1. **VehiclePosition** - Realtime vehicle locations (lat/lon, speed, bearing)
2. **TripUpdate** - Schedule deviations, delays, cancellations
3. **Alert** - Service alerts, disruptions, closures
## Extension Properties
### Prefix: `gtfs-rt:`
### Collection-Level Properties
| Property | Type | Description |
|----------|------|-------------|
| `gtfs-rt:agency_id` | string | Unique identifier for the transit agency |
| `gtfs-rt:agency_name` | string | Human-readable agency name |
| `gtfs-rt:feed_url` | string | URL of the live GTFS-RT feed |
| `gtfs-rt:static_feed_url` | string | URL of associated static GTFS feed |
| `gtfs-rt:feed_types` | string[] | Entity types available: `vehicle_positions`, `trip_updates`, `service_alerts` |
| `gtfs-rt:gtfs_rt_version` | string | GTFS-RT spec version (e.g., "2.0") |
| `gtfs-rt:update_frequency_seconds` | number | Expected feed update interval |
### Item-Level Properties
| Property | Type | Description |
|----------|------|-------------|
| `gtfs-rt:feed_timestamp` | number | POSIX timestamp from FeedHeader |
| `gtfs-rt:feed_version` | string | Matches feed_info.feed_version from static GTFS |
| `gtfs-rt:incrementality` | string | `FULL_DATASET` or `DIFFERENTIAL` |
| `gtfs-rt:entity_count` | number | Total number of FeedEntity objects |
| `gtfs-rt:vehicle_count` | number | Number of VehiclePosition entities |
| `gtfs-rt:trip_update_count` | number | Number of TripUpdate entities |
| `gtfs-rt:alert_count` | number | Number of Alert entities |
### Optional Vehicle Statistics (for vehicle position feeds)
| Property | Type | Description |
|----------|------|-------------|
| `gtfs-rt:vehicles_in_transit` | number | Vehicles with status IN_TRANSIT_TO |
| `gtfs-rt:vehicles_stopped` | number | Vehicles with status STOPPED_AT |
| `gtfs-rt:avg_speed_mps` | number | Average vehicle speed (meters/second) |
| `gtfs-rt:congestion_severe_count` | number | Vehicles reporting severe congestion |
### Optional Alert Statistics (for service alert feeds)
| Property | Type | Description |
|----------|------|-------------|
| `gtfs-rt:alerts_by_cause` | object | Count of alerts by cause type |
| `gtfs-rt:alerts_by_effect` | object | Count of alerts by effect type |
| `gtfs-rt:alerts_severe_count` | number | Alerts with SEVERE severity |
## Geometry Handling
### Collection Geometry
- Bounding box of the transit system's service area
- Can be derived from static GTFS stops.txt
### Item Geometry
- **Option A (recommended)**: Convex hull of all vehicle positions in the snapshot
- **Option B**: Bounding box of vehicle positions
- **Option C**: Same as collection (service area) if no vehicle positions
## Asset Types
### Primary Assets
| Asset Key | Media Type | Description |
|-----------|------------|-------------|
| `feed` | `application/x-protobuf` | Original protobuf GTFS-RT feed |
| `feed-json` | `application/json` | JSON representation of feed |
### Derived Assets
| Asset Key | Media Type | Description |
|-----------|------------|-------------|
| `vehicle-positions` | `application/geo+json` | GeoJSON of vehicle positions |
| `trip-updates` | `application/json` | Extracted trip updates |
| `alerts` | `application/json` | Extracted service alerts |
| `thumbnail` | `image/png` | Map visualization of vehicles |
## File Structure
```
gtfs-rt-stac/
├── README.md                    # Extension documentation
├── PLAN.md                      # This file
├── json-schema/
│   └── schema.json             # JSON Schema for validation
├── examples/
│   ├── collection.json         # Example collection (transit agency)
│   ├── item.json               # Example item (feed snapshot)
│   └── item-vehicle-focus.json # Example with vehicle statistics
```
## Example Use Cases
### 1. Archiving Historical GTFS-RT Data
Store snapshots of GTFS-RT feeds for historical analysis, research, and machine learning.
### 2. Multi-Agency Transit Data Discovery
Enable searching across multiple transit agencies to find feeds covering specific areas or time periods.
### 3. Transit Performance Analysis
Query historical vehicle positions and delays for service quality analysis.
### 4. Service Alert Archives
Catalog service disruptions over time for pattern analysis.
## Implementation Steps
1. **Create Extension README** - Full documentation with field descriptions
2. **Create JSON Schema** - For validation of collections and items
3. **Create Example Collection** - Sample transit agency collection
4. **Create Example Items** - Sample feed snapshot items
5. **Add Asset Role Definitions** - Define standard asset roles for GTFS-RT
## Questions to Consider
1. **Granularity**: Should individual vehicles or trips be separate Items, or should a feed snapshot be one Item?
   - **Recommendation**: Feed snapshot = one Item (simpler, matches how GTFS-RT is distributed)
2. **Geometry precision**: How precise should vehicle position geometries be?
   - **Recommendation**: Use convex hull for Items, service area bbox for Collections
3. **Update frequency**: How often should snapshots be taken?
   - **Recommendation**: Define in collection metadata; typical is 15-30 seconds
4. **Static GTFS linkage**: How to link to the associated static GTFS?
   - **Recommendation**: Use `gtfs-rt:static_feed_url` property and optional `derived_from` link relation
## References
- [GTFS-RT Specification](https://gtfs.org/realtime/reference/)
- [GTFS-RT Protobuf Definition](transit/gtfs-realtime/proto/gtfs-realtime.proto)
- [STAC Specification](https://stacspec.org/)
- [STAC Extension Template](https://github.com/stac-extensions/template)
