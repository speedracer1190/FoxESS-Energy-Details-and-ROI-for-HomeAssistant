
# Home Assistant + FoxESS ROI & Energy Dashboard
FoxESS Energy Dashboard and ROI for HomeAssistant

A complete YAML-based setup for visualising FoxESS inverter data, daily savings, and real-time ROI (Return on Investment) tracking — all inside Home Assistant.

This project integrates the FoxESS Cloud custom component
 with Home Assistant sensors, utility meters, and templates to calculate daily savings, export income, and payback progress on your battery investment.

 Features

📊 Automatic ROI tracking – visualises daily, monthly, and cumulative savings against your system cost.
⚡ Dynamic tariff handling – calculates peak/off-peak rates for grid import and cost avoidance.
☀️ Export income calculation – tracks FiT income based on real exported kWh.
💸 Payback progress bar – shows how close you are to breakeven and flips to “profit” when achieved.
💡 Fully configurable – customise purchase cost, FiT rate, and tariffs via Helpers or YAML.
🧩 No cloud dependency – all computation happens locally in Home Assistant.

Requirements

Home Assistant 2024.6 or newer
FoxESS-HA custom integration (Fox Cloud with API)
 installed via HACS
Bar Card
Mushroom Cards
Vertical Stack Card

Installation
1️⃣ Clone or copy the configuration

Copy the contents of the /config/packages/foxess-roi.yaml (or similar) file into your Home Assistant configuration.
If you prefer GUI setup, follow the UI Helper Setup section below.

2️⃣ Set up the FoxESS Cloud integration

Add this to your configuration.yaml (or in packages/foxessCloud.yaml):

sensor:
  - platform: foxess
    deviceID: <your_device_id>     # datalogger ID or SN
    deviceSN: <your_device_sn>     # inverter serial
    apiKey: <your_api_key>


Restart Home Assistant and confirm you can see entities such as sensor.foxess_grid_consumption, sensor.foxess_feedin, and sensor.foxess_bat_soc.

3️⃣ Create Tariff Automation

This automation switches your utility meters between peak and off-peak periods automatically:

- alias: Tariff - set PEAK at 15:00
  trigger:
    - platform: time
      at: "15:00:00"
  action:
    - service: utility_meter.select_tariff
      target:
        entity_id:
          - utility_meter.grid_import_daily
          - utility_meter.load_daily
      data:
        tariff: peak

- alias: Tariff - set OFFPEAK at 21:00
  trigger:
    - platform: time
      at:
        - "21:00:00"
        - "00:00:15"
  action:
    - service: utility_meter.select_tariff
      target:
        entity_id:
          - utility_meter.grid_import_daily
          - utility_meter.load_daily
      data:
        tariff: offpeak

4️⃣ UI Helper Setup

In Settings → Devices & services → Helpers:

Input Numbers
Name	Entity ID	Default	Notes
Battery Purchase Price	input_number.battery_purchase_price	7500	cost of inverter + battery
ROI Opening Savings	input_number.roi_opening_savings	0	carry-over savings if any
Utility Meters
Name	Source	Cycle	Notes
Savings Monthly	sensor.savings_today	Monthly	Enable Delta values and Periodically resetting
Template Sensors

Create these via Helpers → Template → Template Sensor:

Name	Template
Savings This Month	`{{ states('sensor.savings_monthly')
Total Savings To Date	`{{ (states('input_number.roi_opening_savings')
ROI Remaining	`{{ (states('input_number.battery_purchase_price')
ROI Profit To Date	`{{ (states('sensor.total_savings_to_date')
5️⃣ Add Dashboard Cards
ROI Summary (Card 1)
type: custom:mushroom-template-card
primary: ROI summary
secondary: >
  Today:  ${{ states('sensor.savings_today') }}
  • MTD: ${{ states('sensor.savings_this_month') }}
  • Total: ${{ states('sensor.total_savings_to_date') }}
  • {{ 'Remaining' if states('sensor.roi_remaining')|float(0) > 0 else 'Profit' }}:
    ${{ (states('sensor.roi_remaining')|float(0) if states('sensor.roi_remaining')|float(0) > 0
        else states('sensor.roi_profit_to_date')|float(0)) | round(2) }}
icon: mdi:cash-multiple
layout: vertical
multiline_secondary: true
icon_color: >
  {% if states('sensor.roi_profit_to_date')|float(0) > 0 %} green
  {% elif states('sensor.roi_remaining')|float(0) < 1500 %} amber
  {% else %} red {% endif %}

Payback Progress (Card 2)
type: custom:bar-card
name: Payback progress
entity: sensor.total_savings_to_date
min: 0
max: 7500
target: 7500
decimal: 2
severity:
  - from: 0
    to: 1500
    color: red
  - from: 1500
    to: 5000
    color: orange
  - from: 5000
    to: 7500
    color: green

Remaining / Profit (Card 3)
type: custom:bar-card
name: Remaining / Profit
entity: sensor.roi_remaining
min: -2000
max: 7500
decimal: 2
icon: mdi:cash-minus

6️⃣ Optional Entity Summary
type: entities
title: ROI details
entities:
  - entity: input_number.battery_purchase_price
    name: Purchase Price
  - entity: sensor.total_savings_to_date
    name: Total Saved to Date
  - entity: sensor.roi_remaining
    name: Remaining to Payback (neg = profit)
  - entity: sensor.roi_profit_to_date
    name: Profit to Date

📈 Example Dashboard
ROI Summary
 ├── Payback progress bar (0 → $7 500)
 ├── Remaining / Profit indicator
 └── ROI Details entities table



(optional screenshot placeholder)

🧮 Formulas Used
Metric	Calculation
Grid Cost Today	(kWh × rate for each tariff)
Export Income Today	exported kWh × FiT rate
Savings Today	(would pay − did pay + export income)
ROI Profit To Date	(total savings − purchase price)
🧰 Customization

Change rates (0.4128, 0.2293, 0.01) in template sensors to match your provider.

Edit purchase price in Helper → Battery Purchase Price.

Add extra utility meters if you’d like yearly tracking.

🧡 Credits

macxq/foxess-ha - https://github.com/macxq/foxess-ha
 – FoxESS Cloud Integration

custom-cards/bar-card https://github.com/custom-cards/bar-card

piitaya/lovelace-mushroom https://github.com/piitaya/lovelace-mushroom

YAML layout and ROI logic by @speedracer1190
