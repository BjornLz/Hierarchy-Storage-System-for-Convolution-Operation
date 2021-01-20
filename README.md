# Hierarchy Storage System for Convolution Operation

## AXI-4 Protocol Learning

- AXI-4 handshaking
  - 1. VALID before READY handshake
  - 2. READY before VALID handshake
  - 3. READY and VALID arrive at the same time handshake

- AXI-4 Transaction
  - 1. Concept of Burst
  - 2. Burst Type
  - 3. Burst Size
  - 4. Burst Length

- AXI-4 Architecture
  - 1. Write Transaction & Read Transaction
  - 2. Signals

Following blogs gives me great help learning AXI-4.


[深入 AXI4 总线（一）握手机制](https://zhuanlan.zhihu.com/p/44766356)

[深入 AXI4 总线（二）架构](https://zhuanlan.zhihu.com/p/45122977)

[深入 AXI4 总线（三）传输事务结构](https://zhuanlan.zhihu.com/p/46538028)

###  AXI-4 Handshaking

According to the AXI handshaking rule, device that sends the address and control command actively is called **Master**, device that receives the address and control command passively is called **Slave**. When handshaking is building Master will set the **``VALID``** signal to high to show that Master device is ready to send the data, Slave will set the **``READY``** signal to high to show that slave device is ready to receive data. At each the posedge of the **``ACLK``**  both device will check the status of these two signals. And only when both VALID and READY are set to high will the information switch begin.

Whether VALID is set to high depends entirely on the Master. After Master has finish handling the previous task (while the whole transmission process still ongoing) the VALID signal will set to high no matter the slave is ready or not. Same as Master, soon as the Slave is ready to receive data will the READY signal be set to high. Under this rule, the time of these two signals' arrival will have three different situations.

#### 1. VALID before READY handshake

![VALID Before READY when handshake](https://s3.ax1x.com/2021/01/18/s6tGy4.png)

- At **T1**, **``DATA``** will not be processed by the Master nor the Slave, the status of the **``DATA``** bus will not causing any effect on the upcoming transfer.


- At **T2**, Master is ready to transfer the data so the **``VALID``** signal is set to high, the **``DATA``** is updated, and hold on the bus line. **``READY``** signal is at low level which means the Slave device is busy. For now, the Transfer is not happening.

  - According to the AXI handshake rule, once the **``VALID``** is set to high then the Master device can not turn it down actively until the handshake is finished.


- At **T3**, Slave is ready to receive so the **``READY``** signal is set to high, the information hold on the **``DATA``** bus begins to get processed. For now, the handshake between the Master and the Slave is built. The Transfer begins.

#### 2. READY before VALID handshake

![READY before READY handshake](https://s3.ax1x.com/2021/01/18/s6cWrt.png)

- At **T1**, **``DATA``** will not be processed by the Master nor the Slave, the status of the **``DATA``** bus will not causing any effect on the upcoming transfer.


- At **T2**, Slave is ready to receive so the **``READY``** signal is set to high. However, the Slave detects no **``VALID``** signal is at high level. In general AXI-4 interface usage occasion, there will be multiple Master devices and Slave devices connected on the network in order to achieve multicast or broadcast communication. According to the AXI handshake rule, the Slave can actively choose the status of the **``READY``**. For now, the Transfer is not happening.

  - If there are no other Master devices request for handshake (or simply in unicast) the Slave will continue hold the **``READY``** signal high and keep waiting for the new handshake request.
  - If there are other Master devices request for handshake, soon as the target **``VALID``** signal is detected the Slave will turn the **``READY``** signal to low and begin to process other handshake requests. Notice that there seems to be no concept of ***PRIORITY*** during AXI handshaking.


- At **T3**, Master is ready to send the data so the **``VALID``** signal is set to low, the information hold on the **``DATA``** bus begins to get processed. For now, the handshake between the Master and the Slave is built. The Transfer begins.

#### 3. READY and VALID arrive at the same time handshake

There seems really hard to see this occasion in real situation since the generation of both signals is relatively independent, but I assume that soon as both devices are detected the certain signal is at high level then the Transfer will begin.

***

### AXI-4 Transaction

#### 1. Concept of Burst

In general transaction situation, take communication between BRAMs as an example, when performing a write operation, a certain memory address should be provided along with the data that need to be store at that address. Like digging a hole at the bottom of a bucket, the water will keep flowing off the bucket until the hole is blocked manually or there is no water left in the bucket. The whole transaction process can be continuously performed before the transaction is closed manually or there is no more data that need to be transferred is left.

Different from the transaction mode mentioned above, ***Burst Transaction*** is not a continuously performed mode. Transaction in AXI-4 is all based on this mode. Let's back on our bucket, this time instead of keeping the hole wide open, we install a tap with a flowmeter. When we want to get some water from the bucket, we just give the flowmeter a number to indicate a certain amount of water and turn on the tap. After the exact amount of water is flowed off, the tap will be closed automatically. After turning on the tap multiple times the water we need will finally get.

Notice that in [AXI-4 Guide provided by ARM](https://developer.arm.com/architectures/system-architectures/amba). The whole data switching process is described by terms below.

- ***AXI Transaction*** : Used to describe the whole data switching process. (*The behave of we get water from the bucket*)

- ***AXI Burst*** : Used to describe several times continuous data switching. (*Open the tap one time*)

- ***AXI Transfer / AXI Beat*** : Used to describe one time of data switching. (*Water flows off the tap*)

- ***Burst Type*** : Used to describe the exact strategy of operating the AXI Burst.

- ***Burst Length*** : Used to describe how much continuous data will be switch in one AXI beat. (*The number we give to the flowmeter*)  

- ***Burst Size*** : Used to describe the size of each Transfer in Burst. (*How wide is the hole*)

The relationship between Transaction, Burst, and Transfer can be roughly described using the following formula.

> AXI Transaction = N * AXI Burst
>
> AXI Burst = Burst Length * AXI Transfer

#### 2. Burst Type
After the handshake is successfully built, the burst will begin. Firstly, a serial of information that indicates the burst type, size, length, and an initial address will be announced to the target Slave. According to the ARM, there are three types of burst that can be applied while using AXI-4 interface.

- **INCR** : In this type, the initial address will be increased by the burst every time a Transfer is finished. And this updated address will be used as new initial address for the next Transfer. This type is also set as default burst type at Xilinx HBM IP.
```Verilog
always @ (posedge M_AXI_ACLK)                                         
	  begin                                                                
	    if (M_AXI_ARESETN == 0 || init_txn_pulse == 1'b1)                                            
	      begin                                                            
	        axi_awaddr <= 'b0;                                             
	      end                                                              
	    else if (M_AXI_AWREADY && axi_awvalid)                             
	      begin                                                            
	        axi_awaddr <= axi_awaddr + burst_size_bytes;                   
	      end                                                              
	    else                                                               
	      axi_awaddr <= axi_awaddr;                                        
      end
// copy from Xilinx AXI-4 interface example ip
```

- **FIXED** : In this type, the initial address will not be updated, the whole Burst will repeatedly access the memory in the same area. This type is usually used on occasions that need to access the FIFO device.
```Verilog
always @ (posedge M_AXI_ACLK)                                         
	  begin                                                                
	    if (M_AXI_ARESETN == 0 || init_txn_pulse == 1'b1)                                            
	      begin                                                            
	        axi_awaddr <= 'b0;                                             
	      end                                                              
	    else if (M_AXI_AWREADY && axi_awvalid)                             
	      begin                                                            
	        axi_awaddr <= axi_awaddr;                   
	      end                                                              
	    else                                                               
	      axi_awaddr <= axi_awaddr;                                        
      end
// copy from Xilinx AXI-4 interface example ip
```

- **WRAP** : In this type, firstly according to the initial address and other parameters the **Wrap Boundary** will be calculated. When initial address is less than the wrap boundary then this type acts the same as the **INCR** type. While the initial address reaches the wrap boundary then it will turn back to the most initial address and keep going.
```Verilog
  assign  axi_wrap_boundary = axi_awaddr + N*burst_size_bytes*(burst_length-1'b1);
```

In actual Transaction, burst type is encoded as follows

| AxBURST | Burst Type |
| :-----: | :--------: |
| 0x00    | FIXED      |
| 0x01    | INCR       |
| 0x10    | WRAP       |
| 0x11    | Reserved   |

#### 3. Burst Size

Burst Size should be in 2^n bytes, otherwise ***Narrow Bursts*** are used. So ARM encoded the burst size as follows in actual Transaction.

| AxSIZE  | Burst Size |
| :-----: | :--------: |
| 0x00    | 1          |
| 0x01    | 2          |
| 0x10    | 4          |
| 0x11    | 8          |

In order to transfer data width into burst size, Xilinx AXI-4 interface example IP uses function ``clog2`` to achieve that.
```Verilog
// function called clogb2 that returns an integer which has the
// value of the ceiling of the log base 2.                      
function integer clogb2 (input integer bit_depth);              
begin                                                           
  for(clogb2=0; bit_depth>0; clogb2=clogb2+1)                   
	 bit_depth = bit_depth >> 1;                                 
end                                                           
endfunction

assign M_AXI_ARSIZE = clogb2((C_M_AXI_DATA_WIDTH/8)-1);
```
Actually, there are three situations after the burst size is set, and each of them has a handling strategy. **Narrow Burst** is one of the strategies used when actual needed bit width is less than the channel width. HBM has extremely wide I/O resource so it's worth a closer look.

When narrow burst happened, not all the bits in the channel are useful. In order to get the exact bits, the useless bits should be masked out. But it would be a huge waste of I/O resources to give each bit a mask bit (especially in situations like AXI HBM). So in AXI-4 there is 1 bit of mask for 8 bits of data (1 byte of data). In the write channel this mask signal is called ``WSTRB``.
```Verilog
//All bursts are complete and aligned in this example
assign M_AXI_WSTRB = {(C_M_AXI_DATA_WIDTH/8){1'b1}};
```  

#### 4. Burst Length

In a Transaction, if the burst length is 0 then it seems to mean that there will be no Transfer happen. In order to prevent this status from causing any misunderstand, ARM defines the actual burst length as the value of ``AxLEN`` plus 1.

In AXI-3, burst length in all three burst type is fixed 16. In AXI-4, **FIXED** and **WRAP** type have a maximum burst length of 16, while in **INCR** the maximum will be 256. Yet don't know how burst length and burst size will effect the Transaction, default setting in Xilinx AXI-4 interface example IP is 16 in burst length and 8 in burst size.

![brust](https://s3.ax1x.com/2021/01/20/sRN6KK.png)

***


###  AXI-4 Architecture

AXI-4 has 5 independent channels. Each of the channels follows the AXI handshaking rule. Which means all of the 5 independent channels have their own VALID and READY signal.

| Address Write | Address Read | Write | Read | Respond |
| :-----------: | :----------: | :---: | :--: | :-----: |
| *AW*          | *AR*         | *W*   | *R*  | *B*     |

Notice that Respond channel is used to do response only to write operation, read respond is finished during read operation.

#### 1. Write Transaction & Read Transaction

Before the write transaction happens, `AW` and `W` channels will try to handshake with the `AW` and `W` channels in the target Slave device. After the handshake is successfully created the most initial address and a serial of parameters that describe how the upcoming burst will be like will be sent through `AW`. Then burst will begins, data and write strobe signal will be sent through `W`. After the burst is finished a respond to the whole burst (Not to some specific Transfer) will be sent back through `B`.


According to the AXI-4 guide, it is possible to send the initial address and control signals in the middle of the burst rather than before the burst happens. This project will choose to send those information before the burst to make the whole access process more orderly.


Read transaction works almost the same as the write transaction. Except there is no read strobe signals and no independent respond channel for read transaction. Apart from that, since it is a must to get the initial address and control signals before any output happens if the Slave would output the right data.


#### 2. Signals

There are two global signals in AXI interface, `ACLK` for ***AXI Clock*** and `ARESTn` for ***AXI Global Reset***. Every operation will happens at the posedge of `ACLK`, and `ARESTn` is active low. Interestingly, Xilinx AXI-4 interface example IP have another global signal to initial the Transaction called `m00_axi_init_axi_txn`. (I'm not sure if other AXI-4 design has this signal)

```Verilog
//Generate a pulse to initiate AXI transaction.
	always @(posedge M_AXI_ACLK)										      
	  begin                                                                        
	    // Initiates AXI transaction delay    
	    if (M_AXI_ARESETN == 0 )                                                   
	      begin                                                                    
	        init_txn_ff <= 1'b0;                                                   
	        init_txn_ff2 <= 1'b0;                                                   
	      end                                                                               
	    else                                                                       
	      begin  
	        init_txn_ff <= INIT_AXI_TXN; // .INIT_AXI_TXN(m00_axi_init_axi_txn);
	        init_txn_ff2 <= init_txn_ff;                                                                 
	      end                                                                      
	  end

    assign init_txn_pulse = (!init_txn_ff2) && init_txn_ff;
```

When input a pulse to `m00_axi_init_axi_txn` all the ongoing Trasaction will be initialized.

Here is signal announcement in Xilinx AXI-4 interface example IP.

```Verilog
// Initiate AXI transactions
		input wire  INIT_AXI_TXN,
		// Asserts when transaction is complete
		output wire  TXN_DONE,
		// Asserts when ERROR is detected
		output reg  ERROR,
		// Global Clock Signal.
		input wire  M_AXI_ACLK,
		// Global Reset Singal. This Signal is Active Low
		input wire  M_AXI_ARESETN,
		// Master Interface Write Address ID
		output wire [C_M_AXI_ID_WIDTH-1 : 0] M_AXI_AWID,
		// Master Interface Write Address
		output wire [C_M_AXI_ADDR_WIDTH-1 : 0] M_AXI_AWADDR,
		// Burst length. The burst length gives the exact number of transfers in a burst
		output wire [7 : 0] M_AXI_AWLEN,
		// Burst size. This signal indicates the size of each transfer in the burst
		output wire [2 : 0] M_AXI_AWSIZE,
		// Burst type. The burst type and the size information,
    // determine how the address for each transfer within the burst is calculated.
		output wire [1 : 0] M_AXI_AWBURST,
		// Lock type. Provides additional information about the
    // atomic characteristics of the transfer.
		output wire  M_AXI_AWLOCK,
		// Memory type. This signal indicates how transactions
    // are required to progress through a system.
		output wire [3 : 0] M_AXI_AWCACHE,
		// Protection type. This signal indicates the privilege
    // and security level of the transaction, and whether
    // the transaction is a data access or an instruction access.
		output wire [2 : 0] M_AXI_AWPROT,
		// Quality of Service, QoS identifier sent for each write transaction.
		output wire [3 : 0] M_AXI_AWQOS,
		// Optional User-defined signal in the write address channel.
		output wire [C_M_AXI_AWUSER_WIDTH-1 : 0] M_AXI_AWUSER,
		// Write address valid. This signal indicates that
    // the channel is signaling valid write address and control information.
		output wire  M_AXI_AWVALID,
		// Write address ready. This signal indicates that
    // the slave is ready to accept an address and associated control signals
		input wire  M_AXI_AWREADY,
		// Master Interface Write Data.
		output wire [C_M_AXI_DATA_WIDTH-1 : 0] M_AXI_WDATA,
		// Write strobes. This signal indicates which byte
    // lanes hold valid data. There is one write strobe
    // bit for each eight bits of the write data bus.
		output wire [C_M_AXI_DATA_WIDTH/8-1 : 0] M_AXI_WSTRB,
		// Write last. This signal indicates the last transfer in a write burst.
		output wire  M_AXI_WLAST,
		// Optional User-defined signal in the write data channel.
		output wire [C_M_AXI_WUSER_WIDTH-1 : 0] M_AXI_WUSER,
		// Write valid. This signal indicates that valid write
    // data and strobes are available
		output wire  M_AXI_WVALID,
		// Write ready. This signal indicates that the slave
    // can accept the write data.
		input wire  M_AXI_WREADY,
		// Master Interface Write Response.
		input wire [C_M_AXI_ID_WIDTH-1 : 0] M_AXI_BID,
		// Write response. This signal indicates the status of the write transaction.
		input wire [1 : 0] M_AXI_BRESP,
		// Optional User-defined signal in the write response channel
		input wire [C_M_AXI_BUSER_WIDTH-1 : 0] M_AXI_BUSER,
		// Write response valid. This signal indicates that the
    // channel is signaling a valid write response.
		input wire  M_AXI_BVALID,
		// Response ready. This signal indicates that the master
    // can accept a write response.
		output wire  M_AXI_BREADY,
		// Master Interface Read Address.
		output wire [C_M_AXI_ID_WIDTH-1 : 0] M_AXI_ARID,
		// Read address. This signal indicates the initial
    // address of a read burst transaction.
		output wire [C_M_AXI_ADDR_WIDTH-1 : 0] M_AXI_ARADDR,
		// Burst length. The burst length gives the exact number of transfers in a burst
		output wire [7 : 0] M_AXI_ARLEN,
		// Burst size. This signal indicates the size of each transfer in the burst
		output wire [2 : 0] M_AXI_ARSIZE,
		// Burst type. The burst type and the size information,
    // determine how the address for each transfer within the burst is calculated.
		output wire [1 : 0] M_AXI_ARBURST,
		// Lock type. Provides additional information about the
    // atomic characteristics of the transfer.
		output wire  M_AXI_ARLOCK,
		// Memory type. This signal indicates how transactions
    // are required to progress through a system.
		output wire [3 : 0] M_AXI_ARCACHE,
		// Protection type. This signal indicates the privilege
    // and security level of the transaction, and whether
    // the transaction is a data access or an instruction access.
		output wire [2 : 0] M_AXI_ARPROT,
		// Quality of Service, QoS identifier sent for each read transaction
		output wire [3 : 0] M_AXI_ARQOS,
		// Optional User-defined signal in the read address channel.
		output wire [C_M_AXI_ARUSER_WIDTH-1 : 0] M_AXI_ARUSER,
		// Write address valid. This signal indicates that
    // the channel is signaling valid read address and control information
		output wire  M_AXI_ARVALID,
		// Read address ready. This signal indicates that
    // the slave is ready to accept an address and associated control signals
		input wire  M_AXI_ARREADY,
		// Read ID tag. This signal is the identification tag
    // for the read data group of signals generated by the slave.
		input wire [C_M_AXI_ID_WIDTH-1 : 0] M_AXI_RID,
		// Master Read Data
		input wire [C_M_AXI_DATA_WIDTH-1 : 0] M_AXI_RDATA,
		// Read response. This signal indicates the status of the read transfer
		input wire [1 : 0] M_AXI_RRESP,
		// Read last. This signal indicates the last transfer in a read burst
		input wire  M_AXI_RLAST,
		// Optional User-defined signal in the read address channel.
		input wire [C_M_AXI_RUSER_WIDTH-1 : 0] M_AXI_RUSER,
		// Read valid. This signal indicates that the channel
    // is signaling the required read data.
		input wire  M_AXI_RVALID,
		// Read ready. This signal indicates that the master can
    // accept the read data and response information.
		output wire  M_AXI_RREADY
```
