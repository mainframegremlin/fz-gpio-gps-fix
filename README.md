- # GPS satellite fix, dev boards, and you
	- ## The problem
		- GPS units attached to boards or other hosts can create *painfully long* wait times for a first fix (also known as time to first fix or TTFF), in the matter of literal *days*, if at all. This is especially apparent if the module is new out-of-box and attempting to fix from a cold start. In write-ups and videos that contain usage of a GPS module (in this case we will be covering GY-NEO6MV2 and ATGM336H) attached to a host board, the TTFF is seemingly instant; I've found this to often *not* be the case personally, and the reason for this write-up. For those looking to wardrive with a FZ as a portable solution, you want to be as pick-up and go as possible. Fiddling with fix failures prevents us from doing so, and what this walkthrough intends to solve.
	- ## The solution
		- Thankfully for us, we can create an **isolated environment** to start our module in, and transfer it to one of our boards. This eliminates interference from electronics that causes the GPS module to fail to obtain fix. Through testing, this has been the most reliable method of obtaining a fix. From a cold start, both GPS modules tested would receive TTFF within 2 minutes, from indoors. From warm or hot start, this would usually be instant (a few seconds) once brought near a window. This allows us to get our module "online" and ready to use; eliminating fix failures, and [obtaining a reliable fix with 10 or more satellites](https://imgur.com/a/ZgTs9xx). This has been tested with very small ceramic antennas, to a car mountable SMA antenna; both giving the same result.
		- TL;DR put headers on your modules, use interchangeable cables, isolate GPS and power from standalone for 2 minutes, move module GND/VCC/TX/RX to host board, connect to FZ, run GPS associated tools.
	- ## Requirements for a fix
		- **This is not required knowledge for our solution, but is beneficial to know**, and can help troubleshoot if necessary. A reliable fix with GPS modules requires current "time" (precise, in accordance to what GPS obtains), GNSS receiver position, valid ephemeris data for >= 4 satellites. Ephemeris data refers to precise orbital data, and is used to calculate the positions and velocities of satellites. Ephemeris data is perishable; around 4 hours. This is what is referred to as a hot start; you can re-obtain your fix very easily after your initial fix, if you are within the 4 hour window, as the ephemeris data is still recent.
		- Our other set of data is usually referred to as a GPS almanac. This is a set of data that is broadcast by satellites that provide a *general overview* of the orbit posititions of satellites. Cold starts, paired with new modules (that have *no* almanac data), *as well as introducing electronic interference to our module*, creates a very steep hill for the module to climb, and does it no favors in getting a reliable satellite fix - **if at all in some cases**. Almanac data is primarily used to give the module a "head start", to obtain a fix. This is what gives us our warm start, as long as the module is within ~100km from previous fix.
			- ### A note on ublox's AssistNow
				- This is even further "not required reading" but worth noting here when discussing solutions. Ublox chips/ublox in general offers AssistNow data in the form of a .ubx file that is obtainable from their website after registering for an account/obtaining an API token. This data is similar to an almanac wherein it gives a ublox module (NEO6MV2 in our case) a "head start" in contextual satellite data. For my testing, I did not flash this data - at this point I won't, either, because the method we will be using is reliable and extremely quick. On top of that, u-center is proprietary software from ublox, and only available on Windows, which I don't use. I wanted to mention this "solution" in our guide, because (at least in my opinion) the time investment to flash the .ubx isn't worth it. We will get the same result without the hassle.
	- ## Preparation
		- We need a few things before we start, because we need to design our connectivity from a modular approach.
			- **Some form of jumper/breadboard wires**, you can also make your own. In this case we will be using female headers, since the headers will be soldered to our modules themselves.
			- **Pin headers**; these are what you see on GPIO boards or usually installed on raspberry pi's for the jumpers to connect to. We need these on our GPS module (a pin of 4) *and* our host board. You can either cut the headers, and do the 4 you need, or a whole strip of 8 (this is what we will be doing here). In short: **a strip of 4 header pins, and a strip of 8 header pins (or 8 headers x 2 strips if you want to cover all of the slots on the dev board)**.
			- Your **GPS module**.
			- Your **host module** (FZ dev board in this case).
			- An **external power supply** that allows connection for 3.3V-5V and GND. Doesn't need a LiPo attached to it, but it must be able to be powered by *something* e.g. USB, battery, etc.
				- [Example of a LiPo UPS/jumper for our purposes](https://imgur.com/a/IU3owOH)
				- You can also look for "USB powered 3.3V-5V module"
				- Doesn't matter what it is as long as: it gets power from somewhere, it's regulated at no more than your GPS module; for us that's 5V
			- An **area you have access to that is free from electronics or signal emitting devices**. I simply used windowsills in a kitchen for testing.
	- ## Implementing the fix, and getting our satellite fix
		- [Solder the headers to your GPS module](https://imgur.com/a/erNpzdi). We only need GND, VCC, TX, and RX covered for GPS.
		- [Solder the headers to your host/dev board](https://imgur.com/a/ni6Vvfs). I did all of the middle sections of the dev board rather than cutting an individual connector for each slot.
		- Make sure your antenna is attached to your module. A small ceramic will do, your module probably came with one.
		- Make sure your host/dev board is not attached to your FZ. We will want to connect this once we have our GPS connected to it.
		- [Connect GPS module to your external power supply](https://imgur.com/a/SQfrwWq), use tape if necessary. You can also attach some headers onto this as well, depending on your situation. We only care about VCC and GND since we're not doing anything with TX or RX data yet.
		- Wait a couple of minutes; if you're on cold start/new module, I'd give it 20 minutes unless you have a fix indication such as an LED.
			- On the ATGM336H this is a blinking red light; solid red indicates power
		- Disconnect your module from your external power source, and hook it up to your host/dev board as follows.
			- GPS GND -> GND
			- GPS VCC -> 3V3
			- GPS RX -> IO21 (or TX if you are for some reason using the lower part of the board, and not the middle)
			- GPS TX -> IO9 (or RX if you are for some reason using the lower part of the board, and not the middle)
		- Navigate to Apps -> GPIO -> ESP -> Marauder -> GPS Data (stream).
			- This should start spitting out our NMEA sentence; you may need to walk near a window
		- You should now be able to select Wardrive (ap) and get the following:

			  ```
			  39 | your:moms:mac:address:here:7B,your-moms-ap,[hopefully WPA2_PSK],2025-1-1 1:53:20,11,some,lat,lon,alt,data,WIFI
			  ```
		- ???
		- [Profit](https://imgur.com/a/Z3ywG9m)!
	- ## Wrapping up
		- While our solution takes some prior preparation, it is far more worth adjusting our approach to be more modular/hot swappable, especially if our intention is to wardrive within a small time window (get up and go). This will reliably eliminate the lack of satellite fix, and get us connected to 10-14 satellites quickly; at the very least hitting the minimum requirement of 4 satellites. This is a crucial method for a recon process, where tools should be as immediately available as possible.
  		- Keep in mind, attaching by crimped cables will increase the clearance size, but is still pocketable. You may have to get creative with wiring management, printing a case, etc. However, this is a small price to pay for reliability. Even with the added clearance from the cables, this will still fit into a jacket or bag comfortably. I recommend wrapping your soldered joints in some electrical tape if you aren't using a case, because they will get hot.
    		- It's also worth reiterating, unless you have a very specific use-case, a small ceramic antenna will do the trick just fine. The times given above were using the stock ATGM336H antenna.
