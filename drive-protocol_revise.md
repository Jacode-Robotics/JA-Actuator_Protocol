## [1.Introduction]()

To control JA-Actuator, communication should be established according to the protocol of JA-Actuator.

### [1.1.packet](#packet)

Main Controller and JA-Actuator communicate each other by sending and receiving data called Packet. Packet has two kinds: Instruction Packet, which Main Controller sends to control JA-Actuator, and Status Packet, which JA-Actuator responses to Main Controller.

### [1.2.ID]()

ID is a specific number for distinction of each JA-Actuator when several JA-Actuator’s are linked to one bus. By giving IDs to Instruction and Status Packets, Main Controller can control only JA-Actuator that you want to control

### [1.3.JA-Actuator Protocol]()

If JA-Actuator with the same ID is connected, packet will collide and network problem will occur. Thus, set ID as such that there is no JA-Actuator with the same ID.

**NOTE** : JA-Actuator’s initial ID is 1 at the factory condition.

### [1.4.Instruction Packet]()

Instruction Packet is the command packet sent to the Device.

| Header 1 | Header 2 | Header 3 | Reserved | Packet ID | Length 1 | Length 2 | Instruction | Param   | Param | Param   | CRC 1 | CRC2  |
| -------- | -------- | -------- | -------- | --------- | -------- | -------- | ----------- | ------- | ----- | ------- | ----- | ----- |
| 0xFF     | 0xFF     | 0xFD     | 0x00     | ID        | Len_L    | Len_H    | Instruction | Param 1 | ...   | Param N | CRC_L | CRC_H |

#### [1.4.1.Header]()

The field that indicates the start of the Packet

#### [1.4.2.Reserved]()

Uses 0X00 (Note that Reserved does not use 0XFD). The Reserved functions the same as Header.

| Header      | ID | Length | Inst | Param    | CRC   |
| ----------- | -- | ------ | ---- | -------- | ----- |
| FF FF FD 00 | 01 | 06 00  | 03   | 40 00 01 | DB 66 |

#### [1.4.3.Packet ID]()

The field that indicates an ID of the device that should receive the Instruction Packet and process it

1. Range : 0 ~ 252 (0x00 ~ 0xFC), which is a total of 253 numbers that can be used
2. Broadcast ID : 254 (0xFE), which makes all connected devices execute the Instruction Packet

#### [1.4.4.Length]()

The field that indicates the length of packet field.

1. Devided into low and high bytes in the  Instruction
2. The Length indicates the Byte size of Instruction, Parameters and CRC fields

- `Length = the number of Parameters + 3`
- Status Packet includes 1 byte length ERROR field’s data.

#### [1.4.5.Instruction](#instruction)

| Value |           Instructions           |                                                          Description                                                          |
| :---: | :------------------------------: | :---------------------------------------------------------------------------------------------------------------------------: |
| 0x01 |          [Ping](#ins-ping)          |               Instruction that checks whether the Packet has arrived to a device with the same ID as Packet ID               |
| 0x02 |          [Read](#ins-read)          |                                           Instruction to read data from the Device                                           |
| 0x03 |         [Write](#ins-write)         |                                            Instruction to write data on the Device                                            |
| 0x06 |     [Factory Reset](#ins-reset)     |                       Instruction that resets the Control Table to its initial factory default settings                       |
| 0x08 |        [Reboot](#ins-reboot)        |                                               Instruction to reboot the Device                                               |
| 0x10 |         [Clear](#ins-clear)         |                                           Instruction to reset certain information                                           |
| 0x20 | [Control Table Backup](#ins-backup) |              Instruction to store current Control Table status data to a Backup area or to restore EEPROM data.              |
| 0x55 |    [Status(Return)](#ins-status)    |                                           Return packet for the Instruction Packet                                           |
| 0x82 |     [Sync Read](#ins-sync-read)     |               For multiple devices, Instruction to read data from the same Address with the same length at once               |
| 0x83 |    [Sync Write](#ins-sync-write)    |               For multiple devices, Instruction to write data on the same Address with the same length at once               |
| 0x8A |  [Fast Sync Read](#ins-sync-read)  | For multiple devices, Instruction to read data from the same Address with the same length at once with shorter status packet |
| 0x8B | [Fast Sync Write](#ins-sync-write) | For multiple devices, Instruction to write data from the same Address with the same length at once with shorter status packet |

#### [1.4.6.Parameters]()

As the auxiliary data field for Instruction, its purpose is different for each Instruction.

#### [1.4.7.CRC-16 (IBM/ANSI)]()

16bit CRC field which checks if the Packet has been damaged during communication.

1. Devided into low and high bytes in the  Instruction Packet
2. Range of CRC calculation: From Header (FF FF FD 00) to Parameteres before CRC field in  Instruction Packet

- Polynomial : x16 + x15 + x2 + 1 (polynomial representation : 0x8005)
- Initial Value : 0

**CRC16 Calculation Code**

```
unsigned short update_crc(unsigned short crc_accum, unsigned char *data_blk_ptr, unsigned short data_blk_size)
{
    unsigned short i, j;
    unsigned short crc_table[256] = {
        0x0000, 0x8005, 0x800F, 0x000A, 0x801B, 0x001E, 0x0014, 0x8011,
        0x8033, 0x0036, 0x003C, 0x8039, 0x0028, 0x802D, 0x8027, 0x0022,
        0x8063, 0x0066, 0x006C, 0x8069, 0x0078, 0x807D, 0x8077, 0x0072,
        0x0050, 0x8055, 0x805F, 0x005A, 0x804B, 0x004E, 0x0044, 0x8041,
        0x80C3, 0x00C6, 0x00CC, 0x80C9, 0x00D8, 0x80DD, 0x80D7, 0x00D2,
        0x00F0, 0x80F5, 0x80FF, 0x00FA, 0x80EB, 0x00EE, 0x00E4, 0x80E1,
        0x00A0, 0x80A5, 0x80AF, 0x00AA, 0x80BB, 0x00BE, 0x00B4, 0x80B1,
        0x8093, 0x0096, 0x009C, 0x8099, 0x0088, 0x808D, 0x8087, 0x0082,
        0x8183, 0x0186, 0x018C, 0x8189, 0x0198, 0x819D, 0x8197, 0x0192,
        0x01B0, 0x81B5, 0x81BF, 0x01BA, 0x81AB, 0x01AE, 0x01A4, 0x81A1,
        0x01E0, 0x81E5, 0x81EF, 0x01EA, 0x81FB, 0x01FE, 0x01F4, 0x81F1,
        0x81D3, 0x01D6, 0x01DC, 0x81D9, 0x01C8, 0x81CD, 0x81C7, 0x01C2,
        0x0140, 0x8145, 0x814F, 0x014A, 0x815B, 0x015E, 0x0154, 0x8151,
        0x8173, 0x0176, 0x017C, 0x8179, 0x0168, 0x816D, 0x8167, 0x0162,
        0x8123, 0x0126, 0x012C, 0x8129, 0x0138, 0x813D, 0x8137, 0x0132,
        0x0110, 0x8115, 0x811F, 0x011A, 0x810B, 0x010E, 0x0104, 0x8101,
        0x8303, 0x0306, 0x030C, 0x8309, 0x0318, 0x831D, 0x8317, 0x0312,
        0x0330, 0x8335, 0x833F, 0x033A, 0x832B, 0x032E, 0x0324, 0x8321,
        0x0360, 0x8365, 0x836F, 0x036A, 0x837B, 0x037E, 0x0374, 0x8371,
        0x8353, 0x0356, 0x035C, 0x8359, 0x0348, 0x834D, 0x8347, 0x0342,
        0x03C0, 0x83C5, 0x83CF, 0x03CA, 0x83DB, 0x03DE, 0x03D4, 0x83D1,
        0x83F3, 0x03F6, 0x03FC, 0x83F9, 0x03E8, 0x83ED, 0x83E7, 0x03E2,
        0x83A3, 0x03A6, 0x03AC, 0x83A9, 0x03B8, 0x83BD, 0x83B7, 0x03B2,
        0x0390, 0x8395, 0x839F, 0x039A, 0x838B, 0x038E, 0x0384, 0x8381,
        0x0280, 0x8285, 0x828F, 0x028A, 0x829B, 0x029E, 0x0294, 0x8291,
        0x82B3, 0x02B6, 0x02BC, 0x82B9, 0x02A8, 0x82AD, 0x82A7, 0x02A2,
        0x82E3, 0x02E6, 0x02EC, 0x82E9, 0x02F8, 0x82FD, 0x82F7, 0x02F2,
        0x02D0, 0x82D5, 0x82DF, 0x02DA, 0x82CB, 0x02CE, 0x02C4, 0x82C1,
        0x8243, 0x0246, 0x024C, 0x8249, 0x0258, 0x825D, 0x8257, 0x0252,
        0x0270, 0x8275, 0x827F, 0x027A, 0x826B, 0x026E, 0x0264, 0x8261,
        0x0220, 0x8225, 0x822F, 0x022A, 0x823B, 0x023E, 0x0234, 0x8231,
        0x8213, 0x0216, 0x021C, 0x8219, 0x0208, 0x820D, 0x8207, 0x0202
    };

    for(j = 0; j < data_blk_size; j++)
    {
        i = ((unsigned short)(crc_accum >> 8) ^ data_blk_ptr[j]) & 0xFF;
        crc_accum = (crc_accum << 8) ^ crc_table[i];
    }

    return crc_accum;
}
```

**CRC Calculation Example**

- unsigned short update_crc(unsigned short crc_accum, unsigned char *data_blk_ptr, unsigned short data_blk_size);
- Return Value : 16bit CRC Value
- Arguments
  - crc_accum : set as ‘0’
  - data_blk_ptr : Packet array pointer
  - data_blk_size : number of bytes in the Packet excluding the CRC
  - data_blk_size = Header(3) + Reserved(1) + Packet ID(1) + Packet Length(2) + Packet Length - CRC(2) = 3 + 1 + 1 + 2 + Pakcet Length - 2 = 5 + Packet Length;
  - Packet Length = (LEN_H << 8 ) + LEN_L;  //Little-endian
- Packet Analysis and CRC Calculation
  - Example Packet(Read Instruction Packet to read the 2 bytes from address 0x0000)
    unsigned char TxPacket[] = {0xFF, 0xFF, 0xFD, 0x00, 0x01, 0x07, 0x00, 0x02, 0x00, 0x00, 0x02, 0x00, CRC_L, CRC_H}
  - CRC calculation
    CRC = update_crc(0, TxPacket , 12);   // 12 = 5 + Packet Length(7)
    CRC_L = (CRC & 0x00FF);               //Little-endian
    CRC_H = (CRC>>8) & 0x00FF;

### [1.5.Status Packet]()

Status Packet is the response packet transmitted from the device to a main controller. Note that it has the same construction as the Instruction Packet except the ERROR field is added.

| Header 1 | Header 2 | Header 3 | Reserved | Packet ID | Length 1 | Length 2 | Instruction | ERR | Param | Param | Param | CRC 1 | CRC2 |
| -------- | -------- | -------- | -------- | --------- | -------- | -------- | ----------- | --- | ----- | ----- | ----- | ----- | ---- |
| 0xFF     | 0xFF     | 0xFD     | 0x00     | ID        | Len_L    |          |             |     |       |       |       |       |      |

#### [1.5.1.Instruction]()

Instruction of the Status Packet is designated to 0x55 (Status)

### [1.6.Instruction Details]()

Note that given examples use the following abbreviation to provide clear information.

- Header : H
- Reserved: RSRV
- Length: LEN
- Instruction: INST
- Error: ERR
- Param: P

#### [1.6.1.Ping]()

**Description**

- Instruction to check the existence of a Device and basic information
- Regardless of the Status Return Level of the Device, the Status Packet is always sent to Ping Instruction.

**Packet Parameters**

| Satatus Packet | Description      |
| -------------- | ---------------- |
| Param 1        | Model Number LSB |
| Param 2        | Model Number MSB |
| Param 3        | Firmware Version |

**Example**

- ID1(JA40) : For Model Number 5210(0x0FAA), Version of Firmware 07(0x07)
- Instruction Packet ID : 1

**Ping Instruction Packet**

|  H1  | H2   | H3   | RSRV | Packet ID | LEN1 | LEN2 | INST | CRC 1 | CRC 2 |
| :--: | ---- | ---- | :--: | :-------: | ---- | ---- | ---- | ----- | ----- |
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01   | 0x03 | 0x00 | 0x01 | 0x19  | 0x4E  |

**ID 1 Status Packet**

|  H1  | H2   | H3   | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR  | P1 | P2 | P3 | CRC 1 | CRC 2 |
| :--: | ---- | ---- | :--: | :-------: | ---- | ---- | ---- | ---- | -- | -- | -- | :---: | ----- |
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01   | 0x07 | 0x00 | 0x55 | 0x00 | 5a | 17 | 07 |  10  | B9    |

#### [1.6.2.Read]()

**Description**

- Instruction to read a value from Control Table
- Read Instruction does not respond to Broadcast ID(254 (0xFE))

**Packet Parameters**

| Instruction Packet | Description                               |
| ------------------ | ----------------------------------------- |
| Param 1            | Low-order byte from the starting address  |
| Param 2            | High-order byte from the starting address |
| Param 3            | Low-order byte from the data length (X)   |
| Param 4            | High-order byte from the data length (X)  |

| Status Packet | Description |
| ------------- | ----------- |
| Param 1       | First Byte  |
| Param 2       | Second Byte |
| ...           | ...         |
| Param X       | X-th Byte   |

**Example**

- ID1(JA40) : Present Position(580, 0x0244, 4[byte]) = 27796(0x00006C94)

**Read Instruction Packet**

| H1   | H2   | H3   | RSRV | Packet ID | LEN1 | LEN2 | INST | P1   | P2   | P3   | P4   | CRC 1 | CRC 2 |
| ---- | ---- | ---- | :--: | :-------: | ---- | ---- | ---- | ---- | ---- | ---- | ---- | :---: | :---: |
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01   | 0x07 | 0x00 | 0x02 | 0x44 | 0x02 | 0x04 | 0x00 | 0x14 | 0x95 |

**ID 1 Status Packet**

|  H1  | H2   | H3   | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR | P1 | P2 | P3 | P4 | CRC 1 | CRC 2 |
| :--: | ---- | ---- | :--: | :-------: | ---- | ---- | ---- | --- | -- | -- | -- | -- | :---: | :---: |
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01   | 0x08 | 0x00 | 0x55 | 0x0 | 94 | 6C | 00 | 00 |  F5  |  AF  |

#### [1.6.3.Write]()

**Description**

- Instruction to write a value on the Control Table

**Packet Parameters**

| Instruction Packet | Description                               |
| ------------------ | ----------------------------------------- |
| Param 1            | Low-order byte from the starting address  |
| Param 2            | High-order byte from the starting address |
| Param 2+1          | First Byte                                |
| Param 2+2          | Second Byte                               |
| ...                | ...                                       |
| Param 2+X          | X-th Byte                                 |

**Example**

- ID1(JA40) : Write 2000(0x000007D0) to Goal Position(564, 0x0234, 4[byte])

**Write Instruction Packet**

| H1   | H2   | H3   | RSRV | Packet ID | LEN1 | LEN2 | INST | P1   | P2   | P3   | P4   | P5   | P6   | CRC 1 | CRC 2 |
| ---- | ---- | ---- | ---- | --------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ----- | ----- |
| 0xFF | 0xFF | 0xFD | 0x00 | 0x01      | 0x09 | 0x00 | 0x03 | 0x34 | 0x02 | 0xD0 | 0x07 | 0x00 | 0x00 | 0x1E  | 0xC9  |

**ID 1 Status Packet**

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR | CRC 1 | CRC 2 |
| :--: | :--: | :--: | :--: | :-------: | :--: | :--: | :--: | :--: | :---: | :---: |
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01   | 0x04 | 0x00 | 0x55 | 0x00 | 0xA1 | 0x0C |

#### [1.6.4.Factory Reset]()

**Description**

- Instruction that resets the Control Table to its initial factory default settings.
- When Factory Reset (0x06) Instruction is performed, a device is rebooted and the LED blinks four times in a row.
- In case of when **Packet ID** is a Broadcast ID `0xFE` and **Option** is Reset All `0xFF`, Factory Reset Instruction(0x06) will **NOT** be activated.

**Packet Parameters**

| Instruction Packet | Description                                                                           |
| ------------------ | ------------------------------------------------------------------------------------- |
| Param 1            | 0xFF : Reset all  0x01 : Reset all except ID  0x02 : Reset all except ID and Baudrate |

**Example**

- ID1(JA40) : Apply reset with option 0x01(Reset all except ID)

**Factory reset Instruction Packet**

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST |  P1  | CRC 1 | CRC 2 |
| :--: | :--: | :--: | :--: | :-------: | :--: | :--: | :--: | :--: | :---: | :---: |
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01   | 0x04 | 0x00 | 0x06 | 0x01 | 0xA1 | 0xE6 |

**ID 1 Status Packet**

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR | CRC 1 | CRC 2 |
| :--: | :--: | :--: | :--: | :-------: | :--: | :--: | :--: | :--: | :---: | :---: |
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01   | 0x04 | 0x00 | 0x55 | 0x00 | 0xA1 | 0x0C |

#### [1.6.5.Reboot]()

**Description**

- Instruction to reboot the device

**Example**

- ID1(JA40)

**Reboot Instruction Packet**

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | CRC 1 | CRC 2 |
| :--: | :--: | :--: | :--: | :-------: | :--: | :--: | :--: | :---: | :---: |
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01   | 0x03 | 0x00 | 0x08 | 0x2F | 0x4E |

**ID 1 Status Packet**

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR | CRC 1 | CRC 2 |
| :--: | :--: | :--: | :--: | :-------: | :--: | :--: | :--: | :-: | :---: | :---: |
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01   | 0x04 | 0x00 | 0x55 | 0x0 | 0xA1 | 0x0C |

#### [1.6.6.Clear]()

**Description**

- Instruction to clear the fault.

**Packet Parameters**

- None

**Example**

- ID1(JA40)

**Clear Instruction Packet**

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 |      INST      | p1   | p2   | p3   | p4   | CRC 1 | CRC 2 |
| :--: | :--: | :--: | :--: | :-------: | :--: | :--: | :------------: | ---- | ---- | ---- | ---- | :---: | :---: |
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01   | 0x08 | 0x00 | **0x10** | 0x01 | 0x44 | 0x58 | 0x4C | 0xE0 | 0xCE |

**ID 1 Status Packet**

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR | CRC 1 | CRC 2 |
| :--: | :--: | :--: | :--: | :-------: | :--: | :--: | :--: | :--: | :---: | :---: |
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01   | 0x04 | 0x00 | 0x55 | 0x00 | 0xA1 | 0x0C |

#### [1.6.7.Control Table Backup]()

**Description**

- Instruction to store current Control Table status data to a Backup area, or to restore EEPROM data.
- The Control Table Backup works properly only if **Torque Enable** in RAM area is set as '0' (Torque Off status).
- Available items in Control Table for data backup:

  - All Data in EERPOM
  - Velocity P.I Gains
  - Position P.I.D Gains
  - Feedforward 1st & 2nd Gains

**Packet Parameters**

| P1   | P2-P5               | Description                                              |
| ---- | ------------------- | -------------------------------------------------------- |
| 0x01 | 0x43 0x54 0x52 0x4C | Store current Control Table status data to a Backup area |

**Example**

- ID1(JA40) :Backup the Control Table.

**Backup Instruction Packet**

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST |  P1  |  P2  |  P3  |  P4  |  P5  | CRC1 | CRC2 |
| :--: | :--: | :--: | :--: | :-------: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01   | 0x08 | 0x00 | 0x20 | 0x01 | 0x43 | 0x54 | 0x52 | 0x4C | 0x16 | 0xF5 |

**ID 1 Status Packet**

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR | CRC1 | CRC2 |
| :--: | :--: | :--: | :--: | :-------: | :--: | :--: | :--: | :--: | :--: | :--: |
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01   | 0x04 | 0x00 | 0x55 | 0x00 |      |      |

#### [1.6.8.Fast Sync Read]()

**Description**

- Instruction to read data from multiple devices simultaneously using one Instruction Packet
- The Address and Data Length of the data must all be the same.
- If the Address of the data is not continual, an Indirect Address can be used.
- Status Packet will be returned in order, according to input ID in the Instruction Packet.
- Packet ID field : 0xFE (Broadcast ID)

**Packet Parameters**

| Instruction Packet | Description                               |
| :----------------: | :---------------------------------------- |
|    Parameter 1    | Low-order byte from the starting address  |
|    Parameter 2    | High-order byte from the starting address |
|    Parameter 3    | Low-order byte from the data length(X)    |
|    Parameter 4    | High-order byte from the data length(X)   |
|   Parameter 4+1   | `1st Device` ID                         |
|   Parameter 4+2   | `2nd Device` ID                         |
|         ⋯         | ⋯                                        |
|   Parameter 4+n   | `Nnd Device` ID                         |

|  Status Packet  | Description                             |
| :--------------: | :-------------------------------------- |
|   Parameter 1   | `1st Device` ID                       |
|   Parameter 2   | `1st Device` First Byte               |
|   Parameter 3   | `1st Device` Second Byte              |
|        ⋯        | ⋯                                      |
|   Parameter X   | `1st Device` X-th Byte                |
|  Parameter X+1  | `1st Device` Low-order byte from CRC  |
|  Parameter X+2  | `1st Device` High-order byte from CRC |
|  Parameter X+3  | `2nd Device` Error                    |
|  Parameter X+4  | `2nd Device` ID                       |
| Parameter X+4+1 | `2nd Device` First Byte               |
| Parameter X+4+2 | `2nd Device` Second Byte              |
|        ⋯        | ⋯                                      |
|  Parameter 2X+4  | `2nd Device` X-th Byte                |
| Parameter 2X+4+1 | `2nd Device` Low-order byte from CRC  |
| Parameter 2X+4+2 | `2nd Device` High-order byte from CRC |
|        ⋯        | ⋯                                      |
|  Parameter nX+4  | `Nnd Device` X-th Byte                |

**Example**

- ID1(JA40) : Present Position(580, 0x0244, 4[byte]) = 2000(0x000007D0)
- ID2(JA40) : Present Position(580, 0x0244, 4[byte]) = 9(0x00000009)

**Fast Sync Read Instruction Packet**

| H1   | H2   | H3   | RSRV | Packet ID | LEN1 | LEN2 | INST | P1   | P2   | P3   | P4   | P5   | P6   | CRC 1 | CRC 2 |
| ---- | ---- | ---- | ---- | --------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ----- | ----- |
| 0xFF | 0xFF | 0xFD | 0x00 | 0xFE      | 0x09 | 0x00 | 0x8A | 0x44 | 0x02 | 0x04 | 0x00 | 0x01 | 0x02 | 0x72  | 0xF2  |

**ID1 Status Packet**

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR | ID1 |  D1  |  D2  |  D3  |  D4  | CRC1 | CRC2 |
| :--: | :--: | :--: | :--: | :-------: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| 0xFF | 0xFF | 0xFD | 0x00 |   0xFE   | 0x11 | 0x00 | 0x55 | 0x00 | 0x01 | 0xD0 | 0x07 | 0x00 | 0x00 | 0x0C | 0x83 |

**ID2 Status Packet**

| ERR | ID2 |  D1  |  D2  |  D3  |  D4  | CRC1 | CRC2 |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| 0x00 | 0x02 | 0x09 | 0x00 | 0x00 | 0x00 | 0x11 | 0x40 |

#### [1.6.9.Fast Sync Write]()

**Description**

- Instruction to write data from multiple devices simultaneously using one Instruction Packet
- The Address and Data Length of the data must all be the same.
- Status Packet will be returned in order, according to input ID in the Instruction Packet.
- Packet ID field : 0xFE (Broadcast ID)

**Packet Parameters**

| Instruction Packet | Description                               |
| :----------------: | :---------------------------------------- |
|    Parameter 1    | Low-order byte from the starting address  |
|    Parameter 2    | High-order byte from the starting address |
|    Parameter 3    | Low-order byte from the data length(X)    |
|    Parameter 4    | High-order byte from the data length(X)   |
|    Parameter 5    | [1st Device] ID                           |
|   Parameter 5+1   | [1st Device] 1st Byte                     |
|   Parameter 5+2   | [1st Device] 2nd Byte                     |
|        ...        | [1st Device]...                           |
|   Parameter 5+X   | [1st Device] X-th Byte                    |
|    Parameter 6    | [2nd Device] ID                           |
|   Parameter 6+1   | [2nd Device] 1st Byte                     |
|   Parameter 6+2   | [2nd Device] 2nd Byte                     |
|        ...        | [2nd Device]...                           |
|   Parameter 6+X   | [2nd Device] X-th Byte                    |
|        ...        | ...                                       |

| Status Packet | Description                             |
| :------------: | :-------------------------------------- |
|  Parameter 1  | `1st Device` ID                       |
|  Parameter 2  | `1st Device` Low-order byte from CRC  |
|  Parameter 3  | `1st Device` High-order byte from CRC |
|  Parameter 4  | `2nd Device` Error                    |
|  Parameter 5  | `2nd Device` ID                       |
|  Parameter 6  | `2nd Device` Low-order byte from CRC  |
|  Parameter 7  | `2nd Device` High-order byte from CRC |
|       ⋯       | ⋯                                      |
| Parameter 4N-2 | `Nnd Device` Low-order byte from CRC  |
| Parameter 4N-1 | `Nnd Device` High-order byte from CRC |

**Example**

- ID1(JA40) : Write 150(0x00000096) to Goal Position(116, 0x0074, 4[byte])
- ID2(JA40) : Write 170(0x000000AA) to Goal Position(116, 0x0074, 4[byte])

**Fast Sync Write Instruction Packet**

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST |  P1  |  P2  |  P3  |  P4  |
| :--: | :--: | :--: | :--: | :-------: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| 0xFF | 0xFF | 0xFD | 0x00 |   0xFE   | 0x11 | 0x00 | 0x8B | 0x74 | 0x00 | 0x04 | 0x00 |

|  P5  |  P6  |  P7  |  P8  |  P9  | P10 | P11 | P12 | P13 | P14 | CRC 1 | CRC 2 |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :---: | :---: |
| 0x01 | 0x96 | 0x00 | 0x00 | 0x00 | 0x02 | 0xAA | 0x00 | 0x00 | 0x00 | 0xB2 | 0x8F |

**Status Packet**

**ID 1 Status Packet**

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR | ID1 | CRC1 | CRC2 |
| :--: | :--: | :--: | :--: | :-------: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| 0xFF | 0xFF | 0xFD | 0x00 |   0xFE   | 0x09 | 0x00 | 0x55 | 0x00 | 0x01 | 0x86 | 0x8B |

**ID 2 Status Packet**

| ERR | ID2 | CRC1 | CRC2 |
| :--: | :--: | :--: | :--: |
| 0x00 | 0x07 | 0xEB | 0x64 |

## [2.Control Table](#control-table)

The Control Table is a structure of data implemented in the device. Users can read a specific Data to get status of the device with Read Instruction Packets, and modify Data as well to control the device with WRITE Instruction Packets.

### [2.1.Control Table, Data, Address]()

The Control Table is a structure that consists of multiple Data fields to store status or to control the device. Users can check current status of the device by reading a specific Data from the Control Table with Read Instruction Packets. WRITE Instruction Packets enable users to control the device by changing specific Data in the Control Table. The Address is a unique value when accessing a specific Data in the Control Table with Instruction Packets. In order to read or write data, users must designate a specific Address in the Instruction Packet.

#### [2.1.1.Area (EEPROM, RAM)](#Area "EEPROM, RAM")

The Control Table is divided into 2 Areas. Data in the RAM Area is reset to initial values when the power is reset(Volatile). On the other hand, data in the EEPROM Area is maintained even when the device is powered off(Non-Volatile).

**NOTE:**Data in the EEPROM Area can only be written to if Torque Enable(512) is cleared to ‘0’(Torque OFF).****

#### [2.1.2.Size](#Size)

The Size of data varies from 1 ~ 4 bytes depend on their usage. Please check the size of data when updating the data with an Instruction Packet.

#### [2.1.3.Access](#Access)

The Control Table has two different access properties. ‘RW’ property stands for read and write access permission while ‘R’ stands for read only access permission. Data with the read only property cannot be changed by the WRITE Instruction. Read only property(‘R’) is generally used for measuring and monitoring purpose, and read write property(‘RW’) is used for controlling device.

### [2.2.Control Table of EEPROM Area]()

| Address | Size(Byte) |               Data Name               | Access | Initial<br />Value |                 Range                 |               Unit               |
| :-----: | :--------: | :------------------------------------: | :----: | :----------------: | :------------------------------------: | :-------------------------------: |
|    0    |     2     |       [Model Number](#model-number)       |   R   |        5210        |                   -                   |                 -                 |
|    2    |     4     |  [Model Information](#model-information)  |   R   |         -         |                   -                   |                 -                 |
|    6    |     1     |   [Firmware Version](#firmware-version)   |   R   |         -         |                   -                   |                 -                 |
|    7    |     1     |                 [ID](#id)                 |   RW   |         1         |                0 ~ 252                |                 -                 |
|    8    |     1     |          [Baud Rate](#baud-rate)          |   RW   |         4         |                 0 ~ 4                 |                 -                 |
|   10   |     1     |         [Drive Mode](#drive-mode)         |   RW   |         0         |                 0 ~ 3                 |                 -                 |
|   11   |     1     |     [Operating Mode](#operating-mode)     |   RW   |         4         |                 4、17                 |                 -                 |
|   20   |     4     |      [Homing Offset](#homing-offset)      |   RW   |         0         |           0 ~`<br>` 32,768           |             1 [count]             |
|   31   |     1     |  [Temperature Limit](#temperature-limit)  |   RW   |         80         |                0 ~ 100                |            1 [&deg;C]            |
|   36   |     2     |          [PWM Limit](#pwm-limit)          |   RW   |        1500        |               0 ~ 37,500               |           0.000026 [%]           |
|   38   |     2     |      [Current Limit](#current-limit)      |   RW   |        2000        |               0 ~ 15,000               |            2.5177 [mA]            |
|   40   |     4     | [Acceleration Limit](#acceleration-limit) |   RW   |        2000        |             0 ~ 3,992,644             | 1 [rev/min `<sup>`2 `</sup>`] |
|   44   |     4     |     [Velocity Limit](#velocity-limit)     |   RW   |       2,000       |               0 ~ 6,000               |          0.01 [rev/min]          |
|   48   |     4     | [Max Position Limit](#max-position-limit) |   RW   |       16,384       | -2,147,483,648 ~`<br>` 2,147,483,648 |             1 [count]             |
|   52   |     4     | [Min Position Limit](#min-position-limit) |   RW   |      -16,384      | -2,147,483,648 ~`<br>` 2,147,483,648 |             1 [count]             |

### [2.3.Control Table of RAM Area]()

| Address | Size(Byte) |                  Data Name                  | Access | Initial<br />Value |                         Range                         |          Unit          |
| :-----: | :--------: | :------------------------------------------: | :----: | :----------------: | :----------------------------------------------------: | :---------------------: |
|   512   |     1     |         [Torque Enable](#torque-enable)         |   RW   |         0         |                         0 ~ 1                         |            -            |
|   518   |     1     | [Hardware Error Status](#hardware-error-status) |   R   |         0         |                        0 ~ 254                        |            -            |
|   522   |     2     |                [Profile Time]()                |   RW   |        2000        |                       0 ~ 65,536                       |        1 [msec]        |
|   524   |     2     |      [Velocity I Gain](#velocity-pi-gain)      |   RW   |         0         |                       0 ~ 32,767                       |            -            |
|   526   |     2     |      [Velocity P Gain](#velocity-pi-gain)      |   RW   |         50         |                       0 ~ 32,767                       |            -            |
|   528   |     2     |      [Position D Gain](#position-pid-gain)      |   RW   |         50         |                       0 ~ 32,767                       |            -            |
|   530   |     2     |      [Position I Gain](#position-pid-gain)      |   RW   |         5         |                       0 ~ 32,767                       |            -            |
|   532   |     2     |      [Position P Gain](#position-pid-gain)      |   RW   |        2000        |                       0 ~ 32,767                       |            -            |
|   536   |     2     |  [Feedforward 2nd Gain](#feedforward-2nd-gain)  |   RW   |        2000        |                       0 ~ 32,767                       |            -            |
|   538   |     2     |  [Feedforward 1st Gain](#feedforward-1st-gain)  |   RW   |        150        |                       0 ~ 32,767                       |            -            |
|   550   |     2     |          [Goal Current](#goal-current)          |   RW   |         -         |        -Current Limit(38) ~``Current Limit(38)        |       2.5177 [mA]       |
|   552   |     4     |         [Goal Velocity](#goal-velocity)         |   RW   |         -         |       -Velocity Limit(44) ~``Velocity Limit(44)       |     0.01 [rev/min]     |
|   556   |     4     |  [Profile Acceleration](#profile-acceleration)  |   RW   |         -         |              0 ~``Acceleration Limit(40)              |    1 [rev/min ^2^ ]    |
|   560   |     4     |      [Profile Velocity](#profile-velocity)      |   RW   |         -         |                0 ~``Velocity Limit(44)                |     0.01 [rev/min]     |
|   564   |     4     |         [Goal Position](#goal-position)         |   RW   |         -         | Min Position Limit(52) ~``<br />Max Position Limit(48) |        1[pulse]        |
|   568   |     2     |         [Realtime Tick](#realtime-tick)         |   R   |         -         |                       0 ~ 32,767                       |        1 [msec]        |
|   571   |     1     |         [Moving Status](#moving-status)         |   R   |         -         |                           -                           |            -            |
|   572   |     2     |           [Present PWM](#present-pwm)           |   R   |         -         |                           -                           |       0.0096 [%]       |
|   574   |     2     |       [Present Current](#present-current)       |   R   |         -         |                           -                           |       2.5177 [mA]       |
|   576   |     4     |      [Present Velocity](#present-velocity)      |   R   |         -         |                           -                           | 0.0078125 [count/100us] |
|   580   |     4     |      [Present Position](#present-position)      |   R   |         -         |                           -                           |        1 [count]        |
|   584   |     4     |   [Velocity Trajectory](#velocity-trajectory)   |   R   |         -         |                           -                           |     0.01 [rev/min]     |
|   588   |     4     |   [Position Trajectory](#position-trajectory)   |   R   |         -         |                           -                           |        1 [pulse]        |
|   594   |     1     |   [Present Temperature](#present-temperature)   |   R   |         -         |                           -                           |         1 [°C]         |
|   878   |     1     |          [Backup Ready](#backup-ready)          |   R   |         -         |                         0 ~ 1                         |            -            |

### [2.4.Control table description]()

NOTE : Data in the EEPROM Area can only be written when the value of [Torque Enable(512)] is cleared to `0`.

#### [2.4.1.Model Number(0)]()

This address stores the version number of the firmware installed on your JA actuator.

#### [2.4.2.ID(7)](#ID7)

The JA-Actuator ID is used by the JA-Actuator network to identify individual actuators for instruction packets. Values between 0 and 253 (0xFD) can be assigned to individual JA-Actuator actuators and address 254(0xFE) is is reserved for the global broadcast ID to send instruction packets to all connected devices simultaneously.

**NOTE** : DYNAMIXEL IDs must be unique for each device connected to a DYNAMIXEL network. Multiple devices sharing a single ID may cause communications issues or control failure.

#### [2.4.3.Baud Rate(8)	]()

|   Value   |   Baud Rate   | Actual Baud Rate | Margin of Error |
| :--------: | :-----------: | :--------------: | :-------------: |
| 4(Default) |   2M [bps]   |    2,000,000    |     0.000%     |
|     3     |   1M [bps]   |    1,000,000    |     0.000%     |
|     2     | 115,200 [bps] |     115,226     |     0.023%     |
|     1     | 57,600 [bps] |      57,613      |     0.023%     |
|     0     |  9,600 [bps]  |      9,600      |     0.000%     |

#### [2.4.4.Drive Mode(10)]()

The Drive Mode control table register allows the configuration of several settings related to the movement of your JA actuator:

* Normal/Reverse mode allows the configuration of the movement direction of your JA actuator.

|     Bit     |          Item          | Description                                                                                                                                                                                                                                                                  |
| :---------: | :--------------------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Bit 7(0x80) |           -           | Unused, always ‘0’                                                                                                                                                                                                                                                         |
| Bit 6(0x40) |           -           | Unused, always ‘0’                                                                                                                                                                                                                                                         |
| Bit 5(0x20) |           -           | Unused, always ‘0’                                                                                                                                                                                                                                                         |
| Bit 4(0x10) |           -           | Unused, always ‘0’                                                                                                                                                                                                                                                         |
| Bit 3(0x08) |           -           | Unused, always ‘0’                                                                                                                                                                                                                                                         |
| Bit 2(0x04) | Profile Configuration | **[0]** Velocity-based Profile: Create profiles based on velocity.<br />**[1]** Time-based Profile: Create profiles based on time.                                                                                                                               |
| Bit 1(0x02) | Disable/Enable Profile | **[0]** Enable Profile<br />**[1]** Disable Profile                                                                                                                                                                                                              |
| Bit 0(0x01) |  Normal/Reverse Mode  | **[0]** Normal movement directions: Positive movement direction is counterclockwise, and negative movement direction is clockwise.<br />**[1]** Reverse Mode: Negative movement directions are counterclockwise, and positive movement directions are clockwise. |

#### [2.4.5.Operation Mode(11)]()

Configure the selected operating mode of your JA-Actuator.

| Value      | Operating Mode                 | Description                                                                                                                                                                                                                |
| :--------- | :----------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0          | Current Control Mode           | This mode controls only current/torque regardless of speed and position. This mode is ideal for a gripper or other system that only requires torque control or a system that has additional velocity/position controllers. |
| 1          | Velocity Control Mode          | This mode controls velocity and current, but does not control position.                                                                                                                                                    |
| 3(Default) | Position Control Mode          | This mode controls position, velocity and current. The position range is configured by the Max Position Limit(48) and the Min Position Limit(52) control table items.                                                      |
| 4          | Extended Position Control Mode | This mode is similar to Position Control Mode, but is not limited by the Position Limit control table items. This allows multi turn position based control for applications requiring continuous rotation.                 |
| 17         | Calibration Mode               | Self-calibrate the parameters of motor to improve the accuracy of driver recognition.                                                                                                                                      |

#### [2.4.6.Homing Offset(20)]()

Users can adjust the Home position by setting Home Offset(20). The Homing Offset value is added to the Present Position(580).
Present Position(580) = Actual Position + Homing Offset(20).

| Unit      | Value Range        |
| --------- | ------------------ |
| 1 [count] | 0 ~`<br>` 32,768 |

#### [2.4.7.Temperature Limit(31)]()

This value limits operating temperature.
When the Present Temperature(594) that indicates internal temperature of device is greater than the Temperature Limit(31), the Overheating Error Bit(0x04) in the Hardware Error Status(518) will be set.
If Overheating Error Bit(0x04) is configured in the [Shutdown(63)](), [Torque Enable(512)]() will be set to ‘0’ (Torque OFF). For more details, please refer to the [Shutdown(63)]() section.

| Unit          | Value Range | Description   |
| ------------- | ----------- | ------------- |
| About 1 [°C] | 0 ~ 100     | 0 ~ 100 [°C] |

**NOTE** : Do not set the temperature lower/higher than the default value. When the temperature alarm shutdown occurs, wait for 20 minutes to cool the temperature before reuse. Keep using the product with high temperature can cause severe damage to the device.

#### [2.4.8.PWM Limit(36)]()

This value indicates the maximum PWM output.
Goal PWM(548) cannot be configured with any values exceeding [PWM Limit(36)]().
[PWM Limit(36)]() is commonly applied in all operating mode as an output limit, therefore decreasing PWM output will also decrease torque and velocity of the device.

| Unit         | Range      |
| ------------ | ---------- |
| 0.000026 [%] | 0 ~ 37,500 |

#### [2.4.9.Current Limit(38)]()

This value indicates the maximum current limit.
Goal Current(550) cannot be configured with any values exceeding [Current Limit(38)](). Attempting to write an invalid value will fail and set the Limit Error Bit in the error field of the Status Packet.

| Unit        | Range      |
| ----------- | ---------- |
| 2.5177 [mA] | 0 ~ 15,000 |

#### [2.4.10.Acceleration Limit(40)]()

This value indicates the maximum acceleration limit.
[Profile Acceleration(556)]() cannot be configured with any values exceeding Acceleration Limit(40). Writing invalid or the value over its limit, the Status Packet sends the Data Limit Error via its Error field.

| Unit             | Range         |
| ---------------- | ------------- |
| 1 [rev/min ^2^ ] | 0 ~ 3,992,644 |

#### [2.4.11.Velocity Limit(44)]()

This value indicates maximum velocity of [Goal Velocity(552)]() and Profile Velocity(562). Goal Velocity(552) and [Profile Velocity(560)]() cannot be configured with any values exceeding Velocity Limit(44). Writing invalid or the value over its limit, the Status Packet sends the Data Limit Error via its Error field.

| Unit           | Range     |
| -------------- | --------- |
| 0.01 [rev/min] | 0 ~ 6,000 |

#### [2.4.12.Max/Min Position Limit(48, 52)]()

These values limit maximum and minimum desired positions within a single turn(-2,147,483,648 ~2,147,483,648). The Goal Position(564) can’t exceed these values. Writing invalid or the value over its limit, the Status Packet sends the Data Limit Error via its Error field.

| Unit      | Range                         |
| --------- | ----------------------------- |
| 1 [count] | -2,147,483,648 ~2,147,483,648 |

**NOTE**: Maximum position limit (48) and minimum position limit (52) are only used when the operating mode is position control mode (joint mode).

#### [2.4.13.Torque Enable(512)]()

Torque Enable(512) determines Torque ON/OFF. Writing ‘1’ to Torque Enable’s address will turn on the Torque and all Data in the EEPROM area will be locked.

| Value      | Description                    |
| ---------- | ------------------------------ |
| 0(Default) | Torque Off                     |
| 1          | Torque On and lock EEPROM area |

**NOTE** : [Present Position(580)]() can be reset when [Operating Mode(11)]() and [Torque Enable(512)]() are updated. For more details,

#### [2.4.14.Hardware Error(518)Status]()

This value indicates hardware error status. The JA actuator can protect itself by detecting dangerous situations that could occur during the operation.

|  Bit  |             Item             | Description                                                                                        |
| :---: | :--------------------------: | :------------------------------------------------------------------------------------------------- |
| Bit 7 |   Check EEPROM Area Error   | Saving or loading system config form EEPROM area is failed                                         |
| Bit 6 | Position Limit Reached Error | In position control mode, the current position has moved beyond the Max/Min Position Limit range.  |
| Bit 5 |        Overload Error        | Detects that persistent load exceeds maximum output                                                |
| Bit 4 |              -              | Not used, always '0'                                                                               |
| Bit 3 |        Encoder Error        | An issue has occurred with the Encoder IC                                                          |
| Bit 2 |      Overheating Error      | Detects that internal temperature exceeds the configured operating temperature                     |
| Bit 1 | Velocity Limit Reached Error | Detects the Present Velocity or Profile Velocity when writing Goal Position is over Velocity Limit |
| Bit 0 |              -              | Not used, always '0'                                                                               |

#### [2.4.15.Profile Time]()

If time-based profile is selected in [Drive Mode(10)], Profile Time will be used to generate a time based movement trajectory.

|       item       |  Unit  | Control Mode     | Description                                                               |
| :---------------: | :----: | :--------------- | :------------------------------------------------------------------------ |
| Profile Time(522) | 1 [ms] | Position control | Total profile time.`<br>`If the value is 0, the profile is deactivated. |

#### [2.4.16.Velocity PI Gain(524, 526), Feedforward 2nd Gains(536)]()

These values indicate Gains of Velocity Control Mode. Velocity P Gain of the device’s internal controller is abbreviated to K~V~P.

|                           | Controller Gain | Range      | Description                   |
| ------------------------- | --------------- | ---------- | ----------------------------- |
| Velocity I Gain(524)      | K~V~I          | 0 ~ 32,767 | Velocity Integral Gain        |
| Velocity P Gain(526)      | K~V~P          | 0 ~ 32,767 | Velocity Proportional Gain    |
| Feedforward 2nd Gain(536) | K~FF2nd~       | 0 ~ 32,767 | Acceleration Feedforward Gain |

#### [2.4.17.Position PID Gain(528, 530, 532), Feedforward 1st Gains(538)]()

These Gains are used in Position Control Mode and Extended Position Control Mode. Gains of device’s internal controller can be calculated from Gains of the Control Table as shown below. Position P Gain of device’s internal controller is abbreviated to K~P~P.

|                           | Controller Gain | Range      | Description                |
| ------------------------- | --------------- | ---------- | -------------------------- |
| Position D Gain(528)      | K~P~D          | 0 ~ 32,767 | Position Derivative Gain   |
| Position I Gain(530)      | K~P~I          | 0 ~ 32,767 | Position Integral Gain     |
| Position P Gain(532)      | K~P~P          | 0 ~ 32,767 | Position Proportional Gain |
| Feedforward 1st Gain(538) | K~FF1st~       | 0 ~ 32,767 | Velocity Feedforward Gain  |

#### [2.4.18.Goal Current(550)]()

In Current Control Mode, Goal Current(550) can be used to set the desired current. Goal Current(550) sets a current limit of the current controller in Velocity Control Mode, Position Control Mode and Extended Position Control Mode.
Goal Current(550) cannot exceed [Current Limit(38)]().

#### [2.4.19.Goal Velocity(552)]()

In Velocity Control Mode, Goal Velocity(552) can be used to set the desired velocity.
The Goal Velocity(552) cannot exceed [Velocity Limit(44)](https://emanual.robotis.com/docs/en/dxl/p/ph54-200-s500-r/#velocity-limit44).
The Goal Velocity(552) is used to limit the input(velocity) of velocity controller in Position Control Mode and Extended Position Control Mode.

#### [2.4.20.Profile Acceleration(556)]()

When the [Drive Mode(10)]() is  **Velocity-based Profile** , Profile Velocity(560) sets the maximum velocity of the Profile.
When the [Drive Mode(10)]() is  **Time-based Profile** , Profile Velocity(560) sets the time span to reach the velocity (the total time) of the Profile.
The Profile Velocity(560) is to be applied to **Position Control Mode** or **Extended Position Control Mode** on the [Operating Mode(11)]().

| Velocity-based Profile | Values                                                                                            | Description                           |
| ---------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------- |
| Unit                   | 0.01 [rev/min]                                                                                    | Sets velocity of the Profile          |
| Range                  | 0 ~[Velocity Limit(44)](https://emanual.robotis.com/docs/en/dxl/p/ph54-200-s500-r/#velocity-limit44) | ‘0’ represents an infinite velocity |

| Time-based Profile | Values    | Description                                                                                                                                                                                       |
| ------------------ | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Unit               | 1 [msec]  | Sets the time span for the Profile                                                                                                                                                                |
| Range              | 0 ~ 32737 | ‘0’ represents an infinite velocity.<br />Profile Acceleration(556, Acceleration time) will not exceed 50% of Profile Velocity (560, the time span to reach the velocity of the Profile) value. |

**NOTE** : When Profile Velocity(560) is set to ‘0’, the profile’s acceleration will be ignored.

**NOTE** : Time-based Profile is available from firmware 12.

#### [2.4.21.Profile Velocity(560)]()

When the [Drive Mode(10)]() is  **Velocity-based Profile** , Profile Velocity(560) sets the maximum velocity of the Profile.
When the [Drive Mode(10)]() is  **Time-based Profile** , Profile Velocity(560) sets the time span to reach the velocity (the total time) of the Profile.
The Profile Velocity(560) is to be applied to **Position Control Mode** or **Extended Position Control Mode** on the [Operating Mode(11)]().

| Velocity-based Profile | Values                                                                                            | Description                           |
| ---------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------- |
| Unit                   | 0.01 [rev/min]                                                                                    | Sets velocity of the Profile          |
| Range                  | 0 ~[Velocity Limit(44)](https://emanual.robotis.com/docs/en/dxl/p/ph54-200-s500-r/#velocity-limit44) | ‘0’ represents an infinite velocity |

| Time-based Profile | Values    | Description                                                                                                                                                                                       |
| ------------------ | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Unit               | 1 [msec]  | Sets the time span for the Profile                                                                                                                                                                |
| Range              | 0 ~ 32737 | ‘0’ represents an infinite velocity.<br />Profile Acceleration(556, Acceleration time) will not exceed 50% of Profile Velocity (560, the time span to reach the velocity of the Profile) value. |

 **NOTE** : Time-based Profile is available from firmware v12.

#### [2.4.22.Goal Position(564)]()

Desired position can be set with [Goal Position(564)]().

The available input range is between [Min Position Limit(52)]() and [Max Position Limit(48)]() in Position Control Mode, while Extended Position Control Mode uses a value range between -2,147,483,648 ~ 2,147,483,647.

**NOTE** : [Present Position(580)]() represents a 4 byte continuous range from -2,147,483,648 to 2,147,483,647 when Torque is turned off regardless of Operating Mode(11).
However, [Present Position(580)]() will be reset to an absolute position value within one full rotation in the following cases:

1. When the Operating Mode(11) is changed to  **Position Control Mode** .
2. When torque is turned on in  **Position Control Mode** .
3. When the actuator is turned on or when rebooted using a [Reboot Instruction]().

Note that a [Present Position(580)]() value that has been reset to the absolute value within a single rotation will still be affected by the configured [Homing Offset(20)]() value.

#### [2.4.23.Realtime Tick(568)]()

This value indicates device’s internal time.

| Unit     | Value Range | Description                                      |
| -------- | ----------- | ------------------------------------------------ |
| 1 [msec] | 0 ~ 32,767  | The value resets to ‘0’ when it exceeds 32,767 |

#### [**2.4.24Moving Status(571)**]()

This value provides additional information about the movement. In-Position Bit(0x01) only works with Position Control Mode and Extended Position Control Mode.

|                  |      | Details                                                    | Description                                                                     |
| ---------------- | ---- | ---------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Bit 7            | 0x80 | -                                                          | Unused                                                                          |
| Bit 6            | 0x40 | -                                                          | Unused                                                                          |
| Bit 5 ``~``Bit 4 | 0x30 | Profile Type(0x30)``Profile Type(0x10)``Profile Type(0x00) | Trapezoidal Velocity Profile ``Rectangle Velocity Profile``Profile unused(Step) |
| Bit 3            | 0x08 | -                                                          | Unused                                                                          |
| Bit 2            | 0x04 | -                                                          | Unused                                                                          |
| Bit 1            | 0x02 | -                                                          | Unused                                                                          |
| Bit 0            | 0x01 | In-Position                                                | The device is reached to desired position                                       |

#### [2.4.25.Present PWM(572)]()

The Present PWM(124) indicates current PWM.

#### [2.4.26.Present Current(574)]()

This value indicates the present current flowing on the motor.

#### [2.4.27.Present Velocity(576)]()

This value indicates the present Velocity.

#### [2.4.28.Present Position(580)]()

This value indicates present Position. For more details, please refer to the [Goal Position(564)]().

**NOTE** : [Present Position(580)]() represents a 4 byte continuous range from -2,147,483,648 to 2,147,483,647 when Torque is turned off regardless of Operating Mode(11).
However, [Present Position(580)]() will be reset to an absolute position value within one full rotation in the following cases:

1. When the Operating Mode(11) is changed to  **Position Control Mode** .
2. When torque is turned on in  **Position Control Mode** .
3. When the actuator is turned on or when rebooted using a [Reboot Instruction]().

Note that a [Present Position(580)]() value that has been reset to the absolute value within a single rotation will still be affected by the configured [Homing Offset(20)]() value.

#### [2.4.29.Velocity Trajectory(584)]()

This is a desired velocity trajectory created by Profile. Operating method can be differ by control mode. For more details, please refer to the Profile Velocity(560).

1. **Velocity Control Mode** : When Profile reaches to the endpoint, Velocity Trajectory(584) becomes equal to Goal Velocity(552).
2. **Position Control Mode, Extended Position Control Mode** : The desired Velocity Trajectory is used to create Position Trajectory(588). When Profile reaches to an endpoint, Velocity Trajectory(584) is set to ‘0’.

#### [2.4.30.Position Trajectory(588)]()

This is a desired position trajectory created by Profile. This value is only used in Position Control Mode and Extended Position Control Mode. For more details, please refer to the [Profile Velocity(560)]().

#### [2.4.31.Present Input Voltage(592)]()

This value indicates present voltage that is being supplied to the device. For more details.

#### [2.4.32.Present Temperature(594)]()

This value indicates internal temperature of the device. For more details, please refer to the [Temperature Limit(31)]().

#### [2.4.33.Backup Ready(878)]()

The value in this address indicates whether the backup of the control table exists after sending the Control Table Backup Packet.

| Value | Description                     |
| ----- | ------------------------------- |
| 0     | The backup data doesn’t exist. |
| 1     | A saved backup data exists.     |
