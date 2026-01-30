# Two OpenEVSE One Circuit
### How to have load sharing between two OpenEVSE units on the same circuit breaker.

Current firmware for the OpenEVSE devices has a functionality called 'Shaper'.  This is intended to permit load sharing between the OpenEVSE and other loads without tripping the breaker -- commonly used in homes with a 100 amp panel (or similar) -- permitting the EVSE connected vehicle to charge with the remaining power not used by the loads in the rest of the home.

**Requirements:**
- A CT clamp providing a total home power consumption value via MQTT.
	- The [Shelly EM](https://us.shelly.com/products/shelly-em-120a-clamp?variant=49666055897429) works great for this!
    - However, any CT set capable of publishing the total power used for a two phase circuit can be used.
- An MQTT Broker
- An OpenEVSE on current firmware.

**Operation**
With this Shaper capability, we can also set up two OpenEVSE devices to share a single circuit breaker. 
- One OpenEVSE will be primary and able to pull up to the Max current, minus 6 amps. 
- The other OpenEVSE will be the secondary - always able to charge at the minimum 6 amps, but then able to consume whatever the primary OpenEVSE is not.

**The Setup:**
1. Install the two OpenEVSE devices on the same circuit and circuit breaker
2. Install the ShellyEM (or similar), using a CT clamp large enough to fit both L1 and L2 wires through, and rated for double the current rating of the breaker.
    * For a ShellyEM on a 50 amp breaker, use the 120 amp CT clamp.
	* This is to allow for summing of the current, since the ShellyEM can only receive power from one leg.
	* To accomplish this, pass both L1 and L2 through the CT clamp, but make sure they pass through in opposite directions.
	* e.g. In the photo below, the RED (L1) Breaker side enters the top of the CT, the BLACK (L2) Breaker side should enter the bottom of the CT.
	   ![Image showing a single CT clamp around two lines for measuring split phase current flow.](Two%20Phase%20CT%20Monitoring.jpg)
3. In the ShellyEM, Enable MQTT and configure the MQTT Broker under:
   *Internet & Security* -> *Advanced - Developer Settings*
4. Using MQTT Explorer, or other means, connect to the MQTT Broker and observe the MQTT topic to which the power is posted for the ShellyEM Line where the CT coil is connected.
   Example: `shellies/shellyem-B164B8/emeter/0/power`
5. On the Primary OpenEVSE, set the charge current limit based on the following formula:
   `(Breaker Size) * 0.8 - 6`
   Example for a 50 amp breaker:
   ```
   50 * 0.8 = 40 amps
   40 - 6 = 32 amps limit for Primary OpenEVSE
   ```
   This is set at:
   *Settings* -> *EVSE* -> *MAX CURRENT*
6. On the Secondary OpenEVSE:
	* Set the *MAX CURRENT* value to the same as that calculated and set on the Primary OpenEVSE
	* Enable MQTT and configure the MQTT Broker under:
      *Settings* -> *MQTT*
	* Configure the Shaper under:
      *Settings* -> *Shaper*
		* Set `Max Power Allowed` per the following formula:
	      `(Breaker Size) * 0.8 * (Circuit Voltage)`
		  Example for a 50 amp breaker and 240 volt (nominal) US Circuit:
		  `50 * 0.8 * 240 = 9600 Watts`
		* Set the `Live power load MQTT Topic (w)` to the ShellyEM's Power MQTT topic.
	      Example from above: `shellies/shellyem-B164B8/emeter/0/power`

