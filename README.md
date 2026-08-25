# RP2040 掌機 — 成品集合

MAGC 掌機(RP2040 + ILI9341)的整套可燒錄成品:載入器主程式、四個專題的
韌體、以及選單封面。

整個掌機生態系的總綱與入口頁在
**[rp2040-retro-handheld](https://github.com/pondahai/rp2040-retro-handheld)**
(硬體 BOM、各韌體的介紹與致謝);本 repo 只負責收攏其中走載入器選單那一批的
**可燒錄成品**。

<img width="1600" height="1568" alt="IMG_4780" src="https://github.com/user-attachments/assets/78bd0051-8eed-4c60-a15b-a170771f0b97" />


集合日期: 2026-08-16(當日稍晚重編過一次);字典於 2026-08-25 補收。見第三節

---

## 目錄結構

```
uf2/            可燒錄的韌體 —— 不在 git 裡,見下方
covers/         選單封面(96x96 RGB565,直接放 SD 卡)
cover-sources/  封面的來源圖(要重新轉封面時才需要)
```

⚠️ **`uf2/` 沒有進版控。** 二進位檔每次重編都全變,塞進 git 只會讓歷史膨脹。
成品掛在 [Releases](https://github.com/pondahai/rp2040-handheld-bundle/releases),
七個 uf2 是各自獨立的檔案(不是壓縮包),下載後放進 `uf2/` 即可,
校驗碼見第六節。

---

## 一、快速上手

### 第一次設定機器

1. 板子進 BOOTSEL(拔電 → 按住 BOOTSEL → 插電),掛成 `RPI-RP2`
2. 把 `uf2/loader.uf2` 拖進去 — **這一步只要做一次**
3. SD 卡格式化成 FAT16 或 FAT32
4. 把 `uf2/` 裡**四個專題的 uf2**(`DOOM.uf2`、`infoNES_standalone.uf2`、
   `PicoApple2_standalone.uf2`、`RetroDict.ino.uf2`)和 `covers/` 裡對應的
   `.RAW` 全部複製到 SD 卡**根目錄**(不支援子資料夾)。字典還要多放一個
   `/DICT/` 資料夾,見第二節
5. 開機 → 圖形選單 → 左右選 → A 或 START 執行

⚠️ `loader.uf2` 與 `trampoline.uf2` **不要**放進 SD 卡。它們是燒進板子的,
不是選單項目。

### 按鍵

| 按鍵 | 功能 |
|---|---|
| UP / DOWN | 移動選擇(長按會重複) |
| A / START | 燒錄並執行 |
| B | 重新掃描 SD 卡 |
| **開機時按住 SELECT** | **直接進 BOOTSEL(逃生口)** |

最後一項很重要:載入器為了省空間把 USB 關掉了,所以 `picotool` 沒有 reset
interface 可用 —— 一旦燒進去,沒有這個逃生口就只能靠實體 BOOTSEL 按鈕。

---

## 二、檔案清單

### uf2/ — 韌體

| 檔案 | 大小 | 說明 | 來源 repo |
|---|---|---|---|
| `loader.uf2` | 24,064 B | **載入器主程式**。燒進板子,不是 SD 卡項目 | [rp2040-retro-loader](https://github.com/pondahai/rp2040-retro-loader) |
| `trampoline.uf2` | 6,144 B | 跳板。已經包在下面三個遊戲裡,單獨留著是給重新 merge 用的 | [rp2040-retro-loader](https://github.com/pondahai/rp2040-retro-loader) |
| `DOOM.uf2` | 4,174,336 B | DOOM(韌體 + 1.7MB 地圖檔) | [rp2040-doom-ili9341](https://github.com/pondahai/rp2040-doom-ili9341) |
| `infoNES_standalone.uf2` | 1,056,768 B | InfoNES(紅白機模擬器) | [rp2040-ili9341-infones](https://github.com/pondahai/rp2040-ili9341-infones) |
| `PicoApple2_standalone.uf2` | 401,408 B | PicoApple2(Apple II 模擬器) | [PicoApple2](https://github.com/pondahai/PicoApple2) |
| `RetroDict.ino.uf2` | 434,688 B | 電子字典。給選單用,不夾跳板 | [rp2040-retro-dict](https://github.com/pondahai/rp2040-retro-dict) |
| `RetroDict_standalone.uf2` | 467,456 B | 電子字典 standalone,夾了跳板可直接 USB 燒 | [rp2040-retro-dict](https://github.com/pondahai/rp2040-retro-dict) |

三個遊戲都是 **standalone 版**,意思是檔案最前面帶了跳板,所以**兩種用法都成立**:

1. **放 SD 卡** — 由選單載入。載入器會丟掉 `0x10004000` 以下的區塊(跳板),
   只寫本體
2. **直接拖進 `RPI-RP2`** — 不需要載入器也不需要 SD 卡,開機直接進該專題。
   會覆蓋掉載入器,要換回來就重燒 `loader.uf2`

**字典的狀況不一樣。** `rp2040-retro-dict` 的 `build_offset.bat` 一次產出兩個檔,
兩個都已收進 Releases:`RetroDict.ino.uf2`(434,688 B,給選單用,不夾跳板,
**不能單獨拖進 `RPI-RP2`**,前 16KB 是空的)與 `RetroDict_standalone.uf2`
(467,456 B,夾了跳板可直接 USB 燒)。以上大小是實測值,不是抄 HANDOVER 的。
封面只有 `RETRODICT.INO.RAW` 一份,對應走選單的那個檔;standalone 版是 USB 用的,
不必放 SD 卡。
另外字典還要 SD 卡上的 `/DICT/`(`EC.IDX`、`EC.DAT`、`ECC.IDX`、`FONT.BIN`,
漢英另加 `CE.IDX`、`CE.DAT`),那是資料不是韌體,不走選單。

### covers/ — 選單封面

| 檔案 | 對應的 uf2 |
|---|---|
| `DOOM.RAW` | `DOOM.uf2` |
| `INFONES_STANDALONE.RAW` | `infoNES_standalone.uf2` |
| `PICOAPPLE2_STANDALONE.RAW` | `PicoApple2_standalone.uf2` |
| `RETRODICT.INO.RAW` | `RetroDict.ino.uf2` |

全部是 96x96 RGB565 big-endian、**固定 18432 bytes、無標頭**。

檔名(去掉副檔名)必須跟 uf2 一致才會被認出來。FAT 不分大小寫,所以
`INFONES_STANDALONE.RAW` 配 `infoNES_standalone.uf2` 是對得上的。
沒有封面的項目顯示成灰色色塊,照樣能選。

⚠️ 檔名含副檔名**不要超過 26 字元**,那是 fatlite 的 LFN 拼裝上限。
`PICOAPPLE2_STANDALONE.RAW` 是 25 字元,已經很接近了。

### cover-sources/ — 封面來源圖

`doom_cover.jpg` / `infones_standalone.jpg` / `picoapple2_standalone.jpg` /
`retrodict_icon_96.png`

`retrodict_icon_96.png` 已經是 96x96,是字典 repo 轉封面時的中間產物,原圖在
[rp2040-retro-dict](https://github.com/pondahai/rp2040-retro-dict) 的 `assets/`。

只有要重新轉封面時才需要。轉換指令(在載入器 repo 底下執行):

```bash
python tools/make_thumb.py <來源圖> --fit -o <名稱>.RAW
```

去背照(白底)要多加 `--drop-bg --bg FFFFFF`。**`--bg FFFFFF` 很重要** ——
`--drop-bg` 預設用黑色回填,而選單是白底,會變成一塊黑方塊。
`DOOM.RAW` 就是這樣轉的:

```bash
python tools/make_thumb.py doom_cover.jpg --fit --drop-bg --bg FFFFFF -o DOOM.RAW
```

---

## 三、重編紀錄

### 2026-08-25 — 補收字典

字典的兩個 uf2 補進 Releases(掛在同一個 `v2026.08.16` tag 底下,與其他五個並列)。
編譯順序照樣是先載入器後專題,夾到的跳板與其他三個 standalone 同一份
(`19d1e132634b49f8`,6,144 B,未變)。

| 檔案 | 這次動作 | 說明 |
|---|---|---|
| `RetroDict.ino.uf2` | 新編 | `build_offset.bat`,佈局檢查通過:image `0x10004000..0x10039100`(217,344 B),SP=`0x20042000` Reset=`0x100040e3` |
| `RetroDict_standalone.uf2` | 新編 + merge | 913 blocks(跳板 12 + 本體 849 + 填充 52) |

⚠️ 偏移錯了**不會**編譯失敗,只會在實機上黑畫面,症狀跟沒燒進去一樣 ——
所以 `check_flash_layout.py` 那關一定要過才算數。

⚠️ 載入器 repo 在 `128b79a` 之後已經有新 commit,重編出來的 `loader.uf2`
(25,088 B)與 Releases 裡那份 24,064 B 已經不同。這次**沒有**更新 `loader.uf2`,
Releases 裡仍是 2026-08-16 那份;下次要一起重編再說。

### 2026-08-16 重編紀錄

原本這一節記的是「兩份舊韌體夾的是舊跳板」。**已經解決** —— 五個 uf2 裡有
四個在 2026-08-16 中午重新編譯/merge 過,三個 standalone 現在夾的都是同一份
新跳板(2,820 B / 12 blocks)。

| 檔案 | 這次動作 | 說明 |
|---|---|---|
| `loader.uf2` | 重編 | 載入器跟進到 `10a276e`(沒真的抹寫時顯示 VERIFYING 而非 WRITING FLASH) |
| `trampoline.uf2` | 重編 | 內容同上次瘦身後的版本,只是重新產生 |
| `infoNES_standalone.uf2` | 重新 merge | 本體未變(HEAD 仍是 `b4b858e`,之後只多了文件 commit),換成新跳板 |
| `PicoApple2_standalone.uf2` | 重編 + merge | 跟進到 `d8e78a0`(選單整列 SPI 緩衝 + 檔名快取,捲動不再是 O(n)) |
| `DOOM.uf2` | 未動 | 來源 repo 沒有新 commit,本來就是新跳板 |

✅ **這批重編的檔案已經上機驗證過。** build 期的佈局檢查如下:

```
loader:     11,968 / 16,384 bytes (73%, 剩 4,416)
PicoApple2: image 0x10004000..0x10031000, SP=0x20042000 Reset=0x100040e3
InfoNES:    image 0x10004000..0x10080f78, NVRAM slot0 0x10082000, 餘裕 4,232
```

重編指令:

```bash
# 載入器 + 跳板(在 rp2040-retro-loader 底下,build 目錄已 configure 過)
~/.pico-sdk/cmake/v3.31.5/bin/cmake.exe --build build

# InfoNES —— 這個 build 會自己 merge 出 standalone
~/.pico-sdk/cmake/v3.31.5/bin/cmake.exe --build software/infones/build_offset

# PicoApple2 —— 同樣自己 merge,結尾會 pause,非互動執行要餵 stdin
cmd /c build_offset.bat < /dev/null
```

InfoNES 與 PicoApple2 的 build 都會自己去抓載入器 repo 的
`build/trampoline.uf2` 再 merge,所以**順序很重要:先編載入器,再編專題**,
否則夾到的還是舊跳板。

---

## 四、Flash 佈局

單一事實來源是載入器的 `common/boot_map.h`。

```
0x10000000  boot2(256B) + 載入器 或 跳板     BOOT_REGION_SIZE = 16KB
0x10004000  專題本體,第一個 byte 就是向量表   APP_BASE
```

DOOM 是唯一還帶額外資料的:

```
0x10004000  韌體    ..0x10044448   (263,240 B)
0x10046000  地圖檔  ..0x101FD898 (1,800,344 B)
0x10200000  flash 尾端 — 只剩 10,088 bytes
```

DOOM 從選單載入時要寫入約 1.97MB,**要等 30–45 秒**,畫面看起來像當機是正常的,
不要拔電。其他兩個只有幾百 KB,感覺不出來。

---

## 五、出處

重編當下各 repo 的 commit(2026-08-16 中午):

| 專題 | Repo | Commit | 上一版 |
|---|---|---|---|
| 載入器 / 跳板 | [rp2040-retro-loader](https://github.com/pondahai/rp2040-retro-loader) | [`10a276e`](https://github.com/pondahai/rp2040-retro-loader/commit/10a276e) | `db5228e` |
| DOOM | [rp2040-doom-ili9341](https://github.com/pondahai/rp2040-doom-ili9341) | [`015a01d`](https://github.com/pondahai/rp2040-doom-ili9341/commit/015a01d) | 同 |
| InfoNES | [rp2040-ili9341-infones](https://github.com/pondahai/rp2040-ili9341-infones) | [`b4b858e`](https://github.com/pondahai/rp2040-ili9341-infones/commit/b4b858e) | 同 |
| PicoApple2 | [PicoApple2](https://github.com/pondahai/PicoApple2) | [`d8e78a0`](https://github.com/pondahai/PicoApple2/commit/d8e78a0) | `876b9be` |
| 字典(2026-08-25) | [rp2040-retro-dict](https://github.com/pondahai/rp2040-retro-dict) | [`9adf319`](https://github.com/pondahai/rp2040-retro-dict/commit/9adf319) | 新收 |

InfoNES 的工作區當時有未提交的 nvram / ramdisk 檔(`nvram_path.h`、
`tools/nvram_save_test/`、`tools/ramdisk_fat12.h`),但都還沒被 build 引用,
所以這份韌體等同 `b4b858e`。

`DOOM.uf2` 另外也掛在
<https://github.com/pondahai/rp2040-doom-ili9341/releases/tag/retro-loader-v1>。

**備份狀態**:各專題 repo 的 `.gitignore` 都排除 `*.uf2` / `*.RAW` / 圖檔,
所以這些成品在來源那邊是沒有副本的。現在由本 repo 收著:封面與
`cover-sources/` 進 git,`uf2/` 走 Releases。

---

## 六、校驗碼(sha256 前 16 碼)

```
uf2/loader.uf2                    2b08ad4e9454a29b
uf2/trampoline.uf2                19d1e132634b49f8
uf2/DOOM.uf2                      0f86b7142a523c4b
uf2/infoNES_standalone.uf2        6cea111ccc6c564a
uf2/PicoApple2_standalone.uf2     1a1da3896ea8dee6
uf2/RetroDict.ino.uf2             3ea5227878584827
uf2/RetroDict_standalone.uf2      312f1894d1686348
covers/DOOM.RAW                   c4e8be3cd258d37a
covers/INFONES_STANDALONE.RAW     9f9d1fc3b15bdd16
covers/PICOAPPLE2_STANDALONE.RAW  3f6c27d4fe5af0c3
covers/RETRODICT.INO.RAW          2c1ca71602ced177
```
