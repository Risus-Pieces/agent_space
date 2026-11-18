# Syringe Pump Motor Setup: Handoff Notes

Context for a new chat. I'm a beginner and don't know the part names well, so please explain simply.

## My hardware (identified from photos)
- **Arduino Mega 2560 R3** (blue board, the "brain"). Connected to the computer by a blue USB cable.
- **CNC Shield V3** (red board, protoneer "VER 3.00"), plugged on top of the Mega.
- **3 × A4988 stepper drivers** (small green chips with heatsinks) in the X, Y and Z slots. The A slot is empty.
- **Stepper motor 17HS19-2004S1** (NEMA 17, 2 A rated, 1.8°/step = 200 steps per turn). It's labeled "2" and drives a lead-screw syringe pump (orange frame, pink pusher).
- **Power adapter:** 12 V DC, 1500 mA (model JF48-1201500DU). Its cable has bare wires, no round plug. The wires are screwed into the blue 2-screw block on the red board. One wire has a white stripe, and the tag on the cable says "White Stripe" and shows a **+**.

## Board layout (red board, viewed with the blue screw block at bottom-left)
| Chip position | Axis | Motor pins |
|---|---|---|
| Top-left | **X** | Black row of 4 pins just to its right (small "X" printed nearby). **Use this one.** |
| Top-right | Y | Black row of 4 pins to its right |
| Bottom-left | Z | Black row of 4 pins to its right |
| Middle-right (empty red sockets) | A | Unused |

- **Motor plugs go only on the single black row of 4 pins next to a chip.** Do NOT use the yellow/blue pins on the left edge, the white/black double row on the right edge (END STOPS), or the pins under the chips.
- The Y chip has **3 red jumper caps** under it, which set 1/16 microstepping. That means 200 steps only turns the motor 1/16 of a turn, so a motor there would barely move in the test. It's unknown whether X has jumpers.
- Motor plug wire order is red, blue, green, black. Pairs stay together, so the plug can go on either way round. It must cover all 4 pins. Never shift it over by one pin.

## Pin mapping (CNC Shield V3 on the Mega)
- X: STEP = D2, DIR = D5
- Y: STEP = D3, DIR = D6
- Z: STEP = D4, DIR = D7
- Enable (all drivers) = D8, set LOW to turn the motors on

## Safety rules
1. **Never plug or unplug the motor while the 12 V adapter is plugged into the wall.**
2. **The power polarity must be right.** "12-36V" printed near the screw block is only the accepted voltage range, NOT a +/− marking. The white-stripe wire is very likely +, but **confirm which screw is + with a multimeter or a lab member before powering on.**
3. The 1.5 A adapter is fine for one motor.
4. Optional: the current limit is set with the tiny screw (potentiometer) on the green chip. Target Vref ≈ 0.6–0.8 V, measured between that screw and GND. Leave it alone if the lab already set it.
5. Watch that the syringe pusher doesn't hit the end of the track, because the code has no limit switches. Unplug the power if it's about to.

## Startup order
1. Adapter unplugged from the wall.
2. Motor plug on the black pins next to **X** (top-left chip).
3. USB to computer. In the Arduino IDE choose **Tools → Board → Arduino Mega 2560** and the right **Port**, then upload the code below.
4. Confirm the power wires (+ to +).
5. Plug the adapter into the wall. The motor should turn once, pause 1 second, and repeat.

## Test code (drives X, Y and Z at once, so any slot works)
```cpp
void setup() {
  for (int p = 2; p <= 8; p++) pinMode(p, OUTPUT);
  digitalWrite(8, LOW);            // turn motors on
  for (int p = 5; p <= 7; p++) digitalWrite(p, HIGH);  // direction
}

void loop() {
  for (int i = 0; i < 200; i++) {  // 200 steps = 1 turn at full step
    for (int p = 2; p <= 4; p++) digitalWrite(p, HIGH);
    delayMicroseconds(2000);
    for (int p = 2; p <= 4; p++) digitalWrite(p, LOW);
    delayMicroseconds(2000);
  }
  delay(1000);
}
```
- To reverse direction, change `HIGH` to `LOW` in the direction line.
- If the motor barely moves, the slot probably has the microstep jumpers (1/16). Change `200` to `3200` and `2000` to `300`.

## Troubleshooting
- **Nothing happens:** check that the 12 V adapter is plugged in, the screws are gripping bare copper, and the code uploaded to the right board and port.
- **Buzzes or shakes but doesn't turn:** power off, then flip the motor plug around.
- **Wrong direction:** swap HIGH/LOW in the code.
- **Chip very hot or motor stops randomly:** the current limit is too high. Turn the tiny screw slightly counter-clockwise, with the power off.

## Next steps (not done yet)
- Turn the test code into a real syringe dispenser (e.g. "dispense 2 mL at 1 mL/min"). This needs:
  - the lead-screw pitch (mm the pusher moves per motor turn)
  - the syringe's inner diameter or size (e.g. 10 mL BD)
  - the microstep setting of the slot used
