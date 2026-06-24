# Lantern Protocol

**Lantern Protocol** is a DIY Home Assistant project for reclaiming attention from the phone. It turns selected notifications into ambient light, creating a small domestic firewall between communication and compulsive device checking.

The goal is not to disconnect from people or household signals. The goal is to move technology-mediated communication to a respectful distance. A call, message, doorbell, or garage event can still enter the room, but only through the channels you choose, in a form that does not demand a screen.

Lantern Protocol treats attention as something that should require permission.

## Design intent

Modern phones collapse signals into attention traps with constant stream of notifications that require work to opt-out of at best and at their worst clog your screen and fill you with anxiety.

Lantern Protocol separates the **signal** from the **device**. 

Instead of picking up the phone to see whether something matters, you define a short list of things that are allowed to pass through that are worthy of your attention through whitelists:

- a human is calling
- a human sent you a message
- a human is at your door
- a car is coming in or out of your garage
- your garage has been open too long

Those signals become light throughout your house. Your phone can stay elsewhere. The screen does not become the interface for your attention.

This is a small project, but it belongs to a larger ethic: people should be able to reshape domestic technology around autonomy, consent, privacy, and calm.

## What this does

Lantern Protocol uses Home Assistant to create an ambient notification queue. When an allowed event happens, a helper turns on. A renderer blueprint reads the active helpers and rotates a color-capable bulb through the active notification colors.

It supports setups where a smart plug physically powers the bulb:

1. A selected trigger turns on a queue helper, such as `input_boolean.notification_queue_phone_call`.
2. The renderer turns on the power switch if needed.
3. The renderer waits for the bulb to come online.
4. The renderer overwrites whatever color the bulb remembered from its last power state.
5. The bulb cycles through the active notification colors.
6. Turning the power switch off clears the queue.


## Included files

```text
blueprints/automation/notification_queue_renderer.yaml
blueprints/automation/state_to_queue_action.yaml
blueprints/automation/state_held_to_queue_action.yaml
blueprints/automation/event_to_queue_action.yaml
blueprints/automation/android_last_notification_to_queue_action.yaml
packages/notification_queue_helpers.yaml
examples/example_automations.yaml
```

## Core concept

Lantern Protocol has two layers:

### 1. Queue state

Home Assistant helper toggles remember which kinds of notifications are currently active.

The optional package creates these default helpers:

```text
input_boolean.notification_queue_doorbell
input_boolean.notification_queue_garage_opened
input_boolean.notification_queue_garage_left_open
input_boolean.notification_queue_phone_call
input_boolean.notification_queue_phone_message
```

These helpers are not an inbox. They do not store message contents. They represent categories of attention that you have allowed into the room.

### 2. Ambient rendering

The renderer blueprint turns the queue into light.

Suggested default meanings:

```text
Doorbell:          white
Garage opened:     green
Garage left open:  orange
Phone call:        red
Phone message:     blue
```

If more than one helper is active, the bulb rotates through the colors. Turning the configured power switch off clears the queue.

## Installation

### 1. Add the optional helper package

Copy `packages/notification_queue_helpers.yaml` into your Home Assistant `/config/packages/` folder.

Make sure packages are enabled in `configuration.yaml`:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

Restart Home Assistant.

You can also create equivalent helper toggles manually through the Home Assistant UI.

### 2. Import the blueprints

In Home Assistant:

```text
Settings → Automations & scenes → Blueprints → Import Blueprint
```

Paste the raw GitHub URL for each blueprint file.

Home Assistant supports importing blueprints from GitHub, GitHub gists, and the Home Assistant forums.

### 3. Create a renderer automation

Use:

```text
blueprints/automation/notification_queue_renderer.yaml
```

Configure:

- the color-capable light
- the switch or smart plug that powers the light
- the helper toggles that make up the queue
- the colors and brightness levels
- boot/wait timing for bulbs that need time to reconnect
- whether turning the power switch off clears the queue

### 4. Create source automations

Use the trigger blueprints to decide what is allowed through.

Suggested automations:

```text
Garage opens                         → garage opened helper
Garage remains open for 10 minutes   → garage left open helper
Phone state becomes ringing          → phone call helper
Doorbell event changes               → doorbell helper
Allowed Android notification appears → phone message helper
```

This is where the permission model lives. Only create automations for signals you actually want to pass through.

## Blueprints

### `notification_queue_renderer.yaml`

Renders active queue helpers as colors on a light. It can power on a smart plug first, wait for the bulb, then overwrite the bulb's previous color.

Use this once per notification lamp.

### `state_to_queue_action.yaml`

Turns on a selected helper when an entity changes to a target state.

Useful for:

```text
phone state → ringing
garage door → open
```

### `state_held_to_queue_action.yaml`

Turns on a selected helper when an entity remains in a target state for a duration.

Useful for:

```text
garage door → open for 10 minutes
```

### `event_to_queue_action.yaml`

Turns on a selected helper when an event entity updates.

Useful for:

```text
doorbell event entity changed
```

### `android_last_notification_to_queue_action.yaml`

Filters the Android Companion App Last Notification sensor and turns on a selected helper when a notification matches.

Supported filter fields include placeholders for:

```text
package
category
channel
title / sender
participant destination
```

Create filters based on who sent a message or what communication platform sent the message.

## Example mappings

These examples are placeholders only.

```text
cover.example_garage_door → open → notification_queue_garage_opened
cover.example_garage_door → open for 10 minutes → notification_queue_garage_left_open
sensor.example_phone_state → ringing → notification_queue_phone_call
event.example_doorbell → updated → notification_queue_doorbell
sensor.example_last_notification → allowed messaging package → notification_queue_phone_message
```

## Privacy and safety notes

Lantern Protocol is designed to minimize what is stored.

The queue stores categories, not message content. The Android notification blueprint can inspect attributes such as app package, category, channel, title, or participant destination. Reminder, configure sensitive values locally in Home Assistant.

## Limitations

- If a smart plug cuts physical power to the bulb, the bulb may briefly boot using its own remembered color. The renderer waits for the bulb and then overwrites that color.
- This is an attention queue, not a message archive. It remembers notification types, not individual messages.
- Android apps expose notification attributes differently. Inspect the Last Notification entity in Home Assistant Developer Tools before tightening filters.
- Some bulbs take several seconds to rejoin the network after power is restored. Increase the renderer's wait/boot timing if color commands are missed.
- If the physical switch is the clear gesture, avoid other automations that turn that switch off unless you also want them to clear the queue.

## Why “Lantern Protocol”?

A lantern is a signal that does not require a screen. It can be seen without being handled. It can communicate without pulling you into an interface designed for extraction.

A protocol is a chosen rule for what is allowed through.

Lantern Protocol is a quiet-permission system for domestic technology: communication can enter, but attention is no longer surrendered by default.