**中文** | [English](UserManual-MFProg_Lite_E0.en.md)

# MFProg Lite E0 使用说明书

> 一款适用于 ESP32 系列芯片烧录固件的便携设备。

---

## 1. 产品简介

本设备内置 16MB Flash空间，其中约 11MB 可用于存放固件。

### 核心功能
- **完全离线烧录**：预先将固件存入设备，现场直接对 ESP32 目标板进行烧录，无需电脑参与。
- **多项目管理**：支持存放多组不同的固件项目，通过菜单自由切换选择。
- **USB 拖拽传输**：通过 Type-C 线连接电脑，像操作 U 盘一样拷贝固件文件。
- **压缩加速烧录**：支持压缩固件包，由 ESP32 自行解压写入，大幅缩短烧录时间。
- **OTA 升级**：设备自身固件也可通过 USB 拖拽方式轻松升级。

### 支持的ESP和下载方式

- 支持 ESP32 ESP32-C2 ESP32-C3 ESP32-C5 ESP32-C6 ESP32-C61 ESP32-H2 ESP32-H21 ESP32-H4 ESP32-P4 ESP32-S2 ESP32-S3 ESP32-S31 ESP8266， 随着OTA来支持更多的芯片。
- 基于rom api的下载（可用但暂未开放）
- 加载stub后基于stub的下载（更大的单包大小，更快的速度）
- 加载stub后基于stub的压缩固件下载（推送zlib压缩后的固件）
- 烧录前自动识别flash加密，防止加密后二次下载导致的变砖
- **eFuse 烧录（实验性）**：烧录固件的同时可按 `flasher_args.json` 中的配置烧写 eFuse（如加密密钥、禁用下载模式等），详见 3.5 节。

---

## 2. 快速开始

### 2.1 连接目标板

使用杜邦线或专用烧录线，将设备的烧录接口与目标 ESP32 开发板连接：

```
烧录器          目标 ESP32 板
------          ---------------
TX      ----->   RX
RX      ----->   TX
EN      ----->   EN 
BOOT    ----->   GPIO0/9等，具体查看各ESP32的手册
GND     ----->   GND
```

### 2.2 开机

通过 Type-C 线给设备供电，屏幕亮起后显示启动画面，约 2 秒后自动进入主菜单。

### 2.3 主菜单说明

开机后看到的主界面有四个选项：
- **烧录** —— 选择固件项目并开始烧录
- **文件** —— 浏览已存储的固件项目
- **U盘** —— 进入 U 盘模式，连接电脑传输文件
- **设置** —— 调整蜂鸣器、语言、下载波特率等

使用 **【上】** 和 **【下】** 键移动光标，**【确认】** 键进入对应功能。

---

## 3. 导入 ESP 固件

要将电脑上的 ESP32 固件导入到烧录器中，请按以下步骤操作：

### 3.1 进入 U 盘模式

1. 在主菜单选择 **【U盘】**，按 **【确认】** 键进入。
2. 屏幕提示 "U盘模式已开启" 后，用 Type-C 线将设备连接到电脑。
3. 电脑会识别出一个可移动磁盘（与 U 盘使用方式完全相同）。

### 3.2 固件包生成

#### 3.2.1 使用 GUI 工具进行压缩固件包生成（推荐）

如果你使用 ESP-IDF 开发，直接导入build文件夹即可自动识别。gui工具能通过`flasher_args.json`识别所需文件。

![fw-tool-gui-folder](images/fw-tool-gui-folder.png)

如果是别人提供的固件，内不包含`flasher_args.json`，可以通过添加行和删除选中进行手动编辑

![fw-tool-gui-manual](images/fw-tool-gui-manual.png)

然后选择输出文件夹，点击开始压缩即可生成固件包。

![fw-tool-gui-output](images/fw-tool-gui-output.png)

#### 3.2.2 使用 CLI 工具进行压缩固件包生成（CI场景推荐）

```bash
MFProg-ESP-fw-tool -i <input_folder> -o <output_folder>
或
MFProg-ESP-fw-tool -i "[(0x0, bootloader.bin), (0x10000, app.bin)]" -o <output_folder>
```

#### 3.2.3 手动编辑`flasher_args.json`（不推荐）

工具本身可识别标准的idf输出的`flasher_args.json`。拷贝`flasher_args.json`及提到的文件即可，json多余的部分不影响。如需自行编写，一个最简的json示例如下。需要注意，此方式并不会启用压缩。

```json
{
  "flash_files": {
    "0x0": "bootloader/bootloader.bin",
    "0x60000": "esp-at.bin",
    "0x8000": "partition_table/partition-table.bin",
    "0xd000": "ota_data_initial.bin",
    "0x1e000": "at_customize.bin",
    "0x1f000": "customized_partitions/mfg_nvs.bin"
  }
}
```

### 3.3 拷贝生成的文件夹到U盘

一个正确的项目格式应该如下

![file-tree-in-udisk](images/file-tree-in-udisk.png)

非压缩固件json

![normal-project-json](images/normal-project-json.png)

压缩固件json

![compress-project-json](images/compress-project-json.png)

### 3.4 安全退出 U 盘模式

1. 在电脑上点击"弹出/推出 U 盘"。
2. 在设备上长按 **【确认】** 键退出 U 盘模式，屏幕返回主菜单。
3. 现在你可以在 **【文件】** 或 **【烧录】** 菜单中看到刚刚导入的项目了。

### 3.5 eFuse 烧录配置（实验性功能）

> **警告**：eFuse 是一次性可编程（OTP）存储，烧录**不可逆**，错误配置可能导致芯片永久不可用。该功能尚处于实验阶段，请先在可牺牲的开发板上充分验证后再用于生产。

在项目的 `flasher_args.json` 中添加可选的 `efuse` 段，即可在烧录固件的同时烧写 eFuse：

```json
{
  "flash_files": {
    "0x0": "bootloader/bootloader.bin",
    "0x60000": "esp-at.bin"
  },
  "efuse": {
    "chips": ["esp32s3"],
    "burn": [
      {
        "block": "BLOCK_USR_DATA",
        "offset": 0,
        "data": "a1b2c3d4",
        "comment": "device config v3",
        "burn_after_flash": false
      }
    ]
  }
}
```

字段说明：

| 字段 | 必选 | 说明 |
|------|------|------|
| `chips` | 否 | 芯片型号白名单（如 `esp32s3`）；连接到的芯片不在列表中时拒绝烧录，防止拿错工程 |
| `burn` | 是 | 烧录项数组 |
| `block` | 是 | eFuse 块名，与 espefuse 命名一致（如 `BLOCK_USR_DATA`、`BLOCK_KEY0`、`BLOCK0`） |
| `data` | 是 | 十六进制字符串，表示要置 1 的位（OR 语义，不影响已烧的其它位）；长度 ≤ 36 字节且为 4 的倍数 |
| `offset` | 否 | 块内字节偏移（默认 0，须为 4 的倍数） |
| `comment` | 否 | 备注文字，仅用于日志显示 |
| `burn_after_flash` | 是 | `false` = 在写 flash 之前烧录该项；`true` = flash 写完并校验后再烧录 |

使用注意事项：

- **无二次确认**：手动模式选中项目即按配置烧录，请确保工程配置正确；
- **有限幂等**：仅对未被写保护的块成立——要烧的位已全部烧过则自动跳过。注意设备**先检查写保护、再检查是否已烧过**，因此一旦块被 `WR_DIS` 写保护（如下文示例中的密钥块），重复烧录同一工程会因写保护检查而报错。需要对同一批板子重复烧录或升级固件时，请在工程包中去掉整个 `efuse` 段。
- 字节序与 `espefuse burn_block_data` 一致：字节 `[4i..4i+3]` 组成块内第 i 个 32 位字（小端）。
- RS 编码块（ESP32-S2/S3/C3 等新款芯片的非 BLOCK0 块）**不允许向非空块追加数据**；ESP32 的 3/4 编码块按 6 字节分组判定冲突。已被写保护（WR_DIS）的块会被拒绝并报错。
- 烧录顺序由设备强制保证：先非 BLOCK0 块，再 BLOCK0 普通字段，写保护/禁下载等危险字段最后烧。

> **BLOCK0 重要建议**：BLOCK0 的烧录项（key purpose、`RD_DIS`/`WR_DIS` 保护位、`DIS_DOWNLOAD_MODE` 等）**务必设置 `"burn_after_flash": true`**。该顺序仅在单次烧录阶段内有效，无法跨阶段保护：若 BLOCK0 项放在写 flash 之前（`false`），写保护位会让后续针对对应块的烧录直接失败，而 `DIS_DOWNLOAD_MODE` 等禁下载位会切断烧录通道，导致 flash 尚未写入就失去连接。

- 字段名/密钥/key purpose 到位图的换算需在上位机完成（后续由 MFProg-ESP-fw-tool 提供转换），设备只做块级烧录。
- ESP8266 不支持 eFuse 烧录。

#### 示例：烧录主机预生成的 Flash 加密密钥（ESP32-S3）

对应乐鑫官方文档 [flash 加密 - 使用主机生成的密钥](https://docs.espressif.com/projects/esp-idf/zh_CN/stable/esp32/security/flash-encryption.html#pregenerated-flash-encryption-key) 的流程，以 ESP32-S3、AES-128（256 位密钥）为例：

1. 在电脑上生成密钥文件：

   ```bash
   idf.py secure-generate-flash-encryption-key my_flash_encryption_key.bin
   ```

2. 将密钥**按字节倒序**后转成 hex 字符串（XTS 密钥的硬件字节序要求，与 espefuse `burn_key` 的处理一致）：

   ```bash
   $ python3 -c "print(open('my_flash_encryption_key.bin','rb').read()[::-1].hex())"
   dd26da7be1b5420d868969a06954d9343284497f39b7d23c307095b8bad91005
   ```

   （本例使用的原始密钥文件内容为 `0510d9bab89570303cd2b7397f49843234d95469a06989860d42b5e17bda26dd`，仅作演示，请勿用于实际产品。）

3. 在项目的 `flasher_args.json` 中加入 `efuse` 段（将 `data` 替换为上一步的输出，此处为本例的真实演示值）：

   ```json
   {
     "flash_files": {
       "0x0": "bootloader/bootloader.bin",
       "0x8000": "partition_table/partition-table.bin",
       "0x10000": "app.bin"
     },
     "efuse": {
       "chips": ["esp32s3"],
       "burn": [
         {
           "block": "BLOCK_KEY0",
           "data": "dd26da7be1b5420d868969a06954d9343284497f39b7d23c307095b8bad91005",
           "comment": "flash encryption key (XTS_AES_128_KEY)",
           "burn_after_flash": false
         },
         {
           "block": "BLOCK0",
           "data": "000180000100000000000004000000000000000000000000",
           "comment": "KEY_PURPOSE_0=XTS_AES_128_KEY; WR_DIS[8] locks KEY_PURPOSE_0; WR_DIS[23]+RD_DIS[0] protect BLOCK_KEY0",
           "burn_after_flash": true
         }
       ]
     }
   }
   ```

   其中 BLOCK0 项的位图含义（ESP32-S3）：word2 的 `KEY_PURPOSE_0 = 4`（XTS_AES_128_KEY）、word0 bit8 的 `WR_DIS`（写保护 KEY_PURPOSE_0 本身，与官方 espefuse `burn_key` 的行为一致）、word0 bit23 的 `WR_DIS`（写保护 BLOCK_KEY0）、word1 bit0 的 `RD_DIS`（读保护 BLOCK_KEY0）。保护位由设备自动安排在最后烧录。

4. 在 ESP-IDF 工程中通过 `menuconfig` → Security features 启用 "Enable flash encryption on boot"，选择开发或量产模式，重新构建固件。
5. 用本设备正常烧录该工程。首次启动时 bootloader 会对 flash 执行就地加密（大分区可能耗时约一分钟），**期间请勿断电**。

> **本示例是"密钥 + 明文固件"流程**：固件以明文写入 flash，`SPI_BOOT_CRYPT_CNT` 由 bootloader 在首启就地加密时自动置位，因此上面的位图不需要包含它。乐鑫另有"主机预加密密文"流程（用 `espsecure.py` 预加密并预烧 `SPI_BOOT_CRYPT_CNT`），那是面向写片器/JTAG 等原始写入通道的，**请勿将 `espsecure.py` 输出的密文交给本设备烧录**（见下）。
>
> **关于再次烧录**：首启加密完成后，用本设备再次烧录**明文**固件是安全的——设备检测到 flash 加密已启用时会自动切换为加密写入（明文发送、芯片端用已烧密钥加密后写入，与 esptool 行为一致），要求写入地址 16 字节对齐（ESP32 经典款为 32 字节且需 stub 模式），且未烧录 `DIS_DOWNLOAD_MANUAL_ENCRYPT`（已烧则报错拒绝）与 `DIS_DOWNLOAD_MODE` 等禁下载位。**不要**烧录主机预加密的密文：芯片会对收到的数据再加密一次，双重加密会导致固件无法启动。

> **ESP32（经典款）差异**：flash 加密密钥块为 `BLOCK1`（别名 `flash_encryption`），同样需要字节倒序；若芯片的 `CODING_SCHEME` 为 3/4（非出厂默认，较罕见），密钥块有效长度只有 24 字节。

> **注意**：密钥以明文形式存在于设备存储的 JSON 中，请妥善保管项目文件；生产使用建议每台设备单独生成密钥。

---

## 4. 使用设备烧录

### 4.1 选择项目

1. 在主菜单选择 **【文件】**，按 **【确认】** 进入。
2. 设备会列出所有已导入的项目文件夹。
3. 使用 **【上】** / **【下】** 选择你要烧录的项目的`flasher_args.json`（已使用金色标注），按 **【确认】** 确认。
4. 选择后会自动跳转到烧录页面

### 4.2 烧录过程

烧录过程中屏幕会实时显示：
- **阶段状态**：如 "连接中..."、"擦除中..."、"写入中..."、"校验中..."
- **进度条**：显示当前总体烧录进度
- **芯片信息**：识别到的目标 ESP32 芯片型号

此时请保持：
- 设备与目标板的连接线稳固
- 目标板上电正常
- 不要按任何按键中断烧录

### 4.3 烧录结果

- **成功**：屏幕显示 "烧录成功"，蜂鸣器播放提示音（如已开启）。按 **【确认】** 再次烧录，长按退出页面。
- **失败**：屏幕显示错误信息（如 "连接超时"、"校验失败" 等）。可据此检查问题。

### 4.4 机台信号接口

本设备支持 3 路 GPIO 信号与自动化烧录机台（治具）对接，实现免人工干预的自动烧录流程。

#### 输入触发

- `START` 功能等价于设备面板上的 **【确认】** 键。
- 低电平脉冲（下降沿）即可触发一次单击操作，用于机台自动开始烧录。

#### 输出信号时序

机台可通过读取 `STA_FLASH` 和 `STA_RES` 两路输出判断当前烧录状态（IO为开漏输出）：

| 设备状态 | STA_FLASH | STA_RES |
|:---------|:----------------:|:-------------:|
| 空闲/待机 | 释放 | 释放 |
| 烧录中 | 拉低 | 释放 |
| 烧录成功 | 释放 | 拉低 |
| 烧录失败 | 释放 | 释放 |

#### 机台对接示例

![control-signal](images/control-signal.png)

---

## 5. 系统设置

在主菜单选择 **【设置】** 可以调整以下选项：

| 设置项 | 说明 |
|--------|------|
| 蜂鸣器 | 开启/关闭操作提示音 |
| 语言 | 切换界面显示语言（保存后在退出设置页时设备自动重启生效） |
| BLE | 开启/关闭蓝牙功能 |
| 下载波特率 | 设置 ESP 烧录时的下载波特率（115200 / 460800 / 1000000 / 2000000） |
| 自动进入下载页 | 开启后设备开机自动进入烧录页面 |
| BLE PIN | 显示 BLE 连接使用的 4 位 PIN 码（只读） |
| 进入 Bootloader | 重启进入 Bootloader 升级模式（见第 6 章） |

操作方式：
- **【上】** / **【下】** 选择设置项
- **【确认】** 进入编辑模式
- 编辑模式下 **【上】** / **【下】** 切换选项值
- 再次按 **【确认】** 保存，或长按 **【确认】** 返回主菜单

> **说明**：【BLE PIN】为只读项，无需编辑；【进入 Bootloader】为执行项，按 **【确认】** 后设备立即重启进入升级模式。

> **提示**：下载波特率默认值为 **1000000**。若烧录时频繁出现连接或校验错误，可尝试降低波特率（如 460800 或 115200）。

---

## 6. 设备 OTA 升级（本机固件升级）

当需要更新烧录器自身的固件或 UI 资源时，使用 UF2 升级方式。

### 6.1 何时需要升级？
- 官方发布了新功能或问题修复
- 需要更新内置的 UI 图片、字体等资源

### 6.2 进入升级模式

有两种方式进入 Bootloader 升级模式：

**方式一：按键上电**
1. 断开设备电源。
2. 按住 **【确认】（MID）** 键不放。
3. 重新连接电源（插入 Type-C 线）。
4. 屏幕显示 Bootloader 画面后松开按键。

**方式二：通过设置页重启**
1. 在正常开机状态下，于主菜单进入 **【设置】** 页面。
2. 选择 **【进入 Bootloader】**，按 **【确认】** 键。
3. 设备自动重启并进入 Bootloader 模式。

进入 Bootloader 后，设备会被电脑识别为一个名为 **MFProgBL** 的可移动磁盘。

### 6.3 执行升级

1. 将官方提供的 `.uf2` 升级文件直接**拖拽**到 `MFProgBL` 磁盘中。
2. 等待文件复制完成，设备会自动进行烧录。
3. 烧录完成后设备自动重启，进入新版本的系统。

> **重要**：升级过程中请勿断电或拔出 USB 线，否则可能导致设备无法启动。

---

## 7. 常见问题与故障排除

### 7.1 烧录问题

**Q：提示"连接超时"或无法识别芯片？**
- 检查 TX/RX 是否接反（烧录器 TX 应接目标板 RX）。
- 检查 EN、BOOT、GND 连线是否牢固。
- 确认目标板供电正常。
- 在 **【设置】** 中将下载波特率降低至 460800 或 115200 后重试。

**Q：提示"校验失败"？**
- 多为通信不稳定导致，请降低下载波特率、缩短连线后重试。
- 若目标芯片已启用 flash 加密，设备会自动切换为加密写入模式，明文固件的再次烧录仍然安全；仅当芯片已烧录 `DIS_DOWNLOAD_MANUAL_ENCRYPT`（量产禁用加密下载）等保护位时，设备才会拒绝烧录并报错，这属于正常的防变砖保护。

**Q：提示不支持的芯片型号？**
- 确认芯片在支持列表中（见 1. 产品简介）。部分较新的芯片型号需要较新的设备固件，请先按第 6 章升级本机固件。

### 7.2 文件与 U 盘问题

**Q：电脑识别不出 U 盘？**
- 确认已在主菜单进入 **【U盘】** 模式，并使用支持数据传输的 Type-C 线（部分线仅支持充电）。

**Q：拷贝完固件后项目没出现，或文件损坏？**
- 请先在电脑上"弹出"U 盘，再在设备上长按 **【确认】** 退出 U 盘模式，确保数据完整写入后再拔出。

### 7.3 系统与升级问题

**Q：切换语言后界面没有变化？**
- 语言保存后需重启生效。长按 **【确认】** 退出设置页时，设备会自动重启并应用新语言。

**Q：如何进入 Bootloader 升级模式？**
- 按住 **【确认】（MID）** 键上电，或在 **【设置】** 中选择 **【进入 Bootloader】**，详见第 6 章。

**Q：升级中断后设备无法启动？**
- 重新按住 **【确认】（MID）** 键上电进入 Bootloader，将完整的 `.uf2` 升级文件重新拖入 `MFProgBL` 磁盘即可恢复。


