# RNG-Core Wiki — Tạo Aura (v1.0)

Tài liệu này hướng dẫn **từ cơ bản → nâng cao** để bạn tự tạo aura mới cho RNG-Core: cấu trúc file, roll chance, conditions, commands, icon/GUI, và đặc biệt là **particles + toàn bộ shapes** (kèm mô tả và `shape_params`).

---

## 1) Aura là gì? Tạo ở đâu?

Mỗi aura là **1 file YAML** nằm trong:

```
plugins/RNG-Core/auras/<id>.yml
```

* `<id>` nên **viết thường**, không dấu, không space (vd: `angel`, `starlight`, `crimson_void`)
* Sau khi tạo file → `/rngcore reload` (hoặc restart server nếu bạn vừa update nhiều thứ).

---

## 2) Cấu trúc aura cơ bản (bắt buộc)

Ví dụ tối giản:

```yml
id: angel
display-name: "&#fffb94&lAngel"

rarity:
  n: 1000

chance_percent: 0.1
pool: "default"
```

### 2.1 `display-name`

* Là tên hiển thị: title khi roll, tên ở GUI…
* Hỗ trợ màu:

  * `&a`, `&l`…
  * `&#RRGGBB` (hex)

### 2.2 `rarity.n`

* Chỉ để hiển thị kiểu “1 trên N”, ví dụ `n: 1000` → `1/1000`.
* **Không quyết định roll** nếu bạn dùng `chance_percent` (nhưng nên đặt đúng để dễ đọc).

### 2.3 `chance_percent` (QUAN TRỌNG)

* Đây là “tỉ lệ %” của aura **trong pool**.
* Ví dụ:

  * `100` = cực dễ (so với các aura khác)
  * `1` = 1%
  * `0.1` = 0.1%
  * cực thấp nhất được phép: `0.00000000000001` (%)

> Lưu ý: **Không cần tổng = 100%**. Plugin sẽ normalize bằng tổng `chance_percent` của các aura hợp lệ trong pool (sau khi lọc conditions).

> ⚠️ Lưu ý quan trọng về `chance_percent`:
>
> - `chance_percent` **KHÔNG phải** là tỉ lệ trên 100% tuyệt đối.
> - Đây là **trọng số theo % tương đối** trong pool sau khi lọc conditions.
>
> Ví dụ pool có:
> - Aura A: `chance_percent: 1`
> - Aura B: `chance_percent: 0.1`
>
> → tổng = 1.1  
> → Aura A ≈ 90.9%, Aura B ≈ 9.1%
>
> Vì vậy:
> - Không cần (và không nên) cố làm tổng = 100
> - Aura hiếm chỉ cần **rất nhỏ** (vd `0.00001`)

### 2.4 `pool`

* Nhóm aura. Roll sẽ chọn theo pool (mặc định `default`).
* Sau này bạn có thể tách pool: `event`, `vip`, `world_nether`, v.v.

---

## 3) Conditions (điều kiện xuất hiện khi roll)

Bạn có thể giới hạn aura chỉ roll được trong vài điều kiện.

```yml
conditions:
  worlds: ["world"]
  biomes: ["PLAINS", "MEADOW"]
  time:
    from: 13000
    to: 23000
  weather: ["CLEAR", "RAIN"]
```

### 3.1 `worlds`

* Danh sách world name. Rỗng `[]` nghĩa là **không giới hạn**.

### 3.2 `biomes`

* Dùng tên biome kiểu Bukkit (UPPERCASE).
* Lấy biome bằng F3 hoặc tra wiki biome Bukkit.

### 3.3 `time`

* `0..23999` theo Minecraft
* Có hỗ trợ wrap-around:

  * `from: 13000` `to: 2000` nghĩa là **đêm + rạng sáng**.

### 3.4 `weather`

* `CLEAR`: không mưa, không sấm
* `RAIN`: đang mưa nhưng không sấm
* `THUNDER`: đang sấm sét

---

## 4) Commands khi roll trúng / trùng

```yml
commands:
  on-win:
    - "[console] say {player} đã roll được aura angel!"
  on-duplicate:
    - "[console] eco give {player} 100"
```

### Prefix hỗ trợ

* `[console] <cmd>`: chạy bằng console
* `[player] <cmd>`: chạy bằng người chơi
* `[op] <cmd>`: tạm set OP cho player để chạy cmd (xong trả lại trạng thái OP ban đầu)

### Placeholder trong command

* `{player}`, `{uuid}`
* `{aura_id}`, `{aura_name}`, `{rarity_n}`
* `{is_duplicate}`, `{rolls}`

---

## 5) Icon trong GUI (material / custom model / lore / skull texture)

Aura có thể custom icon để hiện trong GUI:

```yml
icon:
  material: PLAYER_HEAD
  custom_model_data: 0
  lore:
    - "&#fffb94&lThiên thần&7 nơi thiên đàng linh thiêng"
    - "&7với vòng hào quang sáng ngời!"
    - ""
    - "&aClick để {action}"
  skull_texture: "<BASE64>"
```

### 5.1 `material`

* Ví dụ: `FEATHER`, `NETHER_STAR`, `PLAYER_HEAD`…

### 5.2 `custom_model_data`

* Nếu dùng resource pack: set số CMD.
* Nếu không dùng: bỏ hoặc để `0`.

### 5.3 `lore` override (lore tùy chỉnh)

* Bạn có thể thay lore hoàn toàn.
* **Bạn yêu cầu giữ dòng click**: hãy luôn để:

  * `""`
  * `"&aClick để {action}"`

`{action}` thường sẽ thành:

* “Equip” nếu chưa equip
* “Unequip” nếu đang equip (tùy GUI config/plugin)

### 5.4 `skull_texture`

Hỗ trợ các dạng:

* **Base64 JSON** (phổ biến nhất) như bạn đang dùng
* Raw JSON `{"textures":{"SKIN":{"url":"..."}}}`
* URL `https://textures.minecraft.net/texture/<hash>`
* Chỉ `<hash>`

> Khuyến nghị: nếu bạn có base64, cứ dùng base64.

---

## 6) Equip + Particles (nâng cao)

Đây là phần làm “aura có hiệu ứng”.

```yml
equip:
  interval_ticks: 4
  particles:
    "1":
      particle: reddust
      amount: 1
      color: "#ffffff"
      size: 1.0
      height: 1
      follow_yaw: true
      chance: 1.0
      view_distance: 96
      shape: ring
      shape_params:
        radius: 1.2
        points: 48
        thickness: 0.15
      animation:
        rotate_speed_deg: 2.0
```

### 6.1 `interval_ticks`

* Mỗi bao nhiêu tick thì vẽ 1 lần.
* Tick: 20 ticks = 1 giây
* Vd `4` = 0.2s/lần (khá mượt), `10` = 0.5s/lần (nhẹ hơn)

---

## 6.2) Equip Attributes (buff người chơi khi equip)

Aura có thể **buff trực tiếp Attribute của player** khi equip  
(và **tự động gỡ sạch khi unequip / reload / logout**).

```yml
equip:
  attributes:
    - attribute: GENERIC_MAX_HEALTH
      amount: 4
      operation: ADD_NUMBER

    - attribute: GENERIC_MOVEMENT_SPEED
      amount: 0.05
      operation: ADD_SCALAR
````

### Attribute

Dùng enum của Bukkit:
[https://hub.spigotmc.org/javadocs/spigot/org/bukkit/attribute/Attribute.html](https://hub.spigotmc.org/javadocs/spigot/org/bukkit/attribute/Attribute.html)

Ví dụ hay dùng:

* `GENERIC_MAX_HEALTH`
* `GENERIC_ATTACK_DAMAGE`
* `GENERIC_MOVEMENT_SPEED`
* `GENERIC_ARMOR`
* `GENERIC_ATTACK_SPEED`

### Operation

| Operation           | Ý nghĩa         |
| ------------------- | --------------- |
| `ADD_NUMBER`        | + thẳng giá trị |
| `ADD_SCALAR`        | + theo % base   |
| `MULTIPLY_SCALAR_1` | nhân sau cùng   |

> 💡 Mỗi aura dùng **UUID modifier cố định**, nên không bị cộng dồn lỗi khi reload.

---

## 7) Particle layer — các field cơ bản

Mỗi layer là 1 “lớp” particle. Bạn có thể chồng nhiều layer để đẹp.

### 7.1 `particle`

* Tên particle (không phân biệt hoa/thường).
* Alias phổ biến:

  * `reddust` → `REDSTONE` (particle dạng dust có màu)
* Ví dụ: `FLAME`, `END_ROD`, `WAX_ON`, `WAX_OFF`, …

### 7.2 `amount`

* Số particle spawn tại mỗi điểm.
* Nếu shape có nhiều points, amount cao sẽ **rất nặng**.

### 7.3 `color` + `size` (chỉ dùng cho `reddust`)

```yml
particle: reddust
color: "#FF00FF"
size: 1.0
```

### 7.4 `material` (chỉ dùng cho particle cần BlockData / ItemStack)

* Ví dụ: `BLOCK_CRACK`, `ITEM_CRACK`…

### 7.5 Height: `heights` / `height` / `height_range`

**1 unit = 1 block theo trục Y, 0 = chân người chơi**

**Ưu tiên:**

1. `heights`
2. `height`
3. `height_range`

Ví dụ:

```yml
heights: [-1, 0, 1, 2]
```

Hoặc:

```yml
height: 1.2
```

Hoặc:

```yml
height_range: "-2..3"
height_step: 1.0
```

### 7.6 `follow_yaw`

* `true`: shape quay theo hướng người chơi nhìn
* `false`: shape cố định theo trục world

### 7.7 `chance`

* Xác suất layer được vẽ mỗi loop:

  * `1.0` = 100%
  * `0.35` = 35%

### 7.8 `view_distance`

* Tuỳ bản implement: thường dùng để hạn chế “ai thấy”.
* Nếu plugin đang mặc định SELF (chỉ player thấy) thì view_distance chủ yếu là giới hạn logic sau này.

### 7.9 Animation: `rotate_speed_deg`

* Xoay thêm mỗi lần vẽ:

```yml
animation:
  rotate_speed_deg: 2.0
```

* `0` = đứng yên

### 7.10 Rotate shape (đứng im)

Bạn có thể **xoay hình học của shape** theo trục X / Y / Z.  
⚠️ Đây **không phải animation**, chỉ là rotate cố định.

```yml
shape_params:
  rotate_x: 90
  rotate_y: 0
  rotate_z: 0
````

### Ví dụ thực tế

* `ring` mặc định nằm ngang
* `rotate_x: 90` → vòng dựng đứng
* `rotate_y` → xoay quanh người
* `rotate_z` → nghiêng hình

---

# 8) Shapes — danh sách đầy đủ + mô tả + shape_params

Mỗi shape generator tạo ra danh sách điểm (points). Các điểm sẽ được:

* đặt quanh người chơi (gốc tại chân)
* cộng thêm height
* quay theo yaw nếu `follow_yaw: true`

> Mẹo performance: shape càng nhiều `points` càng nặng. Luôn bắt đầu thấp rồi tăng dần.

---

## 8.1 `point` — 1 điểm duy nhất

**Hình:** 1 chấm

```yml
shape: point
shape_params:
  x: 0
  z: 0
```

**Params:**

* `x` (default 0)
* `z` (default 0)

---

## 8.2 `line` — 1 đường thẳng

**Hình:** một đường kéo dài 1 hướng

```yml
shape: line
shape_params:
  length: 2.0
  points: 20
  direction: forward
```

**Params:**

* `length` (double) — độ dài
* `points` (int) — số điểm
* `direction` (string): `forward|backward|left|right`

---

## 8.3 `ring` — vòng tròn rỗng

**Hình:** vòng tròn (có thể dày)

```yml
shape: ring
shape_params:
  radius: 1.2
  points: 48
  thickness: 0.15
  thickness_rings: 2
```

**Params:**

* `radius` (double)
* `points` (int, >= 8)
* `thickness` (double, default 0) — độ dày vành (0 = vòng mảnh)
* `thickness_rings` (int, default 1) — số vòng phụ để tạo độ dày

---

## 8.4 `circle` — hình tròn đặc (fill)

**Hình:** đĩa tròn đầy

```yml
shape: circle
shape_params:
  radius: 1.5
  rings: 6
  points: 48
```

**Params:**

* `radius` (double)
* `rings` (int, default 6) — số vòng fill
* `points` (int, default 48) — độ mịn vòng ngoài

---

## 8.5 `square` — hình vuông (thường là fill)

**Hình:** ô vuông (grid)

```yml
shape: square
shape_params:
  size: 2.0
  grid: 10
```

**Params:**

* `size` (double) — cạnh vuông
* `grid` (int, default 10) — độ mịn (grid x grid)

---

## 8.6 `box` — hộp 3D (wireframe/filled)

**Hình:** khung hộp 3D quanh người chơi

```yml
shape: box
shape_params:
  width: 2.0
  height: 1.5
  depth: 2.0
  points: 80
  mode: wireframe
```

**Params:**

* `width`, `height`, `depth` (double)
* `points` (int, default 80)
* `mode`: `wireframe|filled`

> `box` là shape 3D: có thể có point.y ≠ 0 (vẫn cộng thêm height layer như bình thường).

---

## 8.7 `arc` — cung tròn

**Hình:** nửa vòng / cung theo góc

```yml
shape: arc
shape_params:
  radius: 1.5
  start_deg: 0
  end_deg: 180
  points: 32
```

**Params:**

* `radius` (double)
* `start_deg` (double, default 0)
* `end_deg` (double, default 180)
* `points` (int, default 32)

---

## 8.8 `spiral` — xoắn ốc 2D

**Hình:** đường xoắn ốc từ trong ra ngoài

```yml
shape: spiral
shape_params:
  radius_start: 0.2
  radius_end: 1.5
  turns: 3
  points: 120
  clockwise: true
```

**Params:**

* `radius_start` (double, default 0.2)
* `radius_end` (double, default 1.5)
* `turns` (int, default 3)
* `points` (int, default 120)
* `clockwise` (bool, default true)

---

## 8.9 `summon_circle` — vòng triệu hồi

**Hình:** 2 vòng + rune dots xung quanh (giống magic circle)

```yml
shape: summon_circle
shape_params:
  outer_radius: 2.2
  inner_radius: 1.4
  outer_points: 72
  inner_points: 48
  rune_points: 12
  rune_radius: 1.9
  rune_size: 0.0
```

**Params:**

* `outer_radius`, `inner_radius` (double)
* `outer_points`, `inner_points` (int)
* `rune_points` (int)
* `rune_radius` (double)
* `rune_size` (double, default 0)

  * `0` = mỗi rune là 1 điểm
  * `>0` = mỗi rune là 1 vòng nhỏ

---

## 8.10 `wings` — 2 cánh sau lưng

**Hình:** đôi cánh trái/phải, cong nhẹ

```yml
shape: wings
shape_params:
  wing_span: 1.6
  wing_height: 0.8
  wing_length: 1.2
  points: 60
  curve: 0.7
  tilt_deg: 12
```

**Params:**

* `wing_span` (double) — độ xoè ngang
* `wing_height` (double) — biên độ Y
* `wing_length` (double) — độ dài ra sau (z âm)
* `points` (int, default 60)
* `curve` (0..1, default 0.7)
* `tilt_deg` (double, default 0) — nghiêng cánh

---

## 8.11 `single_wing` — 1 cánh

**Hình:** 1 cánh (trái hoặc phải)

```yml
shape: single_wing
shape_params:
  side: left
  wing_span: 1.6
  wing_height: 0.8
  wing_length: 1.2
  points: 60
  curve: 0.7
  tilt_deg: 12
```

**Params:**

* Tất cả như `wings`
* thêm `side: left|right` (default left)

---

## 8.12 `tail` — đuôi sau lưng

**Hình:** dải đuôi uốn nhẹ phía sau

```yml
shape: tail
shape_params:
  length: 2.5
  width: 0.35
  wave: 0.4
  points: 35
  backward_offset: 0.4
```

**Params:**

* `length` (double)
* `width` (double)
* `wave` (0..1)
* `points` (int)
* `backward_offset` (double)

---

## 8.13 `item` — Item 3D bay quanh người chơi

**Hình:** item (kiếm, feather, relic, v.v.) hiển thị 3D quanh player.

```yml
shape: item
shape_params:
  material: FEATHER
  scale: tiny
  x: 1
  z: 1
````

### `scale`

| scale    | Hiển thị                    |
| -------- | --------------------------- |
| `big`    | Đội đầu – ArmorStand thường |
| `normal` | Đội đầu – ArmorStand baby   |
| `small`  | Cầm tay – thường            |
| `tiny`   | Cầm tay – baby + marker     |

---

# 10) Template aura hoàn chỉnh

```yml
id: angel
display-name: "&#fffb94&lAngel"

icon:
  material: PLAYER_HEAD
  lore:
    - "&#fffb94&lThiên thần&7 nơi thiên đàng linh thiêng"
    - "&7với vòng hào quang sáng ngời!"
    - ""
    - "&aClick để {action}"
  skull_texture: "BASE64_TEX"

rarity:
  n: 1000

chance_percent: 0.1
pool: "default"

conditions:
  worlds: []
  biomes: []
  time: []
  weather: []

commands:
  on-win:
    - "[console] say {player} đã roll được aura angel!"
  on-duplicate:
    - "[console] eco give {player} 100"

equip:
  interval_ticks: 4
  particles:
    "1":
      particle: reddust
      amount: 1
      color: "#fffb94"
      size: 1.0
      heights: [0.6, 0.8, 1.0]
      follow_yaw: true
      chance: 1.0
      shape: wings
      shape_params:
        wing_span: 1.6
        wing_height: 0.8
        wing_length: 1.2
        points: 18
        curve: 0.7
        tilt_deg: 12
```
