# NetFlow_NEMEA_Toolchain

This README describes a exemplary toolchain to use the bwNetFlow NetFlow v9 exporter together with NEMEA flow analysing tools.

![Toolchain Overview](sec_architecture.png "bwNetFlow NEMEA Overview")

## Installation of All Necessary Tools
The toolchain needs the following tools to work: bwNetFlow NetFlow v9 Exporter, IPFIXcol2 and the NEMEA framework.
Follow the instructions of the respective tool for a proper installation.

 bwNetFlow NetFlow v9 Exporter: https://github.com/bwNetFlow/protobuf_to_netflow_converter
 
 ipfixcol2: https://github.com/CESNET/ipfixcol2
 
 NEMEA framework: https://github.com/CESNET/Nemea-Framework

## Exemplary Toolchain Using NEMEA IP Blacklistfilter
### NetFlow Exporter
To start the NetFlow in a mode that it can pass the NetFlow packets to IPFIXcol2 you must define the destination address (dst_ip) and destination port (dst-port) in the exporter settings file as following:
```
dst_ip: 127.0.0.1
dst_port: 2055
```
The exporter can then be started as follows:
```
sudo ./main config.ini
```
### IPFIXcol2
The IPFIXcol2 collector must be modified in a way that it accepts NetFlow packets on dst_ip:dst_port. This can be exemplarily done as follows
```
<ipfixcol2>
  <!-- Input plugins -->
  <inputPlugins>
    <input>
      <name>UDP collector</name>
      <plugin>udp</plugin>
      <params>
        <localPort>2055</localPort>
        <localIPAddress></localIPAddress>
      </params>
    </input>
  </inputPlugins>

  <!-- Output plugins -->
  <outputPlugins>
    <output>
      <name>UniRec output</name>
      <plugin>unirec</plugin>
      <params>
        <!-- UniRec template -->
        <uniRecFormat>TIME_FIRST,TIME_LAST,SRC_IP,DST_IP,PROTOCOL,SRC_PORT,DST_PORT,?TCP_FLAGS,PACKETS,BYTES,?LINK_BIT_FIELD,?DIR_BIT_FIELD</uniRecFormat>
        <!-- TRAP interface configuration -->
        <trapIfcCommon>
          <timeout>NO_WAIT</timeout>
          <buffer>true</buffer>
          <autoflush>500000</autoflush>
        </trapIfcCommon>
        <!-- TRAP interface specification -->
        <trapIfcSpec>
          <tcp>
            <port>8000</port>
            <maxClients>64</maxClients>
          </tcp>
        </trapIfcSpec>
      </params>
    </output>
  </outputPlugins>
</ipfixcol2>
```
In this example, the collector exports the IPFIX packets on port 8000. This must be considered in following modules.
### IP Blacklistfilter
The IP blacklistfilter can process packets exported by the IPFIXcol2 collector. The results are then sent to the subsequent logger. Thus, the interface on which the filter expects IPFIX packets and the export interface to the logger must defined.
This can be exemplarily done as follows:
```
/usr/bin/nemea/ipblacklistfilter -i "t:8000,u:ipbl"
```
Where /usr/bin/nemea is the common location where all modules are installed. t:8000 defines the interface on which IPFIX packets are expected. u:ipbl is the exporting socket, where ipbl is a user defined tag, which stands for ip blacklist in this case. 
### logger
NEMEA's logger module only needs a defined interface on which incoming data are expected. This can be done as follows:
```
/usr/bin/nemea/logger -i "u:ipbl"
```
In this example, the logger prints all alerts to stdout. The see logger help page for more advanced options.
