RDT server & client
==============

 Application implementing reliable data transfer on local interface using UDP. Reliable transfer is achieved via sliding window mechanism.

# Usage
Run as:
```
    ./rdtclient –s source_port –d dest_port
    ./rdtserver –s source_port –d dest_port       
```

# Building
```
make               compile project - release version
make pack          packs all required files to compile this project    
make clean         clean temp compilers files    
make debug         builds in debug mode    
make release       builds in release mode 
```

# Protocol

## Structure of packet
```
       -----------------------------------------------------------------
      | ALIGNMENT  |       DATA TYPE         |    DATA REPRESENTATION   |
       -----------------------------------------------------------------
      |     0      |     unsigned short      |    control checksum      |
      |     2      |      unsigned int       |    sequence number       |    
      |     6      |     unsigned short      |    data length           |
      |     8      |     unsigned short      |    flags                 |
      |    10      |    (unspecified)        |    data                  |
       -----------------------------------------------------------------
```

### Description of fields
  1) Represents 16-bit checksum. Algorithm taken from RFC 1071.
  2) Sequence number is unique identifier for the specification of the transferred data by client or for the acknowledgement ACK/NACK by server.
  3) Data length represents length of the transferred data.
  4) Flags serves for the specification of the packet type.
     There are these flags:
     - ACK  (0x01) - Positive acknowledgement of data with given sequence number.
     - NACK (0x02) - Negative acknowledgement of data with given sequence number.
     - END  (0x04) - There are no other data. Transfer completed.
  5) Data are of an unspecified type so we can send also any binary data.       

## Contact and credits
                             
**Author:**    Radim Loskot  
**gmail.com:** radim.loskot (e-mail)
       