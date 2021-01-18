# Hierarchy Storage System for Convolution Operation

## AXI-4 Protocol Learning
###  AXI-4 Handshaking

According to the AXI handshaking rule, device that sends the address and control command actively is called ***Master***, device that receives the address and control command passively is called ***Slave***. When handshaking is building Master will set the ***VALID*** signal to high to show that Master device is ready to send the data, Slave will set the ***READY*** signal to high to show that slave device is ready to receive data. At each the posedge of the ***ACLK***  both device will check the status of these two signals. And only when both VALID and READY are set to high will the information switch begin.

AXI-4 has 5 independent channels. Each of the channels follows the AXI handshaking rule. Which means all of the 5 independent channels have their own VALID and READY signal.

| Address Write | Address Read | Write | Read | Respond |
| :-----------: | :----------: | :---: | :--: | :-----: |
| *AW*          | *AR*         | *W*   | *R*  | *B*     |

Whether VALID is set to high depends entirely on the Master. After Master has finish handling the previous task (while the whole transmission process still ongoing) the VALID signal will set to high no matter the slave is ready or not. Same as Master, soon as the Slave is ready to receive data will the READY signal be set to high. Under this rule, the time of these two signals' arrival will have three different situations.

#### 1. VALID before READY handshake

![VALID Before READY when handshake](https://s3.ax1x.com/2021/01/18/s6tGy4.png)

- At **T1**, **DATA** will not be processed by the Master nor the Slave, the status of the **DATA** bus will not causing any effect on the upcoming transfer.


- At **T2**, Master is ready to transfer the data so the **VALID** signal is set to high, the **DATA** is updated, and hold on the bus line. **READY** signal is at low level which means the Slave device is busy. For now, the Transfer is not happening.

  - According to the AXI handshake rule, once the **VALID** is set to high then the Master device can not turn it down actively until the handshake is finished.


- At **T3**, Slave is ready to receive so the **READY** signal is set to high, the information hold on the **DATA** bus begins to get processed. For now, the handshake between the Master and the Slave is built. The Transfer begins.

#### 2. READY before VALID handshake

![READY before READY handshake](https://s3.ax1x.com/2021/01/18/s6cWrt.png)

- At **T1**, **DATA** will not be processed by the Master nor the Slave, the status of the **DATA** bus will not causing any effect on the upcoming transfer.


- At **T2**, Slave is ready to receive so the **READY** signal is set to high. However, the Slave detects no **VALID** signal is at high level. In general AXI-4 interface usage occasion, there will be multiple Master devices and Slave devices connected on the network in order to achieve multicast or broadcast communication. According to the AXI handshake rule, the Slave can actively choose the status of the **READY**. For now, the Transfer is not happening.

  - If there are no other Master devices request for handshake (or simply in unicast) the Slave will continue hold the **READY** signal high and keep waiting for the new handshake request.
  - If there are other Master devices request for handshake, soon as the target **VALID** signal is detected the Slave will turn the **READY** signal to low and begin to process other handshake requests. Notice that there seems to be no concept of ***PRIORITY*** during AXI handshaking.


- At **T3**, Master is ready to send the data so the **VALID** signal is set to low, the information hold on the **DATA** bus begins to get processed. For now, the handshake between the Master and the Slave is built. The Transfer begins.

#### 3. READY and VALID arrive at the same time handshake

There seems really hard to see this occasion in real situation since the generation of both signals is relatively independent, but I assume that soon as both devices are detected the certain signal is at high level then the Transfer will begin.
