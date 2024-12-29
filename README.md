<h1 style="text-align:left;">Somfy Situo 5 IO Pure II - ESPHome Integration</h1> [Guide currently incomplete]

<p style="text-align:left;">
  <img src="https://img.shields.io/github/license/SupaHotMoj0/somfy_situo5_io_pure_ii_esphome?style=flat-square" alt="License">
  <img src="https://img.shields.io/github/stars/SupaHotMoj0/somfy_situo5_io_pure_ii_esphome?style=flat-square" alt="Stars">
  <img src="https://img.shields.io/github/forks/SupaHotMoj0/somfy_situo5_io_pure_ii_esphome?style=flat-square" alt="Forks">
</p>

<hr>

<h2 style="text-align:left;">Table of Contents</h2>
<ol style="text-align:left;">
  <li><a href="#introduction">Introduction</a></li>
  <li><a href="#why-use-this-mod">Why Use This Mod?</a></li>
  <li><a href="#requirements">Requirements</a>
    <ul>
      <li><a href="#hardware">Hardware</a></li>
      <li><a href="#software">Software</a></li>
    </ul>
  </li>
  <li><a href="#hardware-setup">Hardware Setup</a>
    <ul>
      <li><a href="#disassembly--wiring">Disassembly &amp; Wiring</a></li>
      <li><a href="#power--voltage">Power &amp; Voltage</a></li>
    </ul>
  </li>
  <li><a href="#esphome-configuration">ESPHome Configuration</a>
    <ul>
      <li><a href="#key-features">Key Features</a></li>
      <li><a href="#installation-steps">Installation Steps</a></li>
      <li><a href="#multi-channel-example">Multi-Channel Example</a></li>
      <li><a href="#single-channel-example">Single-Channel Example</a></li>
    </ul>
  </li>
</ol>

<hr>

<h2 id="introduction" style="text-align:left;">Introduction</h2>
<p style="text-align:left;">
  Inspired by and derived from the code of:💐 @robvdw2 💐 <br>
  This repository shows how to mod a Somfy Situo 5 IO Pure II remote to be controlled by an ESP32.
  By mapping the remote’s buttons to GPIO pins and reading LEDs (to detect active channels), you can seamlessly operate
  multiple shutters via <strong>ESPHome</strong>—all within <strong>Home Assistant</strong>.
</p>

<hr>

<h2 id="why-use-this-mod" style="text-align:left;">Why Use This Mod?</h2>
<ul style="text-align:left;">
  <li><strong>Local Control</strong>: Zero dependency on vendor cloud services.</li>
  <li><strong>Cost-Effective</strong>: No expensive hubs.</li>
  <li><strong>Extendable</strong>: ESPHome supports additional sensors, automations, etc.</li>
</ul>

<hr>

<h2 id="requirements" style="text-align:left;">Requirements</h2>

<h3 id="hardware" style="text-align:left;">Hardware</h3>
<ul style="text-align:left;">
  <li><strong>Somfy Situo 5 IO Pure II Remote</strong><br>
      Prefer an older model with larger solder pads.</li>
  <li><strong>ESP32 Board</strong> (e.g., NodeMCU-32S)<br>
      Enough GPIO (and possibly ADC) pins for button/LED connections.</li>
  <li>Soldering iron, jumper wires, and basic tools.</li>
</ul>

<h3 id="software" style="text-align:left;">Software</h3>
<ul style="text-align:left;">
  <li><strong>ESPHome</strong> (Home Assistant add-on or standalone CLI)</li>
  <li><strong>Home Assistant</strong> (recommended)</li>
  <li><strong>Code Editor</strong> Like Sublime or Notepad (optional)</li>
</ul>

<hr>

<h2 id="hardware-setup" style="text-align:left;">Hardware Setup</h2>

<h3 id="disassembly--wiring" style="text-align:left;">Disassembly &amp; Wiring</h3>
<ol style="text-align:left;">
  <li><strong>Remove PCB</strong>: Carefully extract the remote’s PCB.</li>
  <li><strong>Remove Battery</strong>: The ESP32’s 3.3V line powers the remote.</li>
  <li><strong>Button Pads</strong> (Up, Stop/My, Down, Select):
    <ul>
      <li>The inner circle is ground, outer circle is the signal pin.</li>
      <li>Solder a wire from each outer circle to ESP32 GPIO (pull-down).</li>
    </ul>
  </li>
  <li><strong>Channel LEDs</strong>:
    <ul>
      <li>Solder wires to each LED pad; connect them to ADC or digital inputs on the ESP32.</li>
      <li>These indicate which channel is active.</li>
    </ul>
  </li>
</ol>

<h3 id="power--voltage" style="text-align:left;">Power &amp; Voltage</h3>
<ul style="text-align:left;">
  <li><strong>3.3V from the ESP32</strong> → remote’s battery terminals.</li>
  <li>Ensure the ESP32 regulator can handle the remote’s current draw.</li>
</ul>

<hr>

<h2 id="esphome-configuration" style="text-align:left;">ESPHome Configuration</h2>

<h3 id="key-features" style="text-align:left;">Key Features</h3>
<ul style="text-align:left;">
  <li><strong>Dynamic Channel Selection</strong>: Press “Select” until the correct channel LED is lit.</li>
  <li><strong>Real-Time Feedback</strong>: Detect active channel via LED voltage changes.</li>
  <li><strong>Comprehensive Setup</strong>: Wi-Fi, OTA, MQTT, and optional web server are supported.</li>
</ul>

<h3 id="installation-steps" style="text-align:left;">Installation Steps</h3>
<ol style="text-align:left;">
  <li><strong>Clone or Download</strong> this repository.</li>
  <li><strong>Configure</strong> the provided YAML (Wi-Fi, GPIO pins, etc.).</li>
  <li><strong>Flash</strong> using ESPHome (Home Assistant add-on or CLI).</li>
</ol>

<hr>

<h3 id="multi-channel-example" style="text-align:left;">Multi-Channel Example</h3>
<p style="text-align:left;">
  Below is a <strong>simplified</strong> YAML for controlling multiple channels. Adjust the pins, add channels as needed.
</p>

<pre style="text-align:left;">
<code># multi_channel.yaml
 
esphome:
  name: somfy_multi
  platform: ESP32
  board: nodemcu-32s

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_pwd

api:
  encryption:
    key: !secret api_key

ota:
  password: !secret ota_pwd

mqtt:
  broker: !secret mqtt_broker
  username: !secret mqtt_usr
  password: !secret mqtt_pwd

logger:

script:
  - id: channel_1
    then:
      - while:
          not:
            lambda: 'return id(selected_channel).state == "Channel 1";'
          then:
            - switch.turn_on: select_btn
            - delay: 250ms
      - text_sensor.template.publish:
          id: selected_channel
          state: "Channel 1"

  - id: channel_2
    then:
      - while:
          not:
            lambda: 'return id(selected_channel).state == "Channel 2";'
          then:
            - switch.turn_on: select_btn
            - delay: 250ms
      - text_sensor.template.publish:
          id: selected_channel
          state: "Channel 2"

switch:
  - platform: gpio
    id: select_btn
    pin: GPIO32
    inverted: true
    on_turn_on:
      - delay: 100ms
      - switch.turn_off: select_btn

  - platform: gpio
    id: up_btn
    pin: GPIO25
    inverted: true
    on_turn_on:
      - delay: 400ms
      - switch.turn_off: up_btn

  - platform: gpio
    id: stop_btn
    pin: GPIO27
    inverted: true
    on_turn_on:
      - delay: 400ms
      - switch.turn_off: stop_btn

  - platform: gpio
    id: down_btn
    pin: GPIO26
    inverted: true
    on_turn_on:
      - delay: 400ms
      - switch.turn_off: down_btn

text_sensor:
  - platform: template
    id: selected_channel
    name: "Selected Channel"
    update_interval: never

cover:
  - platform: template
    name: "Channel 1 Shutter"
    open_action:
      - script.execute: channel_1
      - script.wait: channel_1
      - switch.turn_on: up_btn
    stop_action:
      - script.execute: channel_1
      - script.wait: channel_1
      - switch.turn_on: stop_btn
    close_action:
      - script.execute: channel_1
      - script.wait: channel_1
      - switch.turn_on: down_btn
</code>
</pre>

<hr>

<h3 id="single-channel-example" style="text-align:left;">Single-Channel Example</h3>
<p style="text-align:left;">
  For just one shutter, skip the channel selection and wire Up/Stop/Down directly:
</p>
<pre style="text-align:left;">
<code># single_channel.yaml
 
esphome:
  name: somfy_single
  platform: ESP32
  board: nodemcu-32s

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_pwd

api:
  encryption:
    key: !secret api_key
ota:
  password: !secret ota_pwd
mqtt:
  broker: !secret mqtt_broker
  username: !secret mqtt_usr
  password: !secret mqtt_pwd

switch:
  - platform: gpio
    id: up_btn
    pin: GPIO25
    inverted: true
    on_turn_on:
      - delay: 400ms
      - switch.turn_off: up_btn

  - platform: gpio
    id: stop_btn
    pin: GPIO27
    inverted: true
    on_turn_on:
      - delay: 400ms
      - switch.turn_off: stop_btn

  - platform: gpio
    id: down_btn
    pin: GPIO26
    inverted: true
    on_turn_on:
      - delay: 400ms
      - switch.turn_off: down_btn

cover:
  - platform: template
    name: "Somfy Single Shutter"
    open_action:
      - switch.turn_on: up_btn
    stop_action:
      - switch.turn_on: stop_btn
    close_action:
      - switch.turn_on: down_btn
</code>
</pre>


<hr>

<h2 id="web-interface" style="text-align:left;">Web Interface</h2>
<p style="text-align:left;">
  If you add <code>web_server:</code> to your YAML, each shutter can be operated from your browser at
  <code>http://[ESP_IP]</code>. Commands can also be sent via URL, for example:
  <br><code>http://[ESP_IP]/2/down</code> to lower channel 2.
</p>

<hr>

<h2 id="home-assistant-integration" style="text-align:left;">Home Assistant Integration</h2>
<ol style="text-align:left;">
  <li><strong>Auto Discovery</strong>: If MQTT and or the ESPHome is enabled, shutters appear automatically.</li>
</ol>

<hr>

<h2 id="limitations" style="text-align:left;">Limitations</h2>
<ul style="text-align:left;">
  <li><strong>Soldering Challenges</strong>: The newer Pure II PCB pads can be small.</li>
  <li><strong>No Native IO Protocol</strong>: We rely on the remote’s internal logic</li>
  <li><strong>Continuous Power</strong>: The ESP32 must remain powered.</li>
</ul>
