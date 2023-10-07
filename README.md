# WirelessBeaconCreator-ESP
Crafting 802.11 Beacon Frames from Scratch on ESP8266

The ESP8266 WiFi chipset provides an avenue for transmitting tailored WiFi frames without navigating the networking stack. In today’s post, we’ll leverage this feature to manually assemble and broadcast beacon frames, effectively impersonating WiFi access points.

Dissecting the Beacon Frame

Before we dive in, it’s essential to grasp the architecture of the 802.11 beacon frame. Key constituents include:

    Frame control: Specifies the frame kind and assorted flags
    Duration: Pinpoints the time span for transmission
    Destination MAC: Typically a broadcast for beacons
    Source MAC: Represents the MAC of the transmitting interface
    BSSID: Essentially the MAC of the AP or IBSS
    Sequence number: Undergoes an increment with each frame dispatch
    Timestamp: Captures the microsecond timer reading
    Beacon interval: Determines how often beacons are sent
    Capability info: Outlines the 802.11 modes on offer
    SSID: Ranges from 1-32 bytes and indicates the WiFi network’s moniker
    Additional parameters: Encompass vendor-centric fields

Buffering the Frame

The task at hand is to produce a buffer that encapsulates this binary structure. Here’s how to proceed:

    Establish static arrays for unchanging fields such as frame control and BSSID.
    Retain a flexible array specifically for the SSID.
    Kickstart an array in light of the total frame size you anticipate.
    Transfer static array data into the frame buffer.
    Infuse the SSID byte values into its designated slot.
    Tag on the remaining static fields post the SSID.

Given below is the code that embodies this methodology:

// Predetermined frame header 
uint8_t packet_start[10] = { 0x80, 0x00, 0x00, 0x00, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff}; 

// Mid-segment fields such as BSSID
uint8_t packet_middle[14] = { 0xc0, 0x6c, 0x83, 0x51, 0xf7, 0x8f, 0x0f, 0x00, 0x00, 0x00, 0x64, 0x00, 0x01, 0x04};   

// Placeholder carved out for SSID
uint8_t packet_ssid[32] = { 0x00, 0x07, 0x72, 0x72, 0x72, 0x72, 0x72, 0x72, 0x72};

// Residual constant bytes  
uint8_t packet_end[12] = { 0x01, 0x08, 0x82, 0x84, 0x8b, 0x96, 0x24, 0x30, 0x48, 0x6c, 0x03, 0x01}; 

// Frame's final formation
uint8_t packet[MAX_SIZE];

// Migrating header and middle portions 
memcpy(packet, packet_start, 10);  
memcpy(packet+22, packet_middle, 14);

// Inclusion of the actual SSID
packet[36] = ssid_len;
memcpy(packet+37, ssid, ssid_len); 

// Migrating the ending components
memcpy(packet+62, packet_end, 12);

This seamlessly crafts the whole beacon frame, content by content. Your only task is to pinpoint the apt SSID value prior to broadcasting.

Frame Transmission

The API wifi_send_pkt_freedom() provided by ESP8266 facilitates the broadcast of the unadulterated WiFi frame buffer.

Illustratively:

// Broadcast of 3 beacon frames
for(int i=0; i<3; i++) {
  wifi_send_pkt_freedom(packet, sizeof(packet), false);
}

Voila! The WiFi apparatus will handle the encoding of frame components, ensuring its airborne transmission.

Final Thoughts

Constructing beacon frames manually grants you the privilege of mastering the content of the packet. It paves the way for impersonating random SSIDs and BSSIDs, useful in penetration tests among other applications. As always, uphold ethical standards when broadcasting such frames in operational networks!

Noticed the ellipses in the packet sketches? Curious if you can plug in any random data?

Excellent catch! These ellipses within packet arrays signify segments where you can introduce arbitrary or chance data, as seen in:

uint8_t packet_start[10] = { 0x80, 0x00, ..., ..., ...};

Fields open to randomization encompass:

    Sequence number (bytes 2-3)
    Fragment number (byte 4)
    Source and destination MAC addresses (bytes 5-10)

In a similar vein, within packet_middle:

uint8_t packet_middle[14] = {..., ..., ..., 0x00, 0x00};

The Timestamp and Beacon Interval segments (bytes 0-7) can be randomized.

Thus, in essence:

    Rely on credible constant values for segments like frame control and capability info, taking cues from the 802.11 norms.
    However, fields such as the sequence number, timestamps, and MAC addresses can deviate from the actual interface, embracing random values.
    This ensures that each beacon frame broadcasted remains distinctive, dodging identical frame detection mechanisms.
    Such an approach also complicates any efforts to trace and correlate these frames.

The ellipses, therefore, act as markers indicating zones where random data can be infused, enhancing the anonymity of the spoofed beacon frames.
