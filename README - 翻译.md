# S7commPlusDriver

使用S7-1200/1500 Steuerungen进行数据传输。

## 发展立场

目前这是一个开发状态，不适合生产用途。

目的是开发一种通信驱动程序，允许通过对所谓“优化”区域的符号访问来访问 S7 1200/1500 控制器的变量管理。

该实现完全用 C# 编写。OpenSSL 库用于 TLS 加密。

## 系统要求

### CPU
通信驱动程序**仅**支持具有支持安全通信的固件的 CPU 通过 TLS 协议允许。根据目前的知识：

- S7 1200 mit einer  固件 >= V4.3 (TLS 1.3 ab V4.5)
- S7 1500 mit einer  固件 >= V2.9

重要的是，不仅有一个具有适当固件的 CPU，而且还有开发环境配置了对应的版本。这仅适用于 TIA Portal 版本 >= V17。

### OpenSSL
OpenSSL 用于 TLS 通信。如果 OpenSSL 安装在适当的版本中，那么应该安装相应的版本

必须输入安装目录的系统路径。必要的 dll 也存储在项目中并在构建过程中使用

复制到所需版本（x86 或 x64）的输出目录。

必要的 dll 的文件名取决于所使用的操作系统：

对于32位 (x86)：
- libcrypto-3.dll
- libssl-3.dll

对于64位 (x64)：
- libcrypto-3-x64.dll
- libssl-3-x64.dll

## 经过测试的通讯
到目前为止，以下设备已成功测试：
- S7 1211 mit 固件 V4.5
- TIA Plcsim V17 (mit 网络PLC)
- TIA Plcsim V18 (mit 网络PLC)

## 使用 Wireshark 进行分析
由于加密，在没有进一步信息的情况下，无法再使用 Wireshark 查看传输的数据。

对于驱动程序开发，项目中集成了一个功能，可将协商的机密保存到文本文件中
（key_YYYYMMDD_hhmmss.log）。有了这些信息，Wireshark 就能够解密并显示通信。

重要的是，记录必须包含 TLS 连接建立！

有两种方法可以将此信息提供给 Wireshark：

1. 将日志文件放置在某个目录中并让 Wireshark 知晓。为此，请转至 Wireshark 中的*菜单* → *设置*。
   在*Protocols*下选择*TLS*，并在*(Pre)-Master-Secret log filename*字段中选择相应的文件
2. 将机密直接集成到 Wireshark 记录中

如果您想将其传递给其他人进行分析，则最好选择第 2 点，因为您需要的所有内容都在一个录音中。

集成通过 Wireshark 安装目录中的“editcap.exe”程序进行。

为此，必须进行录音Wireshark 可以使用扩展名 *.pcapng* 保存。

使用命令提示符，使用以下语句将“key.log”中的机密添加到记录“test-capture.pcapng”中。

集成并保存在文件“test-capture-with-keys.pcapng”中。如果随后在 Wireshark 中打开后一个文件，根据协议，通信被解密、解码并显示。

如有必要，可以删除 key.log。

```
"C:\Program Files\Wireshark\editcap.exe" --inject-secrets tls,key.log test-capture.pcapng test-capture-with-keys.pcapng
```

为了方便起见，我编写了一个带有图形界面的小型实用程序，可以将文件拖放到其中。

可以拖动，并且按下按键即可调用 editcap。该程序可在此处获取：

https://github.com/thomas-v2/PcapKeyInjector

为了让Wireshark能够解码S7comm-Plus协议，相应的dll必须放在Wireshark安装目录下。

欲了解更多信息并从 Sourceforge 下载 dll：

https://sourceforge.net/projects/s7commwireshark/

## PlcTag-Klasse: PlcTags 中 PLC 数据类型的实现

对于某些数据类型，为了处理PLC的响应，需要提前知道其类型，以便将其转换为.Net中有意义的数据类型。PlcTag 类就是为此目的而提供的。

下表列出了 PLC (TIA V18) 中当前可能的所有数据类型以及它们所在的数据类型。

以 S7comm Plus 协议在网络上传输，以及由此产生的 PlcTag 类中的 .Net 数据类型。

| Supported | PLC Datentyp              | PLC Kategorie     | PLC Info          | Netzwerk Datentyp             | .Net Datentyp PlcTag          | Sonstiges                                         |
| :-------: | --------------------------| ----------------- | ----------------- | ----------------------------- | ----------------------------- | ------------------------------------------------- |
| &check;   | AOM_IDENT                 | Hardwaredatentypen|                   | ValueDWord                    | PlcTagDWord -> uint           |                                                   |
| &check;   | Any                       | Zeiger            | Parameter         | ValueUSIntArray[10]           | byte[10]                      |                                                   |
| &check;   | Array[n..m]               |                   |                   |                               |                               | Zugriff auf Einzelelemente direkt möglich         |
| &check;   | Block_FB                  | Parametertypen    | Parameter         | ValueUInt                     | PlcTagUInt -> ushort          |                                                   |
| &check;   | Block_FC                  | Parametertypen    | Parameter         | ValueUInt                     | PlcTagUInt -> ushort          |                                                   |
| &check;   | Bool                      | Binärzahlen       |                   | ValueBool                     | bool                          |                                                   |
| &check;   | Byte                      | Bitfolgen         |                   | ValueByte                     | byte                          |                                                   |
| &check;   | CONN_ANY                  | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | CONN_OUC                  | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | CONN_PRG                  | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | CONN_R_ID                 | Hardwaredatentypen|                   | ValueDWord                    | PlcTagDWord -> uint           |                                                   |
| &check;   | CREF                      | Systemdatentypen  |                   | ValueStruct / packed          |                               | Zugriff auf Einzelelemente direkt möglich         |
| &check;   | Char                      | Zeichenfolgen     |                   | ValueUSInt                    | char                          | Encoding Voreinstellung ISO-8859-1 für non-ASCII  |
| &check;   | Counter                   | Parametertypen    | Parameter         | ValueUInt                     | PlcTagUInt -> ushort          |                                                   |
| &check;   | Date                      | Datum und Uhrzeit |                   | ValueUInt                     | DateTime                      | TODO: Nur Datum gültig!                           |
| &check;   | Date_And_Time             | Datum und Uhrzeit |                   | ValueUSIntArray[8]            | DateTime                      |                                                   |
| &check;   | DB_ANY                    | Hardwaredatentypen|                   | ValueUInt                     | PlcTagUInt -> ushort          |                                                   |
| &check;   | DB_DYN                    | Hardwaredatentypen|                   | ValueUInt                     | PlcTagUInt -> ushort          |                                                   |
| &check;   | DB_WWW                    | Hardwaredatentypen|                   | ValueUInt                     | PlcTagUInt -> ushort          |                                                   |
| &check;   | DInt                      | Ganzzahlen        |                   | ValueDInt                     | int                           |                                                   |
| &check;   | DTL                       | Datum und Uhrzeit |                   | ValueStruct / packed          | DateTime + uint (for ns)      | Nanosekunden extern, da kein .Net Typ mit ns. Experimental!                                     |
| &check;   | DWord                     | Bitfolgen         |                   | ValueDWord                    | uint                          |                                                   |
| &check;   | EVENT_ANY                 | Hardwaredatentypen|                   | ValueDWord                    | PlcTagDWord -> uint           |                                                   |
| &check;   | EVENT_ATT                 | Hardwaredatentypen|                   | ValueDWord                    | PlcTagDWord -> uint           |                                                   |
| &check;   | EVENT_HWINT               | Hardwaredatentypen|                   | ValueDWord                    | PlcTagDWord -> uint           |                                                   |
| &check;   | ErrorStruct               |                   |                   | ValueStruct / packed          |                               | Zugriff auf Einzelelemente direkt möglich         |
| &check;   | HW_ANY                    | Hardwaredatentypen|                   | ValueWord                     |                               |                                                   |
| &check;   | HW_DEVICE                 | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | HW_DPMASTER               | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | HW_DPSLAVE                | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | HW_HSC                    | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | HW_IEPORT                 | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | HW_INTERFACE              | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | HW_IO                     | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | HW_IOSYSTEM               | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | HW_MODULE                 | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | HW_PTO                    | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | HW_PWM                    | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | HW_SUBMODULE              | Hardwaredatentypen|                   | ValueWord                     | PlcTagWord -> ushort          |                                                   |
| &check;   | IEC_COUNTER               | Systemdatentypen  |                   | ValueStruct / packed          |                               | 33554462, Zugriff auf Einzelelemente direkt möglich |
| &check;   | IEC_DCOUNTER              | Systemdatentypen  |                   | ValueStruct / packed          |                               | Zugriff auf Einzelelemente direkt möglich         |
| &check;   | IEC_LCOUNTER              | Systemdatentypen  |                   | ValueStruct / packed          |                               | Zugriff auf Einzelelemente direkt möglich         |
| &check;   | IEC_LTIMER                | Systemdatentypen  |                   | ValueStruct / packed          |                               | Zugriff auf Einzelelemente direkt möglich         |
| &check;   | IEC_SCOUNTER              | Systemdatentypen  |                   | ValueStruct / packed          |                               | Zugriff auf Einzelelemente direkt möglich         |
| &check;   | IEC_TIMER                 | Systemdatentypen  |                   | ValueStruct / packed          |                               | 33554463, Zugriff auf Einzelelemente direkt möglich |
| &check;   | IEC_UCOUNTER              | Systemdatentypen  |                   | ValueStruct / packed          |                               | Zugriff auf Einzelelemente direkt möglich         |
| &check;   | IEC_UDCOUNTER             | Systemdatentypen  |                   | ValueStruct / packed          |                               | Zugriff auf Einzelelemente direkt möglich         |
| &check;   | IEC_ULCOUNTER             | Systemdatentypen  |                   | ValueStruct / packed          |                               | Zugriff auf Einzelelemente direkt möglich         |
| &check;   | IEC_USCOUNTER             | Systemdatentypen  |                   | ValueStruct / packed          |                               | Zugriff auf Einzelelemente direkt möglich         |
| &check;   | Int                       | Ganzzahlen        |                   | ValueInt                      | short                         |                                                   |
| &check;   | LDT                       | Datum und Uhrzeit |                   | ValueTimestamp                | ulong                         |                                                   |
| &check;   | LInt                      | Ganzzahlen        |                   | ValueLInt                     | long                          |                                                   |
| &check;   | LReal                     | Gleitpunktzahlen  |                   | ValueLReal                    | double                        |                                                   |
| &check;   | LTime                     | Zeiten            |                   | ValueTimespan                 | long                          | Anzahl ns                                         |
| &check;   | LTime_Of_Day (LTOD)       | Datum und Uhrzeit |                   | ValueULInt                    | ulong                         | Anzahl ns seit 00:00:00 Uhr                       |
| &check;   | LWord                     | Bitfolgen         |                   | ValueLWord                    | ulong                         |                                                   |
| &check;   | NREF                      | Systemdatentypen  |                   | ValueStruct / packed          |                               | Zugriff auf Einzelelemente direkt möglich         |
| &check;   | OB_ANY                    | Hardwaredatentypen|                   | ValueInt                      | PlcTagInt -> short            |                                                   |
| &check;   | OB_ATT                    | Hardwaredatentypen|                   | ValueInt                      | PlcTagInt -> short            |                                                   |
| &check;   | OB_CYCLIC                 | Hardwaredatentypen|                   | ValueInt                      | PlcTagInt -> short            |                                                   |
| &check;   | OB_DELAY                  | Hardwaredatentypen|                   | ValueInt                      | PlcTagInt -> short            |                                                   |
| &check;   | OB_DIAG                   | Hardwaredatentypen|                   | ValueInt                      | PlcTagInt -> short            |                                                   |
| &check;   | OB_HWINT                  | Hardwaredatentypen|                   | ValueInt                      | PlcTagInt -> short            |                                                   |
| &check;   | OB_PCYCLE                 | Hardwaredatentypen|                   | ValueInt                      | PlcTagInt -> short            |                                                   |
| &check;   | OB_STARTUP                | Hardwaredatentypen|                   | ValueInt                      | PlcTagInt -> short            |                                                   |
| &check;   | OB_TIMEERROR              | Hardwaredatentypen|                   | ValueInt                      | PlcTagInt -> short            |                                                   |
| &check;   | OB_TOD                    | Hardwaredatentypen|                   | ValueInt                      | PlcTagInt -> short            |                                                   |
| &check;   | PIP                       | Hardwaredatentypen|                   | ValueUInt                     | PlcTagUInt -> ushort          |                                                   |
| &check;   | Pointer                   | Zeiger            | Parameter         | ValueUSIntArray[6]            | byte[6]                       |                                                   |
| &check;   | PORT                      | Hardwaredatentypen|                   | ValueUInt                     | PlcTagUInt -> ushort          |                                                   |
| &check;   | RTM                       | Hardwaredatentypen|                   | ValueUInt                     | PlcTagUInt -> ushort          |                                                   |
| &check;   | Real                      | Gleitpunktzahlen  |                   | ValueReal                     | float                         |                                                   |
| &check;   | Remote                    | Zeiger            | Parameter         | ValueUSIntArray[10]           | PlcTagAny -> byte[10]         | Identisch zu Any-Pointer                          |
| &check;   | S5Time                    | Zeiten            |                   | ValueWord                     | ushort, ushort                | TODO: TimeBase, TimeValue. Vereinheitlichen?      |
| &check;   | SInt                      | Ganzzahlen        |                   | ValueSInt                     | sbyte                         |                                                   |
| &check;   | String                    | Zeichenfolgen     |                   | ValueUSIntArray[stringlen + 2]| string                        | Encoding Voreinstellung ISO-8859-1 für non-ASCII  |
| &check;   | Struct                    |                   |                   |                               |                               | Zugriff auf Einzelelemente direkt möglich         |
| &check;   | Time                      | Zeiten            |                   | ValueDInt                     | int                           | Anzahl ms mit Vorzeichen                          |
| &check;   | Time_Of_Day (TOD)         | Datum und Uhrzeit |                   | ValueUDInt                    | uint                          | Anzahl ms seit 00:00:00 Uhr                       |
| &check;   | Timer                     | Parametertypen    | Parameter         | ValueUInt                     | PlcTagUInt -> ushort          |                                                   |
| &check;   | UDInt                     | Ganzzahlen        |                   | ValueUDInt                    | uint                          |                                                   |
| &check;   | UInt                      | Ganzzahlen        |                   | ValueUInt                     | ushort                        |                                                   |
| &check;   | ULInt                     | Ganzzahlen        |                   | ValueULInt                    | ulong                         |                                                   |
| &check;   | USInt                     | Ganzzahlen        |                   | ValueUSInt                    | byte                          |                                                   |
| &cross;   | Variant                   | Zeiger            | Parameter         |                               |                               | Erhält keine Adresse                              |
| &check;   | WChar                     | Zeichenfolgen     |                   | ValueUInt                     | char                          |                                                   |
| &check;   | WString                   | Zeichenfolgen     |                   | ValueUIntArray[stringlen + 2] | string                        |                                                   |
| &check;   | Word                      | Bitfolgen         |                   | ValueWord                     | ushort                        |                                                   |

## 执照

除非另有说明，GNU 较宽通用公共许可证适用于所有源代码，
版本 3 或更高版本。

## 作者

* **Thomas Wiens** - *Initial work* - [thomas-v2](https://github.com/thomas-v2)
