# ZMK Input Processor: Keybind

> ⚠️ **Development Notice**  
> This module is currently under active development and **somewhat works**.  
> Use at your own risk. API and behavior may change significantly.

A ZMK firmware module that quantizes XY pointer device inputs (like trackballs or touchpads) into discrete keypress events.

## Important Implementation Note (ZMK Version Behavior)

⚠️ **If using this processor on a non-default layer** (the most likely scenario), be aware that in the current main branch behavior of ZMK:

- **All subsequent processors** (including default ones) will still trigger
- This may cause **unwanted simultaneous actions**, such as:
  - Mouse movement (default trackball behavior) 
  - AND arrow key cursor movement (your bound keys)

### Workarounds:

1. **Use a modified ZMK fork** that fixes processor return code checking:
   - Ready-to-use fork: [zettaface/zmk](https://github.com/zettaface/zmk) (I try to keep it up-to-date with main ZMK branch)
   - Or manually apply the one-line change from my fork to your own. It's that simple
2. **Use `zip_xy_scaler` to nullify trackball movement**:
    Reference: [ZMK Issue #2967 Comment](https://github.com/zmkfirmware/zmk/issues/2967#issuecomment-2982313146)

## Installation

Include this module on your ZMK's west manifest in `config/west.yml`:

```yaml
manifest:
  remotes:
    # ...
    # START #####
    - name: zettaface
      url-base: https://github.com/zettaface
    # END #######
    # ...
  projects:
    # ...
    # Optional: Use this fork of ZMK in case of troubles with simultaneous actions
    # - name: zmk
    #   remote: zettaface
    #   revision: main
    #   import: app/west.yml
    # ...
    # START #####
    - name: zmk-input-processor-keybind
      remote: zettaface
      revision: main
    # END #######
    # ...
```

## Configuration

### Required Parameters
- **`bindings`** *(phandle-array)*  
  The key behaviors to trigger when input is detected. This is mandatory.

### Optional Parameters

- **`mode`** *(int, default: 0)*  
  How input translates to key events:
  - `0` = Raw direct movement
  - `1` = 4-directional (up/down/left/right) 
  - `2` = 8-directional (includes diagonals) 
    *Example: Diagonal movement (up+right) will press both up and right keys simultaneously*
- **`track_remainders`** *(boolean, default: false)*  
  When enabled, saves partial movement between activations
- **`continuous_key_press`** *(boolean, default: false)*  
  If true, the bound keys will be held down continuously while trackball movement is detected, rather than being pressed and released per tick.
- **`threshold`** *(int, default: 1)*  
  Minimum movement required to trigger (must be positive)
- **`max_threshold`** *(int, default: 200)*  
  Upper limit for threshold (caps sensitivity)
- **`tick`** *(int, default: 10)*  
  Movement units needed per activation (higher = less sensitive)
- **`wait-ms`** *(int, default: 0)*  
  Delay before next activation (milliseconds)
- **`tap-ms`** *(int, default: 20)*  
  Press-to-release timing (milliseconds)
- **`max_pending_activations`** *(int, default: 5)*  
  Maximum queued actions per axis

## Examples

Roughly, `overlay` of the split-peripheral trackball should look like below.

```

#include <input/processors/zmk-input-processor-keybind.dtsi>

&trackball_listener {
    status = "okay";
    device = <&trackball>;

    input-processors = <&zip_keybind_keys>;
};

```
### 4-Way Arrow Keys (Tap-style)

```

/ {
    zip_keybind_keys: zip_keybind_keys {
        compatible = "zmk,input-processor-keybind";
        #input-processor-cells = <0>;
        track_remainders;
        bindings = <&kp RIGHT>,
                  <&kp LEFT>,
                  <&kp DOWN>,
                  <&kp UP>;
        tick = <40>;
        wait-ms = <0>;
        tap-ms = <10>;
        // mode = <1>
        // threshold = <10>
        // max_threshold = <200>
        // max_pending_activations = <10>
    };
};

```

### 8-Way Arrow Keys (Continuous Press for Gaming)

```

/ {
    zip_keybind_arrows: zip_keybind_arrows {
        compatible = "zmk,input-processor-keybind";
        #input-processor-cells = <0>;
        bindings = <&kp RIGHT>,
                  <&kp LEFT>,
                  <&kp DOWN>,
                  <&kp UP>;
        tick = <10>;
        wait-ms = <0>;
        tap-ms = <50>;
        threshold = <2>;
        mode = <2>;
        continuous_key_press;
    };
};

```