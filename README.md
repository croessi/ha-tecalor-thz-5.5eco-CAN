# ha-tecalor-thz-5.5eco-CAN

ha-tecalor-thz-5.5eco-CAN is a ESPhome / Home Assistant configuration to monitor & configure tecalor thz 5.5 eco Heating Pumps via a CAN Interface.
It requires setting up an ESP32 Microcontroller with a MCP2515 CAN-Tranceiver and some configuring in Home Assistant.
It is based on the great work of the Home Assistant community, especially the work of https://github.com/bullitt186/ha-stiebel-control/ and [roberreiters](https://community.home-assistant.io/t/configured-my-esphome-with-mcp2515-can-bus-for-stiebel-eltron-heating-pump/366053) and [Jürg Müller](http://juerg5524.ch/list_data.php).

## Installation - NEEDS REWORK


### ESPHome
* Set up a new ESPHome Project
* setup the MCP2515 CAN interface according to your wiring/layout (https://esphome.io/components/canbus.html?highlight=mcp2515#mcp2515-component) 
* copy the folder `tecalor55eco`to your `/config/esphome` folder (full path should be `/config/esphome/tecalor55eco`)
* add the following packages by inserting the following lines into your ESPHOME project:
 ```yaml
  packages:
   manager: !include tecalor55eco/manager_sensors.yaml
   kessel: !include tecalor55eco/kessel_sensors.yaml
   heizmodul: !include tecalor55eco/heizmodul_sensors.yaml
   compute: !include tecalor55eco/compute_sensors.yaml
 ```
* add the following lines to trigger processing of incomming CAN messages (SET cs_pin according to your setup)
 ```yaml
  canbus:
    - platform: mcp2515
      id: my_mcp2515
      spi_id: McpSpi
      cs_pin: GPIO33
      can_id: 0x6a2
      use_extended_id: false
      bit_rate: 20kbps
      on_frame:

    - can_id: 0x180
      then: !include tecalor55eco/kessel_can.yaml
 ```


* You may want to check/change the CAN IDs of the Manager, Kessel, etc. In order to do so, you have to change them in two places:
  * `stiebeltools\heatingpump.h`:
    ```c
    static const CanMember CanMembers[] =
    {
    //  Name              CanId     ReadId          WriteId         ConfirmationID
      { "ESPCLIENT"     , 0x700,    {0x00, 0x00},   {0x00, 0x00},   {0xE2, 0x00}}, //The ESP Home Client, thus no valid read/write IDs
      { "KESSEL"        , 0x180,    {0x31, 0x00},   {0x30, 0x00},   {0x00, 0x00}},
      { "MANAGER"       , 0x480,    {0x91, 0x00},   {0x90, 0x00},   {0x00, 0x00}},
      { "HEIZMODUL"     , 0x500,    {0xA1, 0x00},   {0xA0, 0x00},   {0x00, 0x00}}
    };
    ```
  * `heatingpump.yaml`: Look for these blocks in the lower part of the file
    ```yaml
    #########################################
    #                                       #
    #   HEIZMODUL Nachrichten               #
    #                                       #
    #########################################
        - can_id: 0x500
          then:
            - lambda: |-
                unsigned short canId = 500;
    ```

### Home Assistant
#### Entities and Helpers
* place the file `packages/ha_stiebel_control.yaml` in your `/config/packages/` folder in Home Assistant.
* add the package folder to your `configuration.yaml` under `homeassistant` (if not already set up)
```yaml
homeassistant:
  packages: !include_dir_named packages
```
#### Dashboard
* Install the lovelace card [apexcharts-card](https://github.com/RomRider/apexcharts-card)
* Install the lovelace card [lovelace-mushroom](https://github.com/piitaya/lovelace-mushroom)
* Create a new Dashboard, switch to RAW mode and paste the content of `dashboard.yaml`. The result should look similar to this:
![Dashboard Screenshot](assets/img/dashboard.jpg "Dashboard Screenshot")

## Using
* FIXME

## Contributing

Pull requests are welcome. For major changes, please open an issue first
to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License

[GPLv3] (https://www.gnu.org/licenses/gpl-3.0.en.html)
