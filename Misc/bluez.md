# bluez 协议栈分析

- hci request请求结构体
```c
struct hci_request {
  uint16_t ogf;	// ogf
  uint16_t ocf;	// ocf
  int   event;	// 事件
  void   *cparam; // 输入参数
  int   clen;		// 输入参数长度
  void   *rparam;	// 输出参数
  int   rlen;		// 输出参数长度
};
```

```c
/* HCI Packet types */
#define HCI_COMMAND_PKT		0x01
#define HCI_ACLDATA_PKT		0x02
#define HCI_SCODATA_PKT		0x03
#define HCI_EVENT_PKT		0x04
#define HCI_VENDOR_PKT		0xff
```
- 设置inquiry mode 0x00
`hciconfig hci0 inqmode 1`

- Inquiry_Mode:
| Value Parameter| Description|
| --- | ---|
|0x00 |Standard Inquiry Result event format|
|0x01| Inquiry Result format with RSSI|
|0x02 |Inquiry Result with RSSI format or Extended Inquiry Result format|
|0x03|-0xFF Reserved|

- 手动下发inquiry 命令
`hcitool cmd 0x01 0x0001 0x33 0x8b 0x9e 0x08 0xff`