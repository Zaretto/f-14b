# Nasal/fallback — Compatibility Layer

The `Nasal/fallback/` subdirectory provides a compatibility layer that allows the F-14 to work with older FlightGear versions while benefiting from newer framework features. If a module is already present in FlightGear's core data (fgdata), the core version is used; otherwise, the fallback version from this subdirectory is loaded.

## Files

| File | Purpose |
|------|---------|
| readme.txt | Configuration guide for fallback module XML declarations |
| fallback-module.nas | Runtime loader — checks globals and loads missing modules |
| emexec.nas | Emesary-based real-time module executive |
| UpdateManager.nas | Conditional update handler with delta thresholds |
| frame_utils.nas | Workload partitioning across frames |

## How It Works

Fallback modules are declared in the aircraft configuration XML (f-14-common.xml) under a `<fallback><nasal>` section. On startup:

1. `fallback-module.nas` iterates through all child nodes under `/fallback/nasal`
2. For each module, it checks if it already exists in the global namespace via `contains(globals, module)`
3. If the module is not present in FlightGear core, it loads the fallback version using `io.load_nasal()`
4. All loading operations are logged for debugging

This means the F-14 runs on FlightGear 2017.3+ but automatically uses the improved core versions when running on 2020.4+.

## Module Details

### emexec.nas — Frame Executive

Author: Richard Harrison (2018)

Provides frame-synchronised execution for aircraft subsystem updates via the Emesary notification system.

**Key classes:**

- **EmesaryExecutive** (`ExecModule` singleton) — creates a timer that fires based on current frame rate, dynamically adjusting from 1 Hz to a configurable maximum (default 30 Hz). Uses a low-pass filter to smooth frame rate calculations. Waits for `sim/signals/fdm-initialized` before starting.

- **FrameNotification** — sent every frame to all registered recipients. Monitors properties specified via `FrameNotificationAddProperty` notifications. Maintains a hash of current property values, tracks frame count, elapsed time, and frame rate.

- **OperationTimer** — profiling aid for measuring execution time breakpoints within classes.

**Registration:**
```
ExecModule.register(ident, properties_to_monitor, object, rate, frame_offset)
```
- `rate > 1` means process every Nth frame (frame skipping)
- `frame_offset` staggers multiple subscribers across frames
- Object must have an `update(notification)` method

### UpdateManager.nas — Conditional Updates

Author: Richard Harrison

Reduces CPU load by only triggering callbacks when monitored values change beyond a specified delta threshold. Taken from FlightGear 2020.4 core.

**Key methods:**

- `FromProperty(propname, delta, callback)` — monitors a single property; triggers when numeric change exceeds delta, or on any string change
- `FromPropertyHashList(keylist, delta, callback)` — monitors multiple properties; triggers if any exceeds delta
- `FromHashValue(key, delta, callback)` — monitors a key in a hash (e.g. from Emesary notification data)
- `FromHashList(keylist, delta, callback)` — monitors multiple hash keys; triggers if any exceeds delta

Primarily used by canvas and instrument display systems to avoid unnecessary redraws.

### frame_utils.nas — Workload Partitioning

Author: Richard Harrison (2019)

Distributes intensive processing across multiple frame cycles to maintain real-time performance.

**Key class:**

- **PartitionProcessor** — processes `partition_size` elements per invocation (e.g. 20 per frame). Can also limit by time using `set_max_time_usec()` (recommended: 500 microseconds).

Three callback functions:
1. `init_method(pp, obj, data)` — called once when starting a fresh pass
2. `process_method(pp, obj, element)` — called for each element; return 1 to continue, 0 to stop
3. `complete_method(pp, obj, data)` — called when all elements are processed

Used by the radar system to process large contact lists incrementally without degrading frame rate.

## Relationships

```
fallback-module.nas  (Loader)
    │
    ├── emexec.nas  (Frame Executive)
    │   └── Coordinates all module updates at controlled rates
    │
    ├── UpdateManager.nas  (Conditional Updates)
    │   └── Used by display/canvas systems to avoid unnecessary redraws
    │
    └── frame_utils.nas  (Workload Partitioning)
        └── Used by radar and other systems for incremental processing
```

The three modules work together: `emexec` drives the update loop, `UpdateManager` filters out unchanged values to reduce display updates, and `frame_utils` partitions heavy computation across frames.
