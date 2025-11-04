# 1. 安裝

1. 使用 git 克隆此倉庫：
  ```
cd ~
git clone https://github.com/vvuk/eddy-ng
```
2. 然後運行安裝腳本：
  ```
cd ~/eddy-ng
./install.sh
```
（如果您的 Kalico 或 Klipper 沒有安裝在 /usr/local/bin 中`~/klipper`，請將路徑作為第一個參數提供，例如 /usr/local/bin/Kalico/ 或 /usr/local/bin/Klipper `./install.sh ~/my-klipper`/ 。）

## 更新`eddy-ng`

[](https://github.com/vvuk/eddy-ng/wiki/Installation#updating-eddy-ng)

運行 `git pull`，如果運行的是 Klipper，則再次運行`./install.sh`：

```
cd ~/eddy-ng
git pull
./install.sh # only if Klipper
```

更新後，最好重新建置並重新刷寫固件，因為可能會有更改。
### 更新 Klipper

[](https://github.com/vvuk/eddy-ng/wiki/Installation#updating-klipper)

由於 Klipper 沒有擴充機制，`eddy-ng`安裝過程需要在安裝過程中對 Klipper 檔案進行一些變更。您可能需要`eddy-ng`在更新之前先卸載 Klipper（卸載來源文件，而不是您的配置），然後在更新後重新執行安裝腳本。例如：

```
# uninstall eddy-ng
cd ~/eddy-ng
./install.sh --uninstall
# update Klipper
cd ~/klipper
git pull
# re-install eddy-ng
cd ~/eddy-ng
./install.sh
```
如果您在使用 git 指令更新 Klipper 時遇到問題，可以直接使用下列指令手動清除`eddy-ng`變更：

```
cd ~/klipper
git restore src/Makefile klippy/extras/bed_mesh.py
```

然後更新 Klipper。`install.sh`更新後再次運行。
# 2. 安全须知

與其他許多探頭不同，渦流探頭可以直接讀取感測器數據，而無需上下移動來「觸發」感測器。它有兩個要求：

1. 探頭的線圈必須位於床的上方。
2. 床體必須採用均勻的金屬材料（例如，柔性鋼板中的彈簧鋼）。
3. 如果床體內部有磁鐵，那麼這些磁鐵必須是均勻且磁性較弱的（例如，磁性貼紙，而不是單獨的強磁鐵）。

如果以上任何一項不成立，探頭將無法正確讀取高度，並可能造成損壞。

歸位和類似歸位的操作（`QUAD_GANTRY_LEVEL`，`Z_TILT_ADJUST`）需要調整才能安全執行。
## 安全回家

在執行 Z 軸歸零操作時，探針必須位於工作台上方。請確保您已設定好裝置（例如`safe_z_home`，`homing_override`用於將感測器移至工作台上方的裝置），以便在 Z 軸歸零前完成此操作。如果感測器不在工作台上方，工具頭可能無法停止，造成損壞。

## 網床

`bed_mesh`若掃描高度為 Z=2.0mm（預設的「原點高度」`eddy-ng`），則精確度最高。在`bed_mesh`配置部分，您應該進行設定`horizontal_move_z: 2.0`。

使用渦流探頭時，無需進行回縮和Z軸移動，因為高度讀數是直接進行的。使用渦流探頭進行探測時，上下移動工具頭沒有任何優勢；事實上，這樣做很可能會降低精確度。

## 龍門架調平
### Klipper

為了獲得最精確的結果，應使用探頭在 2.0 毫米處進行調平（這是預設的「初始高度」`eddy-ng`）。在`[quad_gantry_level]`配置部分，您應該進行設定 `horizontal_move_z: 2.0`。

然而，在龍門架嚴重傾斜的情況下，以 2 毫米的高度執行 QGL 操作是危險的，因為傾斜角度可能超過 2 毫米。為了安全地執行 QGL 操作，可以添加一個宏，該宏首先在更高的「安全性」高度執行 QGL 操作，然後再下降到 2 毫米。

```
[quad_gantry_level]
...  # the rest of your quad_gantry_level section
horizontal_move_z: 2.0

[gcode_macro QUAD_GANTRY_LEVEL]
rename_existing: _QUAD_GANTRY_LEVEL
gcode:
    SAVE_GCODE_STATE NAME=STATE_QGL
    BED_MESH_CLEAR
    # If QGL has not been done yet, correct for any major skew first
    # at 8mm height
    {% if not printer.quad_gantry_level.applied %}
    _QUAD_GANTRY_LEVEL horizontal_move_z=8 retry_tolerance=1      
    {% endif %}
    # Complete with a full QGL at the configured height
    _QUAD_GANTRY_LEVEL
    RESTORE_GCODE_STATE NAME=STATE_QGL
```
## Z軸傾斜調節
### Klipper
1. 
與先前設定的值一樣`quad_gantry_level`，此調整將根據該值進行`horizontal_move_z`。如果尚未進行 Z 軸傾斜調整，則使用過低的值可能會導致工具頭在傾斜角度較大時撞擊工作台。以下巨集將先進行粗略的傾斜調整，然後再進行更精確的微調。

```
[z_tilt]
horizontal_move_z: 2.0
...  # the rest of your z_tilt section

[gcode_macro Z_TILT_ADJUST]
rename_existing: _Z_TILT_ADJUST
gcode:
    SAVE_GCODE_STATE NAME=STATE_Z_TILT
    BED_MESH_CLEAR
    {% if not printer.z_tilt.applied %}
    _Z_TILT_ADJUST horizontal_move_z=8 retry_tolerance=1
    {% endif %}
    _Z_TILT_ADJUST horizontal_move_z=2
    RESTORE_GCODE_STATE NAME=STATE_Z_TILT
`
```

# 3. EDDY TAP
> [!warning] 
>  **EDDY TAP电路设计为由本人原创开发,不得用作商业用途,QQ群:715552579
**

## 探針安裝

使用 EDDY TAP 探頭進行敲擊的功能和精度`eddy-ng`對探頭線圈和噴嘴尖端之間的距離非常敏感。最佳效果似乎是將 EDDY TAP 的線圈安裝在噴嘴尖端上方約 5 毫米處。這裡指線圈 PCB 板的底部。

> [!tip] 
> 建议使用原作者为FZ  EDDY TAP设计的固定座 !!

下面列出了一些已知性能良好的支架，但其他許多支架也同樣適用。

## 軟體安裝

警告：安裝程式假定您之前未配置過 Eddy。如果您之前已配置 BTT Eddy 但未安裝 eddy-ng，請確保：

- 刪除/不要包含`eddy.cfg`您可能正在使用的檔案。 BTT 的配置中存在一些無關的宏，這些宏會導致問題。
- 請務必從底部的已儲存變數區域中刪除“`probe_eddy_current`和”部分。`temperature_probe``printer.cfg`
- 您可能需要將先前的 [bed_mesh] 設定（通常在 eddy.cfg 檔案中）遷移到 eddy-ng 中。最好將 [bed_mesh] 設定遷移到 printer.cfg 檔案中，因為它與感測器無關。

1. `eddy-ng`運行倉庫`install.sh`中的腳本進行安裝`eddy-ng`。
2. 重建並刷寫 EDDY TAP（與其連接的 MCU）的 Klipper 韌體。

3. `printer.cfg`透過將以下內容新增至設定檔或已包含的設定檔中來更新配置：

適用於 EDDY TAP（請務必將變數替換為您設定中的對應值）。

```
[mcu EDDY TAP]
canbus_uuid: UUID # 对于 CAN 总线：请将此替换为您的 CAN 总线 UUID

[probe_eddy_ng ldc1612_eddy_coil]
sensor_type: ldc1612_internal_clk #内置晶振
i2c_mcu: EDDY TAP
#i2c_bus: i2c1_PB6_PB7
i2c_software_scl_pin: FZeddy:PA9 # i2c 接口的scl pin
i2c_software_sda_pin: FZeddy:PA8 # i2c 接口的sda pin
#i2c_address: 42 
x_offset: 0      #FZ
y_offset: 17.556 #FZ
tap_mode: wma #wma和butter两种模式
i2c_speed: 400000 #i2c速度

[temperature_sensor EDDY TAP]
sensor_type: Generic 3950
sensor_pin: FZeddy:PB0

[temperature_sensor EDDY TAP_mcu]
sensor_type: temperature_mcu
sensor_mcu: EDDY TAP
min_temp: 10
max_temp: 100

```

# 4.  校準
## 初步驗證

完成刷機和配置後，請確保可以與感測器通訊。運行：

```
PROBE_EDDY_NG_STATUS
```

幾次。如果你看到類似這樣的內容：

```
Last coil value: 3189314.38 (-infmm) raw: 0x4409e8b
  (Not calibrated) status: 0x48 UNREADCONV1 DRDY
```

> [!warning] 
> 如果出現`0xffffffff`類似錯誤，則表示感測器通訊存在問題。請查看 Klippy 日誌以尋找線索，並檢查接線和配置。 

  > [!tip] 
> 初始使用回传类似这种信息为正常现象 。
> 
Last coil value: 0.00 (-infmm) raw: 0x4409e8b
  ERROR:0b1  status: 0x48 UNREADCONV1 DRDY  

## 自動校準

加入宏

```
[gcode_macro PROBE_EDDY_NG_SETUP_AUTO]
gcode:
  BED_MESH_CLEAR
  G28 X Y
  G90
  G1 X{ printer.toolhead.axis_maximum.x/2 } Y{ printer.toolhead.axis_maximum.y/2 } F3000
  PROBE_EDDY_NG_SETUP {rawparams} DRIVE_CURRENT=15 MAX_DC_INCREASE=13
  ```
1. 將床加熱至(实际打件温度)。
2. 開始設定過程：
```
PROBE_EDDY_NG_SETUP_AUTO
```
3. 手動設定 Z 軸.紙片測試。
4. 完成后自动测试电流,储存
5. 
  > [!tip] 
>  不建议手动调电流,自动失败代表存在距离上硬件上的问题!!

# 5.  輕敲

輕敲操作依賴於偵測感測器（即工具頭）何時不再靠近列印平台。這通常是由於噴嘴接觸到列印平台造成的。此外，噴嘴尖端殘留的耗材也會導致Z軸偏移量過大，造成敲擊操作。請確保噴嘴清潔後再進行輕敲操作（巨集命令建議見下文）。

通常情況下，整個印表機都有一定的柔性，可以在初始接觸時吸收一部分力。這意味著，噴嘴實際接觸列印板的時間可能比偵測到接觸的時間稍晚。`tap_adjust_z`由於這種柔性通常不會改變，因此您可以使用該參數為計算出的高度添加一個值。

## 首次點擊

XYZ 歸位後，試試：

```
PROBE_EDDY_NG_TAP
```

這將與您的列印平台接觸—最大接觸距離為-0.250，這是預設目標Z軸偏移量，預設速度為3mm/s；列印平台或噴嘴損壞的風險極低，但如果您擔心，可以進行一些實驗`TARGET_Z=-0.100`。您也可以指定`SAMPLES=1`只進行一次測試。如果測試成功，它將設定Z軸偏移量以及“敲擊偏移量”，後者用於在敲擊後更精確地進行列印平台網格劃分。

如果看到錯誤訊息，例如“`Error during homing probe: Sensor error (ldc status: UNREADCONV1 DRDY`和`Tap failed at xxxx`”，請跳至故障排除部分。

如果您看到「probe completed movement without triggering」的錯誤，這表示步進馬達已向下移動`TARGET_Z`，但探針未觸發。您可能還會看到「成功」的敲擊，但該值可能非常接近您的設定值`TARGET_Z`。在這兩種情況下，請手動將工具頭移動到 TARGET_Z 位置，並檢查它是否實際接觸到建置平台（而不僅僅是輕微接觸），因為它很可能沒有接觸。根據您的印表機，您可能需要更小的設定值`TARGET_Z`，或者重新校準可能會有所幫助。 TARGET_Z 值並不表示工具頭會移動到該 Z 軸位置；如果一切正常，敲擊將被偵測到，並且工具頭會在到達該位置之前停止。 （例如，在我的 Ender 3 印表機上，我需要`TARGET_Z=-0.500`才能成功敲擊，但典型的 Z 軸偏移量在 -0.200 範圍內。）

## 點擊成功

點擊成功後，您應該確認兩件事：

首先，檢查`Z=0.0`手感是否合適。在工具頭下方墊一張紙，手動將其向下移動到 0（或從 0.5 開始，每次向下移動 0.1）。如果一切正常，應該會有適度的摩擦力。

如果摩擦力過大或過小，請多敲擊幾次。如果偵測結果穩定，您可以嘗試`tap_adjust_z`新增一個常數值。您可以在運行時嘗試更改此值`PROBE_EDDY_NG_SET_TAP_ADJUST_Z VALUE=0.050`，然後再次運行`PROBE_EDDY_NG_TAP`（請參閱下文「調整敲擊 Z 軸」）。預設的敲擊設定最多進行 5 次敲擊，尋找任意 3 次敲擊的標準差小於等於 0.025 的敲擊結果，然後取這 3 次敲擊結果的平均值。所有這些值都是可配置的。

您也可以查看點擊圖表，看看點擊檢測是過早還是太晚。

其次，使用 Klipper 進行床面網格劃分`BED_MESH_CALIBRATE METHOD=rapid_scan`。如果一切正常，列印點的床面網格偏差應該非常接近 0（±0.005）。 （目前僅適用於 Klipper。）

### 調整 Tap Z

當工具頭接觸到列印平台且步進馬達持續運轉時，整個系統會發生輕微的形變，因為步進馬達施加的力需要傳遞出去。根據閾值、系統其他部分的剛度以及其他因素，檢測到的敲擊 Z 軸偏移量可能（通常）略低或（在極少數情況下）略高。您可以使用組態參數`tap_adjust_z`指定固定值，該值將在敲擊時加到計算出的 Z 軸偏移量中，例如：

```
tap_adjust_z: 0.025
```

你可以透過以下步驟找到這個數值：先執行一次 Z 軸校準`TAP`，然後在工具頭下方墊一張紙，手動將工具頭移到 0.0 位置。如果感覺太緊，就稍微向上移動工具頭，直到找到適合第一層列印的鬆緊度（別忘了稍微向上移動一點，然後再向下移動一點，以補償反沖）。找到合適的鬆緊度後，Z 軸位置就是你的 Z 軸校準值。你可以在設定檔中設定這個值，也可以在運行`tap_adjust_z`時使用類似 `--no `PROBE_EDDY_NG_SET_TAP_ADJUST_Z VALUE=0.050`...`tap_adjust_z`

### 交替點擊模式

預設的點擊模式是`butter`，它使用巴特沃斯濾波器來精確檢測點擊。還有一種精度較低的移動平均模式，稱為`wma`（這曾經是原始模式，但已被替換）。在某些情況下，它可能有用；此模式的閾值通常較高。但它也很有可能在不久的將來被移除。如果您遇到過這種情況，即，而，`butter`但，如果，則無法提供良好的結果`wma`

# 6. 宏

## 归位

⚠️在進行 Z 軸归位之前，您必須有相應的裝置`safe_z_home`或`homing_override`設置，將工具頭移到工作台上方。感測器下方需要金屬支撐，如果您嘗試在 (0,0) 點回零，則可能會出現問題，具體取決於感測器的位置。 *

建議的 Z 軸歸巢序列可以如下所示（通過`homing_override`）：

```
[homing_override]
axes: z
set_position_z: 0
gcode:
   G90
   G0 Z15 F600
   G28 X Y
    ## Z 限位开关的 XY 位置
    ##通过后将X0和Y0更新为你的值（如X150、Y150）
    ## Z 轴限位位置定义
   G0 X150 Y150 F3600    #300mm   
   G28 Z F1000
   #G0 Z5 F1000
   G90
   G0 Z2 F1000
   G4 S1
   M400
   PROBE_EDDY_NG_PROBE_STATIC HOME_Z=1
   G0 Z2 F1000
```

使用 ` `PROBE_EDDY_NG_PROBE_STATIC HOME_Z=1``HOME_Z=1`並非絕對必要（換句話說，直接使用 `with` 也可以`G28 Z`），但它可以幫助更精確地設定 Z 軸原點。 `tap` 會自動設定精確的 Z 軸偏移量。

## 開始列印

建議的列印流程如下：

1. 删除床网
2. G28
3. 3Z/龙门架调平
4. 热床加热(实际温度),热端(预热温度)
5. 清洁热端喷嘴(重要!!)
6. G28
7. `PROBE_EDDY_NG_TAP`
8. 床网
9. 自由发挥
10. 列印
## 結束印刷

`PROBE_EDDY_NG_TAP`設定 Z 軸偏移量和“敲擊偏移量”，後者應用於列印床網格和其他探針操作。這些設定僅對上一次敲擊有效。我建議在列印結束後清除敲擊偏移量（可能還需要清除 Z 軸偏移量），以避免混淆：

```
PROBE_EDDY_NG_SET_TAP_OFFSET VALUE=0
```
# 7. 配置參考

這裡列出了配置選項及其預設值。請將這些配置選項新增至您的`[probe_eddy_ng ...]`配置部分。請勿修改任何 Python 檔案！
## 基本配置

- `probe_speed: 5.0`- 執行正常歸航操作的速度
- `lift_speed: 10.0`- 探測作業期間提升工具頭的速度
- `move_speed: 50.0`- 在 xy 平面上的移動速度（通常僅用於校準）
- `home_trigger_height: 2.0`- 虛擬限位開關觸發的高度。建議取值範圍為 1.0 到 3.0，2.0 或 2.5 是不錯的選擇。此高度也用於掃描操作。
- `x_offset: 0.0`，`y_offset: 0.0`- 探針相對於工具頭的位置。
- `calibration_z_max: 5.0`- 用於校準的最大 z 值。 5.0 是一個不錯的預設值，無需使用更高的值進行校準。校準過程會自動在開始和結束時使用無效值。
- `reg_drive_current: 15`- LDC1612感測器的「驅動電流」。該值通常因感測器而異，取決於線圈設計和工作距離。 BTT Eddy的預設值為15。 （可透過將工具頭放置在工作台上方約10毫米處並執行命令來獲得初始值`LDC_NG_CALIBRATE_DRIVE_CURRENT`。）

## 點選配置

- `tap_drive_current: 0`- 用於輕敲操作的「驅動電流」。如果為零或未設置，則`reg_drive_current`使用預設值。輕敲操作是指讀取比基本歸位更靠近列印床的數值，因此可能需要不同的驅動電流，通常更高。例如，BTT Eddy 在該值為 16 時表現最佳。請注意，感測器需要分別針對兩種驅動電流進行校準。將此參數傳遞`DRIVE_CURRENT`給`EDDY_NG_CALIBRATE`.
- `tap_start_z: 3.2`- 啟動回零操作的 Z 軸位置。需要對該高度進行微調，以確保感測器能夠在整個回零範圍內（即從該值到 tap_target_z）提供讀數，而這取決於 tap_drive_current。當 tap_drive_current 增加時，感測器可能無法讀取更高高度的值。例如，BTT Eddy 通常無法在驅動電流為 16 時讀取高於 3.5mm 的高度（但在驅動電流為 15 時無法讀取低於 0.5mm 的值）。 （請注意，所有這些值都是噴嘴到工具頭的偏移量。實際的感測器線圈安裝位置更高——但必須位於噴嘴上方 2.5 到 3mm 之間，理想情況下約為 2.75mm。如果出現振幅誤差，請嘗試稍微升高或降低感測器線圈。）
- `tap_target_z: -0.250`- 攻牙操作的目標 Z 軸位置。這是攻牙失敗時工具頭可以到達的最低位置。請勿將此值設定得太低，否則攻牙失敗時工具頭可能會試圖穿透列印平台。 -0.250 這樣的值與手動調整 Z 軸時噴嘴向下移動過多一兩個刻度並無太大差異。
- `tap_mode`- 如何計算抽頭。選項有`wma`（預設）或`butter`。`wma`使用移動平均導數，`butter`使用巴特沃斯濾波器，後者更準確且對雜訊不敏感。
- `tap_threshold`- 偵測輕觸的閾值。此值取決於感測器和輕觸模式。對於`wma`，預設值為`1000`。對於`butter`，預設值為`250`。運行並查看圖表可以獲得一個合適的閾值`PROBE_EDDY_NG_TAP`。您也可以透過實驗來確定此值。較低的值會使輕觸檢測更加靈敏。較高的值會導致偵測等待時間過長才能辨識到輕觸。您可以使用命令`THRESHOLD`的手動參數`TAP`進行實驗以找到合適的值。您可能還需要為不同的建置平台使用不同的閾值。
- `tap_speed: 3.0`- 執行點擊操作的速度。此值不應遠低於 3.0，但您可以嘗試更低或更高的值。此值不宜過高，因為 Klipper 需要一些時間來回應點擊觸發，即使過了點擊點，工具頭仍會以該速度繼續移動。因此，您可以考慮任何您認為能夠舒適地觸發工具頭移動到 tap_target_z 的速度。
- `tap_adjust_z: 0.0`- 一個固定的附加量，用於添加到計算出的攻絲 Z 軸偏移量中。如果計算出的攻牙高度過高或過低，可以使用此值。正值會抬高刀頭，負值會降低刀頭。
- `tap_samples: 3`- 計算抽頭偏移時要考慮的抽頭樣本數。
- `tap_max_samples: 5`- 放棄之前可以採集的最大水龍頭樣本數量。
- `tap_samples_stddev: 0.025`- 任何計數樣本必須具有的最大標準差`tap_samples`才能被考慮。換句話說，在預設配置下，將採集 3 個樣本（`tap_samples`）。如果它們的平均值小於此值`tap_samples_stddev`，則使用它們的平均值。如果大於此值，則將繼續採集樣本，`tap_max_samples`直到 3 個樣本的標準差足夠低為止。

# 8. EDDY NG作者连结

github
[[https://github.com/vvuk/eddy-ng/wiki]]

bilibili(搬砖)
[[https://www.bilibili.com/video/BV1TYbmzMEta/?spm_id_from=333.337.search-card.all.click&vd_source=b18807914422c7882aaf99bd8d17c669]]