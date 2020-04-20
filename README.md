**Project state: SUSPENDED**

Check these repos:
 - https://github.com/andersonshatch/midea-ac-py
 - https://github.com/NeoAcheron/midea-ac-py

As I know @andersonshatch has the latest version and @NeoAcheron does not have
access to Midea devices anymore.

---

## How to use

The API key is `3742e9e5842d4ad59c2db887e12449f9` if you extract
it from their `.apk` package. But I'm not sure.

### Installation

    gem install midea-air-condition

### CLI

#### Configure

Before you can use the client you have to configure it. It will ask for email, password and app key. The configuration will be saved in `~/.midea/config`

    midea-ac configure

#### List devices

You can list all the devices connected to the account

    midea-ac list

#### Get device

You can get the status of a device

    midea-ac get DEVICE_ID

#### Set device

You can Set the power temperature and fan speed of a device

    midea-ac set DEVICE_ID --power=[on|off] --target-temperature=TARGET_TEMPERATURE --fan-speed FAN_SPEED


### Example:

```
require_relative 'lib/midea_air_condition'

client = MideaAirCondition::Client.new(
  'username/email',
  'passsssssword',
  app_key: 'api_key'
)

client.debug = true
client.login
devices = client.appliance_list

target = 0

devices.each do |device|
  print "[id=#{device['id']} type=#{device['type']}]"
  print " #{device['name']} is "
  print 'not ' if device['onlineStatus'] != '1'
  print 'online and '
  print 'not ' if device['activeStatus'] != '1'
  puts 'active.'

  target = device['id'] if device['onlineStatus'] == '1'
end

# Status
builder = client.new_packet_builder
command = MideaAirCondition::Command::RequestStatus.new
builder.add_command(command)

response = client.appliance_transparent_send(
  target,
  builder.finalize
)

device = MideaAirCondition::Device.new(response)

puts "#{target} is turned #{(device.power_status ? 'on' : 'off')}."
if device.power_status
  puts 'Details:'
  puts "  Target temperature is #{device.temperature} celsius."
  puts "  // Indoor temperature: #{device.indoor_temperature} celsius."
  puts "  // Outdoor temperature: #{device.outdoor_temperature} celsius."
  puts "  Mode: #{device.mode_human}."
  puts "  Fan speed: #{device.fan_speed}."
  puts "  TimerOn is #{(device.on_timer[:status] ? '' : 'not')} active."
  puts "    at:  #{device.on_timer_human}" if device.on_timer[:status]
  puts "  TimerOff is #{(device.off_timer[:status] ? '' : 'not')} active."
  puts "    at: #{device.off_timer_human}" if device.off_timer[:status]
  puts "  Eco mode is #{(device.eco ? 'on' : 'off')}."
end

# Turn on
builder = client.new_packet_builder
command = MideaAirCondition::Command::Set.new
command.turn_on
command.temperature 25
builder.add_command(command)

response = client.appliance_transparent_send(
  target,
  builder.finalize
)

device = MideaAirCondition::Device.new(response)
puts "  Target temperature is #{device.temperature} celsius."

# Turn off
builder = client.new_packet_builder
command = MideaAirCondition::Command::Set.new
command.turn_off
builder.add_command(command)

response = client.appliance_transparent_send(
  target,
  builder.finalize
)
device = MideaAirCondition::Device.new(response)
puts "  Target temperature is #{device.temperature} celsius."

# Set temperature to 23 celsius
builder = client.new_packet_builder
command = MideaAirCondition::Command::Set.new
command.temperature 23
builder.add_command(command)

response = client.appliance_transparent_send(
  target,
  builder.finalize
)

device = MideaAirCondition::Device.new(response)
puts "  Target temperature is #{device.temperature} celsius."
```

## CRC Table + base64

I tried to generate the crc8 table, but I can't.
The table itself is a very long one,
so I packed base64 encoded it.

At least this one is not working:

```
crc8_table = []
256.times do |i|
  crc = i
  8.times do
    crc = (crc << 1) ^ (crc & 0x80 != 0 ? 0x07 : 0)
  end
  crc8_table[i] = crc & 0xff
end
```

The final table should look like this:

```
crc8_854_table = [
    0x00, 0x5E, 0xBC, 0xE2, 0x61, 0x3F, 0xDD, 0x83,
    0xC2, 0x9C, 0x7E, 0x20, 0xA3, 0xFD, 0x1F, 0x41,
    0x9D, 0xC3, 0x21, 0x7F, 0xFC, 0xA2, 0x40, 0x1E,
    0x5F, 0x01, 0xE3, 0xBD, 0x3E, 0x60, 0x82, 0xDC,
    0x23, 0x7D, 0x9F, 0xC1, 0x42, 0x1C, 0xFE, 0xA0,
    0xE1, 0xBF, 0x5D, 0x03, 0x80, 0xDE, 0x3C, 0x62,
    0xBE, 0xE0, 0x02, 0x5C, 0xDF, 0x81, 0x63, 0x3D,
    0x7C, 0x22, 0xC0, 0x9E, 0x1D, 0x43, 0xA1, 0xFF,
    0x46, 0x18, 0xFA, 0xA4, 0x27, 0x79, 0x9B, 0xC5,
    0x84, 0xDA, 0x38, 0x66, 0xE5, 0xBB, 0x59, 0x07,
    0xDB, 0x85, 0x67, 0x39, 0xBA, 0xE4, 0x06, 0x58,
    0x19, 0x47, 0xA5, 0xFB, 0x78, 0x26, 0xC4, 0x9A,
    0x65, 0x3B, 0xD9, 0x87, 0x04, 0x5A, 0xB8, 0xE6,
    0xA7, 0xF9, 0x1B, 0x45, 0xC6, 0x98, 0x7A, 0x24,
    0xF8, 0xA6, 0x44, 0x1A, 0x99, 0xC7, 0x25, 0x7B,
    0x3A, 0x64, 0x86, 0xD8, 0x5B, 0x05, 0xE7, 0xB9,
    0x8C, 0xD2, 0x30, 0x6E, 0xED, 0xB3, 0x51, 0x0F,
    0x4E, 0x10, 0xF2, 0xAC, 0x2F, 0x71, 0x93, 0xCD,
    0x11, 0x4F, 0xAD, 0xF3, 0x70, 0x2E, 0xCC, 0x92,
    0xD3, 0x8D, 0x6F, 0x31, 0xB2, 0xEC, 0x0E, 0x50,
    0xAF, 0xF1, 0x13, 0x4D, 0xCE, 0x90, 0x72, 0x2C,
    0x6D, 0x33, 0xD1, 0x8F, 0x0C, 0x52, 0xB0, 0xEE,
    0x32, 0x6C, 0x8E, 0xD0, 0x53, 0x0D, 0xEF, 0xB1,
    0xF0, 0xAE, 0x4C, 0x12, 0x91, 0xCF, 0x2D, 0x73,
    0xCA, 0x94, 0x76, 0x28, 0xAB, 0xF5, 0x17, 0x49,
    0x08, 0x56, 0xB4, 0xEA, 0x69, 0x37, 0xD5, 0x8B,
    0x57, 0x09, 0xEB, 0xB5, 0x36, 0x68, 0x8A, 0xD4,
    0x95, 0xCB, 0x29, 0x77, 0xF4, 0xAA, 0x48, 0x16,
    0xE9, 0xB7, 0x55, 0x0B, 0x88, 0xD6, 0x34, 0x6A,
    0x2B, 0x75, 0x97, 0xC9, 0x4A, 0x14, 0xF6, 0xA8,
    0x74, 0x2A, 0xC8, 0x96, 0x15, 0x4B, 0xA9, 0xF7,
    0xB6, 0xE8, 0x0A, 0x54, 0xD7, 0x89, 0x6B, 0x35
]
```
