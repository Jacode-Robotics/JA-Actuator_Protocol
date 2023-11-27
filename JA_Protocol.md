

# JA protocol

## Instruction Packet
Instruction Packet is the command packet sent to the Device.
|Header 1|Header 2|Header 3|Reserved|Packet ID|Length 1|Length 2|Instruction|Param |Param|Param|CRC 1|CRC2|
|--|--|--|--|--|--|--|--|--|--|--|--|--|
|0xFF|0xFF|0xFD|0x00|ID|Len_L|Len_H|Instruction|Param 1 | ...|Param N|CRC_L|CRC_H|

### Header
The field that indicates the start of the Packet
### Reserved
Uses 0X00 (Note that Reserved does not use 0XFD). The Reserved functions the same as  Header.
|Header| ID|Length|Inst|Param|CRC|
|--|--|--|--|--|--
| FF FF FD 00 | 01 |06 00|03|40 00 01| DB 66
### Packet ID
The field that indicates an ID of the device that should receive the Instruction Packet and process it

1.  Range : 0 ~ 252 (0x00 ~ 0xFC), which is a total of 253 numbers that can be used
2.  Broadcast ID : 254 (0xFE), which makes all connected devices execute the Instruction Packet

### Length
The field that indicates the length of packet field.

1.  Devided into low and high bytes in the  Instruction 
2.  The Length indicates the Byte size of Instruction, Parameters and CRC fields

-   `Length = the number of Parameters + 3`
-   Status Packet includes 1 byte length ERROR field’s data.
### Instruction
| Value |            Instructions             |                         Description                          |
| :---: | :---------------------------------: | :----------------------------------------------------------: |
| 0x01  |          Ping          | Instruction that checks whether the Packet has arrived to a device with the same ID as Packet ID |
| 0x02  |          Read       |           Instruction to read data from the Device           |
| 0x03  |         Write   |           Instruction to write data on the Device            |
| 0x06  |     Factory Reset | Instruction that resets the Control Table to its initial factory default settings |
| 0x08  |        Reboot    |               Instruction to reboot the Device               |
| 0x10  |        Clear       |           Instruction to reset certain information           |
| 0x20  |Control Table Backup | Instruction to store current Control Table status data to a Backup area or to restore EEPROM data. |
| 0x55  |    Status(Return)   |           Return packet for the Instruction Packet           |
| 0x8A  |  Fast Sync Read   | For multiple devices, Instruction to read data from the same Address with the same length at once |
| 0x8B  | Fast Sync Write  | For multiple devices, Instruction to write data from the same Address with the same length at once |
### Parameters
1.  As the auxiliary data field for Instruction, its purpose is different for each Instruction.

### CRC-16 (IBM/ANSI)
16bit CRC field which checks if the Packet has been damaged during communication.

1.  Devided into low and high bytes in the  Instruction Packet
2.  Range of CRC calculation: From Header (FF FF FD 00) to Parameteres before CRC field in  Instruction Packet

  - Polynomial : x16 + x15 + x2 + 1 (polynomial representation : 0x8005)
  - Initial Value : 0

#### CRC16 Calculation Code

```c
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

#### CRC Calculation Example
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


## Status Packet
Status Packet is the response packet transmitted from the device to a main controller. Note that it has the same construction as the Instruction Packet except the ERROR field is added.
|Header 1|Header 2|Header 3|Reserved|Packet ID|Length 1|Length 2|Instruction|ERR|Param |Param|Param|CRC 1|CRC2|
|--|--|--|--|--|--|--|--|--|--|--|--|--|--|
|0xFF|0xFF|0xFD|0x00|ID|Len_L|Len_H|Instruction|**Error**|Param 1 | ...|Param N|CRC_L|CRC_H|
### Instruction
Instruction of the Status Packet is designated to 0x55 (Status)
## Instruction Details
Note that given examples use the following abbreviation to provide clear information.

-   Header : H
-   Reserved: RSRV
-   Length: LEN
-   Instruction: INST
-   Error: ERR
-   Param: P
### Ping
#### Description
-   Instruction to check the existence of a Device and basic information   
-   Regardless of the Status Return Level of the Device, the Status Packet is always sent to Ping Instruction.      
#### Packet Parameters
| Satatus Packet | Description|
|--|--|
| Param 1 | Model Number LSB|
| Param 2 | Model Number MSB|
| Param 3 | Firmware Version|
#### Example 

-   ID1(JA40) : For Model Number 4010(0x0FAA), Version of Firmware 01(0x01)   
-   Instruction Packet ID : 1

**Ping Instruction Packet**

|H1|H2 |H3 |RSRV |Packet ID |LEN1|LEN2|INST|CRC 1 |CRC 2 |
|--|--|--|--|--|--| --|--|--|--|
| 0xFF | 0xFF |0xFD|0x00|0x01|0x03|0x00|0x01|0x19|0x4E|

**ID 1 Status Packet**
|H1|H2 |H3 |RSRV |Packet ID |LEN1|LEN2|INST|ERR|P1|P2|P3|CRC 1 |CRC 2 |
|--|--|--|--|--|--| --|--|--|--|--|--|--|--|
| 0xFF | 0xFF |0xFD|0x00|0x01|0x07|0x00|0x55|0x00|0xAA|0x0F|0x01|0xC4|0xEF|
### Read
#### Description
-   Instruction to read a value from Control Table 
-   Read Instruction does not respond to Broadcast ID(254 (0xFE))
#### Packet Parameters
| Instruction Packet | Description|
|--|--|
| Param 1 | Low-order byte from the starting address|
| Param 2 | High-order byte from the starting address|
| Param 3 | Low-order byte from the data length (X)|
| Param 4 | High-order byte from the data length (X)|


| Status Packet | Description|
|--|--|
| Param 1 | First Byte|
| Param 2 | Second Byte|
| ...| ...|
| Param X | X-th Byte|


#### Example 

-   ID1(JA40) : Present Position(580, 0x0244, 4[byte]) = 32765(0x00007FFD)


**Read Instruction Packet**

|H1|H2 |H3 |RSRV |Packet ID |LEN1|LEN2|INST|P1|P2|P3|P4|CRC 1 |CRC 2 |
|--|--|--|--|--|--| --|--|--|--|--|--|--|--|
| 0xFF | 0xFF |0xFD|0x00|0x01|0x07|0x00|0x02|0x44|0x02|0x04|0x00|0x14|0x95|

**ID 1 Status Packet**
|H1|H2 |H3 |RSRV |Packet ID |LEN1|LEN2|INST|ERR|P1|P2|P3|P4|CRC 1 |CRC 2 |
|--|--|--|--|--|--| --|--|--|--|--|--|--|--|--|
| 0xFF | 0xFF |0xFD|0x00|0x01|0x08|0x00|0x55|0x00|0xFD|0x7F|0x00|0x00|0x9B|0x9A|
### Write
#### Description
-   Instruction to write a value on the Control Table 
#### Packet Parameters
| Instruction Packet | Description|
|--|--|
| Param 1 | Low-order byte from the starting address|
| Param 2 | High-order byte from the starting address|
| Param 2+1 | First Byte|
| Param 2+2 | Second Byte|
| ...| ...|
| Param 2+X | X-th Byte|

#### Example 

-   ID1(JA40) : Write 2000(0x000007D0) to Goal Position(564, 0x0234, 4[byte])

**Write Instruction Packet**

|H1|H2 |H3 |RSRV |Packet ID |LEN1|LEN2|INST|P1|P2|P3|P4|P5|P6|CRC 1 |CRC 2 |
|--|--|--|--|--|--| --|--|--|--|--|--|--|--|--|--|
| 0xFF | 0xFF |0xFD|0x00|0x01|0x09|0x00|0x03|0x34|0x02|0xD0|0x07|0x00|0x00|0x1E|0xC9|

**ID 1 Status Packet**
|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR  | CRC 1 | CRC 2 |
|:----:|:----:|:----:|:----:|:---------:|:----:|:----:|:----:|:----:|:------|------:|
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01    | 0x04 | 0x00 | 0x55 | 0x00 | 0xA1  |  0x0C |

### Factory Reset
#### Description
-   Instruction that resets the Control Table to its initial factory default settings.   
-   When Factory Reset (0x06) Instruction is performed, a device is rebooted and the LED blinks four times in a row.  
-   In case of when **Packet ID** is a Broadcast ID `0xFE` and **Option** is Reset All `0xFF`, Factory Reset Instruction(0x06) will **NOT** be activated.
#### Packet Parameters
| Instruction Packet | Description|
|--|--|
| Param 1 |   0xFF : Reset all  0x01 : Reset all except ID  0x02 : Reset all except ID and Baudrate|


#### Example 

-   ID1(JA40) : Apply reset with option 0x01(Reset all except ID)

**Factory reset Instruction Packet**
|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST |  P1  | CRC 1 | CRC 2 |
|:----:|:----:|:----:|:----:|:---------:|:----:|:----:|:----:|:----:|:-----:|:-----:|
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01    | 0x04 | 0x00 | 0x06 | 0x01 | 0xA1  | 0xE6  |

**ID 1 Status Packet**
|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR  | CRC 1 | CRC 2 |
|:----:|:----:|:----:|:----:|:---------:|:----:|:----:|:----:|:----:|:-----:|:-----:|
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01    | 0x04 | 0x00 | 0x55 | 0x00 | 0xA1  | 0x0C  |

### Reboot
#### Description
-   Instruction to reboot the device
#### Example 

-   ID1(JA40) 

**Reboot Instruction Packet**
|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | CRC 1 | CRC 2 |
|:----:|:----:|:----:|:----:|:---------:|:----:|:----:|:----:|:-----:|:-----:|
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01    | 0x03 | 0x00 | 0x08 | 0x2F  | 0x4E  |

**ID 1 Status Packet**
|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR  | CRC 1 | CRC 2 |
|:----:|:----:|:----:|:----:|:---------:|:----:|:----:|:----:|:----:|:-----:|:-----:|
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01    | 0x04 | 0x00 | 0x55 | 0x00 | 0xA1  | 0x0C  |

### Clear
#### Description
-   Instruction to clear the fault.
#### Packet Parameters
- None

#### Example 

-   ID1(JA40) 

**Clear Instruction Packet**
|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 |   INST   | CRC 1 | CRC 2 |
|:----:|:----:|:----:|:----:|:---------:|:----:|:----:|:--------:|:-----:|:-----:|
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01    | 0x08 | 0x00 | **0x10** | 0xE0  | 0xCE  |


**ID 1 Status Packet**
|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR  | CRC 1 | CRC 2 |
|:----:|:----:|:----:|:----:|:---------:|:----:|:----:|:----:|:----:|:-----:|:-----:|
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01    | 0x04 | 0x00 | 0x55 | 0x00 | 0xA1  | 0x0C  |

### Control Table Backup
#### Description
-   Instruction to store current Control Table status data to a Backup area, or to restore EEPROM data.    
-   The Control Table Backup works properly only if **Torque Enable** in RAM area is set as '0' (Torque Off status).    
-   Available items in Control Table for data backup:
    
    -   All Data in EERPOM
        
    -   Velocity P.I Gains
        
    -   Position P.I.D Gains
        
    -   Feedforward 1st & 2nd Gains

#### Packet Parameters
| P1 | P2-P5|Description|
|--|--|--|
| 0x01 | 0x43 0x54 0x52 0x4C|Store current Control Table status data to a Backup area  |
#### Example 

-   ID1(JA40) :Backup the Control Table.

**Backup Instruction Packet**

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST |  P1  |  P2  |  P3  |  P4  |  P5  | CRC1 | CRC2 |
|:----:|:----:|:----:|:----:|:---------:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01    | 0x08 | 0x00 | 0x20 | 0x01 | 0x43 | 0x54 | 0x52 | 0x4C | 0x16 | 0xF5 |

**ID 1 Status Packet**
|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR  | CRC1 | CRC2 |
|:----:|:----:|:----:|:----:|:---------:|:----:|:----:|:----:|:----:|:----:|:----:|
| 0xFF | 0xFF | 0xFD | 0x00 |   0x01    | 0x04 | 0x00 | 0x55 | 0x00 | 0xA1 | 0x0C |

### Fast Sync Read
#### Description    
-   Instruction to read data from multiple devices simultaneously using one Instruction Packet
   
-   The Address and Data Length of the data must all be the same.
    
-   If the Address of the data is not continual, an Indirect Address can be used.
    
-   Status Packet will be returned in order, according to input ID in the Instruction Packet.
    
-   Packet ID field : 0xFE (Broadcast ID)
#### Packet Parameters
| Instruction Packet | Description                               |
|:------------------:|:------------------------------------------|
|    Parameter 1     | Low-order byte from the starting address  |
|    Parameter 2     | High-order byte from the starting address |
|    Parameter 3     | Low-order byte from the data length(X)    |
|    Parameter 4     | High-order byte from the data length(X)   |
|   Parameter 4+1    | `1st Device` ID                           |
|   Parameter 4+2    | `2nd Device` ID                           |
|         ⋯          | ⋯                                         |
|   Parameter 4+n    | `Nnd Device` ID                           |

|  Status Packet   | Description                           |
|:----------------:|:--------------------------------------|
|   Parameter 1    | `1st Device` ID                       |
|   Parameter 2    | `1st Device` First Byte               |
|   Parameter 3    | `1st Device` Second Byte              |
|        ⋯         | ⋯                                     |
|   Parameter X    | `1st Device` X-th Byte                |
|  Parameter X+1   | `1st Device` Low-order byte from CRC  |
|  Parameter X+2   | `1st Device` High-order byte from CRC |
|  Parameter X+3   | `2nd Device` Error                    |
|  Parameter X+4   | `2nd Device` ID                       |
| Parameter X+4+1  | `2nd Device` First Byte               |
| Parameter X+4+2  | `2nd Device` Second Byte              |
|        ⋯         | ⋯                                     |
|  Parameter 2X+4  | `2nd Device` X-th Byte                |
| Parameter 2X+4+1 | `2nd Device` Low-order byte from CRC  |
| Parameter 2X+4+2 | `2nd Device` High-order byte from CRC |
|        ⋯         | ⋯                                     |
|  Parameter nX+4  | `Nnd Device` X-th Byte                |


#### Example 

-   ID1(JA40) : Present Position(580, 0x0244, 4[byte]) = 2000(0x000007D0)
-   ID2(JA40) : Present Position(580, 0x0244, 4[byte]) = 9(0x00000009)

**Fast Sync Read Instruction Packet**

|H1|H2 |H3 |RSRV |Packet ID |LEN1|LEN2|INST|P1|P2|P3|P4|P5|P6|CRC 1 |CRC 2 |
|--|--|--|--|--|--| --|--|--|--|--|--|--|--|--|--|
| 0xFF| 0xFF|0xFD|0x00|0xFE|0x09|0x00|0x8A|0x44|0x02|0x04|0x00|0x01|0x02|0x72|0xF2|

**ID1 Status Packet**
|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR  | ID1  |  D1  |  D2  |  D3  |  D4  | CRC1 | CRC2 |
|:----:|:----:|:----:|:----:|:---------:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
| 0xFF | 0xFF | 0xFD | 0x00 |   0xFE    | 0x11 | 0x00 | 0x55 | 0x00 | 0x01 | 0xD0 | 0x07 | 0x00 | 0x00 | 0x0C | 0x83 |

**ID2 Status Packet**
| ERR  | ID2  |  D1  |  D2  |  D3  |  D4  | CRC1 | CRC2 |
|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
| 0x00 | 0x02 | 0x09 | 0x00 | 0x00 | 0x00 | 0x11 | 0x40 |

### Fast Sync Write
#### Description    
-   Instruction to write data from multiple devices simultaneously using one Instruction Packet
   
-   The Address and Data Length of the data must all be the same.
        
-   Status Packet will be returned in order, according to input ID in the Instruction Packet.
    
-   Packet ID field : 0xFE (Broadcast ID)
#### Packet Parameters
| Instruction Packet | Description                               |
|:------------------:|:------------------------------------------|
|    Parameter 1     | Low-order byte from the starting address  |
|    Parameter 2     | High-order byte from the starting address |
|    Parameter 3     | Low-order byte from the data length(X)    |
|    Parameter 4     | High-order byte from the data length(X)   |
|    Parameter 5     | [1st Device] ID                           |
|   Parameter 5+1    | [1st Device] 1st Byte                     |
|   Parameter 5+2    | [1st Device] 2nd Byte                     |
|        ...         | [1st Device]...                           |
|   Parameter 5+X    | [1st Device] X-th Byte                    |
|    Parameter 6     | [2nd Device] ID                           |
|   Parameter 6+1    | [2nd Device] 1st Byte                     |
|   Parameter 6+2    | [2nd Device] 2nd Byte                     |
|        ...         | [2nd Device]...                           |
|   Parameter 6+X    | [2nd Device] X-th Byte                    |
|        ...         | ...                                       |

|  Status Packet   | Description                           |
|:----------------:|:--------------------------------------|
|   Parameter 1    | `1st Device` ID                       |
|  Parameter 2   | `1st Device` Low-order byte from CRC  |
|  Parameter 3   | `1st Device` High-order byte from CRC |
|  Parameter 4   | `2nd Device` Error                    |
|  Parameter 5   | `2nd Device` ID                       |
| Parameter 6 | `2nd Device` Low-order byte from CRC  |
| Parameter 7 | `2nd Device` High-order byte from CRC |
|        ⋯         | ⋯                                     |
| Parameter 4N-2 | `Nnd Device` Low-order byte from CRC  |
| Parameter 4N-1 | `Nnd Device` High-order byte from CRC |

#### Example

-   ID1(JA40) : Write 150(0x00000096) to Goal Position(116, 0x0074, 4[byte])
-   ID2(JA40) : Write 170(0x000000AA) to Goal Position(116, 0x0074, 4[byte])

**Fast Sync Write Instruction Packet**

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST |  P1  |  P2  |  P3  |  P4  |
|:----:|:----:|:----:|:----:|:---------:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
| 0xFF | 0xFF | 0xFD | 0x00 |   0xFE    | 0x11 | 0x00 | 0x8B | 0x74 | 0x00 | 0x04 | 0x00 |

|  P5  |  P6  |  P7  |  P8  |  P9  | P10  | P11  | P12  | P13  | P14  | CRC 1 | CRC 2 |
|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|:-----:|:-----:|
| 0x01 | 0x96 | 0x00 | 0x00 | 0x00 | 0x02 | 0xAA | 0x00 | 0x00 | 0x00 | 0xB2  | 0x8F  |


**Status Packet**
#### ID 1 Status Packet

|  H1  |  H2  |  H3  | RSRV | Packet ID | LEN1 | LEN2 | INST | ERR  | ID1  | CRC1 | CRC2 |
|:----:|:----:|:----:|:----:|:---------:|:----:|:----:|:----:|:----:|:----:|:----:|:----:|
| 0xFF | 0xFF | 0xFD | 0x00 |   0xFE    | 0x09 | 0x00 | 0x55 | 0x00 | 0x01 | 0x86 | 0x8B |

#### ID 2 Status Packet

| ERR  | ID2  | CRC1 | CRC2 |
|:----:|:----:|:----:|:----:|
| 0x00 | 0x07 | 0xEB | 0x64 |
