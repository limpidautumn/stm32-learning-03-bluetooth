# Custom Communication Protocol


- Request Packet  
  - Header *1  
    - 1 byte, fixed 0x0C  
    - 1 byte, total packet length in bytes  
  - Payload *n  
    - 1 byte, position (Red = 0x00, Green = 0x01, Blue = 0x02)  
    - 1 byte, state (On = 0xFF, Off = 0x00)  
  - Trailer *1  
    - 1 byte, fixed 0x0A  
    - 1 byte, checksum (sum of all preceding bytes)  

- Response Packet  
  - Header *1  
    - 1 byte, fixed 0x1C  
    - 1 byte, total packet length in bytes  
  - Payload *n  
    - 1 byte, position (Red = 0x00, Green = 0x01, Blue = 0x02)  
    - 1 byte, state (On = 0xFF, Off = 0x00)  
  - Trailer *1  
    - 1 byte, fixed 0x1A  
    - 1 byte, checksum (sum of all preceding bytes)  
