# Emesary - Inter-Object Communication System

Technical overview of the Emesary notification system used throughout the F-14.

---

## Overview

**Emesary** is a publish-subscribe notification system that enables communication between disparate classes that have no knowledge of each other. It implements the **Observer pattern** using a decoupled message-passing architecture.

**Author:** Richard Harrison (richard@zaretto.com)

**Version:** 4.8

**License:** GPL V2

**References:**
- http://chateau-logic.com/content/emesary-nasal-implementation-flightgear
- http://www.chateau-logic.com/content/class-based-inter-object-communication
- http://wiki.flightgear.org/Emesary_Notifications

---

## Core Concept

Traditional object communication requires objects to know about each other:

```nasal
# Traditional approach - tight coupling
radar.lockTarget(target);
weapons.setTarget(target);
hud.displayTarget(target);
```

Emesary decouples this:

```nasal
# Emesary approach - loose coupling
var notification = TargetNotification.new(target);
emesary.GlobalTransmitter.NotifyAll(notification);

# Radar, weapons, and HUD all receive notification
# None need to know about the others
```

**Key benefit:** Classes can be added, removed, or modified without affecting others. The radar doesn't know the weapons system exists, and vice versa.

---

## Architecture

### Three Core Classes

**1. Transmitter**

Sends notifications to all registered recipients.

```nasal
var myTransmitter = emesary.Transmitter.new("MyTransmitter");
myTransmitter.NotifyAll(notification);
```

**Global Transmitter:** Most systems use the singleton `emesary.GlobalTransmitter` rather than creating their own.

**2. Notification**

A message object containing data to be transmitted.

```nasal
var Notification = {
    new: func(type, ident, typeId) {
        return {
            parents: [Notification],
            NotificationType: type,    # String identifying notification type
            Ident: ident,              # Optional identifier or value
            TypeId: typeId,            # Unique numeric ID (for MP bridging)
            IsDistinct: 1,             # 1=only latest matters, 0=queue all
        };
    }
};
```

**3. Recipient**

An object that receives notifications by implementing a `Receive()` method.

```nasal
var MyRecipient = emesary.Recipient.new("MyRecipient");
MyRecipient.Receive = func(notification) {
    if (notification.NotificationType == "TargetLocked") {
        # Handle the notification
        return emesary.Transmitter.ReceiptStatus_OK;
    }
    return emesary.Transmitter.ReceiptStatus_NotProcessed;
};

emesary.GlobalTransmitter.Register(MyRecipient);
```

---

## Usage Pattern

### 1. Define a Notification Type

```nasal
var TargetNotification = {
    new: func(target) {
        var n = emesary.Notification.new("TargetNotification", target.callsign, 42);
        n.target = target;
        n.range = target.range;
        return n;
    }
};
```

### 2. Create Recipients

```nasal
# Radar system
var RadarRecipient = emesary.Recipient.new("RadarSystem");
RadarRecipient.Receive = func(notification) {
    if (notification.NotificationType == "TargetNotification") {
        me.updateRadarDisplay(notification.target);
        return emesary.Transmitter.ReceiptStatus_OK;
    }
    return emesary.Transmitter.ReceiptStatus_NotProcessed;
};
emesary.GlobalTransmitter.Register(RadarRecipient);

# Weapons system
var WeaponsRecipient = emesary.Recipient.new("WeaponsSystem");
WeaponsRecipient.Receive = func(notification) {
    if (notification.NotificationType == "TargetNotification") {
        me.lockMissile(notification.target);
        return emesary.Transmitter.ReceiptStatus_OK;
    }
    return emesary.Transmitter.ReceiptStatus_NotProcessed;
};
emesary.GlobalTransmitter.Register(WeaponsRecipient);
```

### 3. Send Notifications

```nasal
# From anywhere in the code - sender doesn't know who's listening
var notification = TargetNotification.new(currentTarget);
emesary.GlobalTransmitter.NotifyAll(notification);
```

Both radar and weapons systems receive the notification, even though:
- The sender doesn't know they exist
- They don't know about each other
- They can be added/removed independently

---

## Receipt Status Codes

Recipients must return a status code from `Receive()`:

| Status | Value | Meaning |
|--------|-------|---------|
| `ReceiptStatus_OK` | 0 | Processed successfully |
| `ReceiptStatus_Fail` | 1 | Processing failed |
| `ReceiptStatus_Abort` | 2 | Fatal error, stop processing |
| `ReceiptStatus_Finished` | 3 | Definitively handled, stop processing |
| `ReceiptStatus_NotProcessed` | 4 | Didn't handle this notification |
| `ReceiptStatus_Pending` | 5 | Processing underway, status unknown |
| `ReceiptStatus_PendingFinished` | 6 | Handled, status unknown, stop processing |

**Typical usage:**
- Return `OK` if you handled the notification
- Return `NotProcessed` if you ignored it (not your type)
- Return `Finished` if you handled it and no one else should process it

---

## Multiplayer Bridge

Emesary notifications can be **bridged** across multiplayer using the property tree.

### How MP Bridging Works

1. **Outgoing Bridge** serializes notifications to MP properties
2. **FlightGear multiplayer** transmits properties via UDP
3. **Incoming Bridge** on other clients deserializes and re-transmits locally

```
Aircraft A:                              Aircraft B:
Notification sent
    ↓
OutgoingMPBridge receives
    ↓
Serializes to sim/multiplay/emesary/bridge[19]
    ↓
           === UDP Transmission ===
                                   ↓
                        IncomingMPBridge reads property
                                   ↓
                        Deserializes notification
                                   ↓
                        Transmits to local recipients
```

### Bridge Configuration

**Outgoing Bridge:**

```nasal
var routedNotifications = [
    ArmamentNotification.new(),
    GeoBridgedNotification.new()
];

var outgoingBridge = emesary_mp_bridge.OutgoingMPBridge.new(
    "F-14",                    # Aircraft identifier
    routedNotifications,       # Notifications to bridge
    19                         # MP property index
);
```

**Incoming Bridge:**

```nasal
var incomingBridge = emesary_mp_bridge.IncomingMPBridge.startMPBridge(
    routedNotifications
);
```

### Bridge Properties

Notifications are transmitted using a single MP property:

- `sim/multiplay/emesary/bridge[19]` - Message string (default: 128 characters, configurable)

### Message Format

```
!typeid!sequence!serialized_data~
```

- `!` separates fields
- `~` marks message end
- `typeid`: Numeric notification type (must be < 16 for user notifications)
- `sequence`: Prevents duplicate processing
- `serialized_data`: Semicolon-separated notification fields

### Message Reliability

Emesary MP bridges provide **near-reliable** message delivery over UDP:

1. **Repeated Transmission:** Messages are retransmitted for their lifetime (default 10 seconds)
2. **Sequence Tracking:** Each message has a unique sequence ID
3. **Duplicate Filtering:** Incoming bridges ignore already-processed sequence IDs
4. **High Probability:** Design ensures most clients receive most messages within the lifetime
5. **Not Guaranteed:** UDP packet loss and timing can cause missed messages

**Why not 100% reliable?**
- UDP doesn't guarantee delivery or ordering
- Clients may join mid-transmission
- Property updates may be dropped under high load
- Sequence ID filtering prevents reprocessing, so a missed message is missed permanently

### IsDistinct Flag

Notifications have an `IsDistinct` property:

- **IsDistinct = 1 (true):** Only the latest message matters. Older queued messages are replaced.
  - Example: Position updates (only current position relevant)
- **IsDistinct = 0 (false):** All messages must be queued and transmitted.
  - Example: Missile launches (can't miss any event)

---

## F-14 Notification Types

The F-14 uses Emesary extensively for system integration:

| Notification | TypeId | Purpose | Bridged |
|--------------|--------|---------|---------|
| `ArmamentNotification` | varies | Missile launches, impacts, damage | Yes |
| `ArmamentInFlightNotification` | varies | Missile telemetry during flight | Yes |
| `GeoBridgedNotification` | varies | Position/velocity data | Yes |
| `AINotification` | - | AI contact list updates | No |
| `SliceNotification` | - | Radar slice requests | No |
| `WeaponNotification` | - | Current weapon selection | No |
| `WeaponRequestNotification` | - | MFD requesting weapon info | No |
| `AircraftEventNotification` | - | Aircraft events (takeoff, landing, etc.) | Optional |

### Example: Missile Launch

When a missile is fired:

1. **Fire Control** sends `ArmamentNotification` with type "mlaunched"
2. **Outgoing Bridge** serializes it to MP properties
3. **Other MP players** receive via incoming bridge
4. **Their systems** process notification:
   - Radar system: Track missile
   - Damage system: Prepare for impact
   - RWR system: Display launch warning
   - Tacview: Log launch event

All without any system knowing about the others.

---

## Benefits

**Decoupling:**
- Systems don't depend on each other
- Can add/remove/modify systems independently
- Easy to test systems in isolation

**Extensibility:**
- New systems can listen to existing notifications
- No changes needed to senders

**Multiplayer:**
- Automatic synchronization across clients
- Consistent state without custom networking code

**Performance:**
- Only active recipients process notifications
- No polling required
- Efficient property tree bridging

---

## Performance Considerations

**Overrun Detection:**

Transmitters can detect slow recipients:

```nasal
emesary.GlobalTransmitter.OverrunDetection(10); # Warn if >10ms
```

**Recipient Count:**

Keep recipient count reasonable. `GlobalTransmitter` may have 20-50 recipients.

**Notification Frequency:**

- High-frequency notifications (position updates): Use `IsDistinct=1`
- Event notifications (weapon launches): Use `IsDistinct=0`

**MP Bridge Bandwidth:**

- MP property string length: Configurable (default 128 characters)
- Update rate: Configurable (default 1 Hz)
- Message lifetime: Configurable (default 10 seconds)
  - Messages retransmitted for the lifetime duration
  - Incoming bridges track sequence IDs and only process new messages
  - Design provides "almost reliable" delivery (high probability all clients receive it)
  - Not 100% reliable due to UDP nature and client-side duplicate filtering

---

## Debugging

**Print Recipients:**

```nasal
emesary.GlobalTransmitter.PrintRecipients();
```

**Trace Notifications:**

```nasal
outgoingBridge.trace = 1; # Enable detailed logging
```

**Check Receipt Status:**

```nasal
var status = emesary.GlobalTransmitter.NotifyAll(notification);
if (emesary.Transmitter.IsFailed(status)) {
    print("Notification failed!");
}
```

---

## Implementation in F-14

### Systems Using Emesary

**Nasal/emesary.nas (aircraft local copy):**
- Core Emesary implementation
- May differ from FlightGear core version

**Nasal/emesary_mp_bridge.nas:**
- Multiplayer bridging
- Serialization/deserialization

**Notification Sources:**
- Nasal/ArmamentNotification.nas
- Nasal/GeoBridgedTransmitter.nas
- Nasal/M_frame_notification.nas

**Major Recipients:**
- Nasal/Radar/radar-system.nas (AINotification)
- Nasal/fire-control.nas (WeaponNotification)
- Nasal/damage.nas (ArmamentNotification)
- Nasal/station-manager.nas (WeaponNotification)
- Nasal/tacview.nas (ArmamentNotification)

### Typical Flow: Target Lock

```
1. Radar detects target
2. User presses lock key
3. Radar sends: TargetNotification
   ↓
4. Recipients receive:
   - HUD: Display target box
   - Weapons: Slave seeker
   - RIO displays: Update TID
   - Datalink: Transmit to wingmen (via MP bridge)
```

No system knows about the others. Each just listens for `TargetNotification`.

---

## References

**Emesary Documentation:**
- http://chateau-logic.com/content/emesary-nasal-implementation-flightgear
- http://wiki.flightgear.org/Emesary_Notifications

**FlightGear Core Files:**
- `fgdata/Nasal/emesary.nas` - Core implementation
- `fgdata/Nasal/emesary_mp_bridge.nas` - Multiplayer bridging

**F-14 Files:**
- See RADAR.md for radar system notifications
- See WEAPONS_SYSTEM.md for weapons notifications

---

For radar system usage, see [RADAR.md](RADAR.md).

For weapons system usage, see [WEAPONS_SYSTEM.md](WEAPONS_SYSTEM.md).

For complete aircraft documentation, see [README.md](README.md).
