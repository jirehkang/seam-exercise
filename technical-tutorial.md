# Controlling Disco Lights

Seam makes it easy to manage hardware with code. This guide shows you how to use the Seam API to control your disco lights. You'll learn how to list available lights, select one, and adjust its color, brightness, and mode to match your event's energy.

## Prerequisites

Before diving in, make sure you've completed the following setup.

- Ensure your disco lights are online and [connected](https://docs.seam.co/guides/setup-disco-lights-in-dashboard) to your Seam account via the [Seam Dashboard](https://console.seam.co/)
- Install the [Seam Python SDK](https://pypi.org/project/seam/) 
- [Authenticate](https://docs.seam.co/latest/core-concepts/authentication) with the Seam API using your API key

## 1. List All Disco Lights

Start by retrieving all disco lights connected to your account using [`seam.disco_lights.list()`](https://docs.seam.co/api/disco-lights/list-disco-lights).

```python
import seam

all_lights = seam.disco_lights.list()
```

This method returns an array of [`DiscoLight`](https://docs.seam.co/api/DiscoLight) objects under the `disco_lights` key. Here's an example:

```json
{
  "disco_lights": [
    {
      "device_id": "disco_123",
      "name": "Dance Floor Spot A",
      "online": true,
      "supports_color": true,
      "supports_brightness": true,
      "supports_modes": ["solid", "strobe", "pulse", "rainbow"],
      "state": "off",
      "brightness": 80,
      "color": "#FF00FF",
      "mode": "solid"
    }
  ]
}
```

## 2. Get a Specific Disco Light

Once you have a list of your lights, use [`seam.disco_lights.get()`](https://docs.seam.co/api/disco-lights/get-disco-light) to fetch the details of a single disco light by its `device_id`.

This example selects the first light that's online.

```python
for light in all_lights["disco_lights"]:
    if light["online"]:
        target_light = seam.disco_lights.get({ "device_id": light["device_id"] })["disco_light"]
        break
```

## 3. Modify Disco Light Settings

Now that you've picked your light, let's make it do something: turn it on, make it brighter, switch modes, or change the color.

> Note: Not all devices support brightness adjustments, mode switches, or color changes. Check the light's `supports_color`, `supports_brightness`, and `supports_mode` properties to confirm its capabilities. 

### Turn On or Off

With the `device_id` of the light you selected, use [`seam.disco_lights.on()`](https://docs.seam.co/api/disco-lights/turn-on) or [`seam.disco_lights.off()`](https://docs.seam.co/api/disco-lights/turn-off) to toggle the light on or off.

```python
# Turn light on
seam.disco_lights.on({ "device_id": target_light["device_id"] })

# Turn light off
seam.disco_lights.off({ "device_id": target_light["device_id"] })
```

### Set Brightness

Control the light's brightness using [`seam.disco_lights.set_brightness()`](https://docs.seam.co/api/disco-lights/set-brightness). Set a value between 0 and 100 (unit: %).

```python
seam.disco_lights.set_brightness({
    "device_id": target_light["device_id"],
    "value": 60
})
```

### Set Light Mode

Change how the light behaves using [`seam.disco_lights.set_mode()`](https://docs.seam.co/api/disco-lights/set-mode). Available modes include `solid`, `strobe`, `pulse`, and `rainbow`. 

```python
seam.disco_lights.set_mode({
    "device_id": target_light["device_id"],
    "mode": "solid"  
})
```

### Change Color

Adjust the color of the disco light using [`seam.disco_lights.set_color()`](https://docs.seam.co/api/disco-lights/set-color). Specify the color value in hex code format.

> Note: Color changes are only supported in `solid` mode. 

``` python
seam.disco_lights.set_color({
    "device_id": target_light["device_id"],
    "hex": "#FF00FF"
})
```

## Test Disco Light Settings

After making changes to your disco light, it's good practice to confirm that they were applied successfully. You can do this by retrieving the updated light state and checking the key values.

The example below assumes you've just turned the light on and set its brightness to 60.

```python
# Step 1: Retrieve updated light state
updated = seam.disco_lights.get({ "device_id": target_light["device_id"] })["disco_light"]

# Step 2: Assert expected values
assert updated["state"] == "on", "Light should be turned on"
assert updated["brightness"] == 60, "Brightness should be set to 60"
```

In addition to code-based assertions, you can also visually confirm that your light changes are reflected in the physical venue or through the [Disco Lights Dashboard](https://seam-disco-lights-dashboard.vercel.app/).

## Congratulations!

You've just built a working integration with the Seam Disco Lights API! 🎉 

From discovering devices to adjusting colors, brightness, and modes, you've laid the groundwork for controlling the vibe of any venue with code.

## Next Steps

With the basics in place, try customizing your setup for different event scenarios, device names, or venue zones.

For instance, you can apply different behaviors based on the name of the light. You might set all stage lights to a bold color while dimming the rest.

```python
all_lights = seam.disco_lights.list()["disco_lights"]

for light in all_lights:
    device_id = light["device_id"]

    if "stage" in light["name"].lower():  # case-insensitive match
        seam.disco_lights.on({ "device_id": device_id })
        seam.disco_lights.set_mode({ "device_id": device_id, "mode": "solid" })
        seam.disco_lights.set_color({ "device_id": device_id, "hex": "#FF00FF" })  # magenta
    else:
        if light["supports_brightness"]:
            seam.disco_lights.set_brightness({ "device_id": device_id, "value": 25 })
```

For full details on the methods you've used, see the [Disco Lights API Reference](https://docs.seam.co/api/disco-lights).

## Common Errors 

As you continue building, here are some common errors to watch out for and how to handle them.

| Error Code	|  How to Handle | 
| ------------- | ---------------- |
| `device_offline` | Retry once the disco light is back online. |  
| `unsupported` | Check the disco light's supported capabilities before changing mode, brightness, or color. |   
| `out_of_range` | Use a brightness value between `0` and `100`. |  
| `invalid_color` | Use a valid hex format like `#FF0000`. |  
| `mode_mismatch` | Switch to `solid` mode before changing the color. |  
