# เอกสารการแปลภาษาไทยและปรับค่าให้เหมาะกับประเทศไทย (Genergy Dashboard)

เอกสารนี้อธิบายขอบเขตการแก้ไข สถาปัตยกรรมการโหลดไฟล์ ตำแหน่งของข้อความที่แปล
และขั้นตอนการนำขึ้นใช้งานบน Home Assistant สำหรับผู้ที่ต้องดูแลรักษาโค้ดต่อ

## 1. ขอบเขตงาน

- แปลข้อความที่ผู้ใช้มองเห็นทั้งหมดบนหน้า Overview เป็นภาษาไทย
- แปล UI ตั้งค่า (Settings) ส่วนหลัก ได้แก่ ชื่อแท็บ หัวข้อส่วน ป้ายเอนทิตี
  สวิตช์ฟีเจอร์ ปุ่ม และข้อความแจ้งสถานะ
- เปลี่ยนสกุลเงินจากยูโรเป็นบาท ทั้งสัญลักษณ์และค่าตั้งต้น
- ปรับเกณฑ์ราคาค่าไฟให้อยู่ในสเกลของประเทศไทย
- เพิ่มไฟล์แปลของตัวช่วยตั้งค่า (config flow) เป็นภาษาไทย

ศัพท์เฉพาะที่คงไว้เป็นภาษาอังกฤษโดยเจตนา ได้แก่ EMHASS, HAEO, MPC, Nordpool,
SoC, HVAC, PV เนื่องจากเป็นชื่อเฉพาะหรือมาตรฐานสากล

## 2. สถาปัตยกรรมการโหลดไฟล์ของส่วนขยาย

ส่วนขยายให้บริการไฟล์ frontend ทั้งหมดจากไดเรกทอรี `frontend/` ไม่ใช่ `dist/`
จุดลงทะเบียนทรัพยากรอยู่ใน `__init__.py` ผ่าน `add_extra_js_url` และ
`_NoCacheJSView` ดังนั้นการแก้ไขต้องทำที่ `frontend/` เท่านั้น ส่วน `dist/`
เป็นไฟล์โหมดปลั๊กอินแบบเดิมที่ไม่ถูกใช้งาน

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#ffffff','primaryTextColor':'#000000','primaryBorderColor':'#000000','lineColor':'#000000','secondaryColor':'#f2f2f2','tertiaryColor':'#ffffff','fontFamily':'monospace'}}}%%
flowchart TD
    A["Home Assistant core"] --> B["__init__.py async_setup_entry"]
    B --> C["add_extra_js_url<br/>sigenergy-dashboard.js"]
    B --> D["_NoCacheJSView<br/>serve /genergy_dashboard/js"]
    B --> E["register dashboard<br/>from default_dashboard.json"]
    B --> F["load translations<br/>en.json / th.json"]
    C --> G["Browser frontend"]
    D --> G
    E --> G
    F --> H["Config flow UI"]
    G --> I["sigenergy-house-card.js<br/>การ์ดบ้านสามมิติ"]
    G --> J["sigenergy-dashboard.js<br/>การ์ดตั้งค่า กราฟ Sankey modal"]
```

## 3. ไฟล์ที่แก้ไข

ตารางสรุปไฟล์ที่เปลี่ยนแปลง ขอบเขต และผลที่ผู้ใช้เห็น

| ไฟล์ | ขอบเขตการแก้ไข | ผลต่อผู้ใช้ |
|------|----------------|------------|
| `frontend/sigenergy-house-card.js` | ป้ายบนบ้าน สถานะ และข้อความเวลาใช้งานแบตเตอรี่ | ป้ายและสถานะบนการ์ดบ้านเป็นไทย |
| `frontend/sigenergy-dashboard.js` | การ์ดสถานะ กราฟ โหนด Sankey modal UI ตั้งค่า สกุลเงิน และเกณฑ์ราคา | ทุกข้อความบนหน้าหลักและตั้งค่าเป็นไทย แสดงหน่วยบาท |
| `default_dashboard.json` | หัวข้อการ์ดตั้งต้น ธีม หน่วย และเกณฑ์ราคา | แดชบอร์ดที่สร้างใหม่เป็นไทยและธีมมืด |
| `translations/th.json` | ไฟล์ใหม่ สำหรับตัวช่วยตั้งค่า | หน้าตั้งค่าตอนเพิ่มส่วนขยายเป็นไทย |

## 4. แผนผังตำแหน่งข้อความที่แปล

ไดอะแกรมต่อไปนี้จับคู่ส่วนของ UI กับฟังก์ชันหรือบล็อกข้อมูลที่เป็นแหล่งข้อความ
ใช้เป็นจุดอ้างอิงเวลาต้องแก้ไขเพิ่มเติม

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#ffffff','primaryTextColor':'#000000','primaryBorderColor':'#000000','lineColor':'#000000','secondaryColor':'#f2f2f2','tertiaryColor':'#ffffff','fontFamily':'monospace'}}}%%
flowchart LR
    subgraph HC["sigenergy-house-card.js"]
        HC1["LABELS"]
        HC2["click_zones / _defaultClickZones"]
        HC3["_render สถานะแบต กริด EV"]
        HC4["_batteryRuntime ข้อความเวลา"]
    end
    subgraph DB["sigenergy-dashboard.js"]
        DB1["DEFAULT pricing display"]
        DB2["status และ stat cards"]
        DB3["_sankeyNodes _panelNodes"]
        DB4["_GENERGY_MODAL_TYPES"]
        DB5["_genergyElementRows"]
        DB6["_renderEntities _renderFeatures"]
        DB7["chart series และ axis titles"]
        DB8["toast และ status messages"]
    end
    subgraph TPL["default_dashboard.json"]
        T1["card primary titles"]
        T2["pricing display block"]
    end
    subgraph TR["translations/th.json"]
        TR1["config flow steps"]
    end
```

## 5. การแปลงสกุลเงินและเกณฑ์ราคา

ค่าตั้งต้นเดิมอ้างอิงตลาดยุโรป (ยูโร, สเกลราคา 0.10 ถึง 0.25 ต่อ kWh)
ปรับเป็นบาทและสเกลของไทยดังนี้

| รายการ | ค่าเดิม | ค่าใหม่ |
|--------|---------|---------|
| สัญลักษณ์สกุลเงิน | EUR | THB |
| เกณฑ์ราคาถูก (cheap_threshold) | 0.10 | 3.0 |
| เกณฑ์ราคาแพง (expensive_threshold) | 0.25 | 4.5 |
| แหล่งราคาตั้งต้น (source) | nordpool | custom |

หมายเหตุ ค่าเกณฑ์ราคาเป็นค่าตั้งต้นสำหรับการสร้างแดชบอร์ดใหม่ ผู้ใช้ปรับได้
ภายหลังที่แท็บ ราคาค่าไฟ ในการ์ดตั้งค่า

## 6. ขั้นตอนการนำขึ้นใช้งาน

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#ffffff','primaryTextColor':'#000000','primaryBorderColor':'#000000','lineColor':'#000000','secondaryColor':'#f2f2f2','tertiaryColor':'#ffffff','fontFamily':'monospace'}}}%%
sequenceDiagram
    participant Dev as ผู้ดูแล
    participant Edit as Studio Code Server
    participant HA as Home Assistant
    participant UI as เบราว์เซอร์
    Dev->>Edit: คัดลอกไฟล์ที่แก้ไป config/custom_components
    Edit->>HA: บันทึกไฟล์
    Dev->>HA: รีสตาร์ท Home Assistant
    HA->>HA: ลงทะเบียน JS resources ใหม่
    Dev->>UI: hard refresh ล้างแคช
    UI->>HA: โหลด sigenergy-dashboard.js เวอร์ชันใหม่
    UI-->>Dev: แสดงผลภาษาไทยและหน่วยบาท
```

ไฟล์ที่ต้องคัดลอกไปทับ ภายใต้ `config/custom_components/genergy_dashboard/`

```
frontend/sigenergy-house-card.js
frontend/sigenergy-dashboard.js
default_dashboard.json
translations/th.json
```

## 7. การดูแลรักษาเมื่อ upstream ออกเวอร์ชันใหม่

การอัปเดตส่วนขยายผ่าน HACS จะเขียนทับไฟล์ที่แก้ไว้ทั้งหมด แนวทางจัดการ

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#ffffff','primaryTextColor':'#000000','primaryBorderColor':'#000000','lineColor':'#000000','secondaryColor':'#f2f2f2','tertiaryColor':'#ffffff','fontFamily':'monospace'}}}%%
flowchart TD
    A["upstream release ใหม่"] --> B{"ต้องการฟีเจอร์ใหม่หรือไม่"}
    B -- "ไม่" --> C["ปิด auto-update ใน HACS<br/>คงเวอร์ชันที่แปลแล้ว"]
    B -- "ใช่" --> D["อัปเดตผ่าน HACS"]
    D --> E["diff ไฟล์ frontend กับเวอร์ชันที่แปล"]
    E --> F["นำชุดแปลกลับเข้าไฟล์ใหม่"]
    F --> G["ตรวจ syntax และ JSON"]
    G --> H["รีสตาร์ทและทดสอบ"]
```

แนวทางที่แนะนำคือเก็บชุดข้อความแปลเป็นคู่ค้นหาแทนที่ (mapping) เพื่อให้นำกลับเข้า
ไฟล์เวอร์ชันใหม่ได้รวดเร็ว และตรวจสอบทุกครั้งด้วย `node --check` สำหรับไฟล์ JS และ
`JSON.parse` สำหรับไฟล์ JSON ก่อนนำขึ้นใช้งาน

## 8. รายการตรวจสอบก่อนส่งมอบ

| ขั้นตอน | คำสั่งหรือวิธี | เกณฑ์ผ่าน |
|---------|----------------|-----------|
| ตรวจไวยากรณ์ JS | `node --check <file>.js` | ไม่มี error |
| ตรวจความถูกต้อง JSON | `node -e "JSON.parse(...)"` | parse ได้ |
| ตรวจสัญลักษณ์ยูโรตกค้าง | ค้นหา `€` และ `EUR` | ไม่พบ |
| ตรวจการแสดงผลจริง | เปิดแดชบอร์ดหลังรีสตาร์ท | ข้อความเป็นไทย หน่วยบาท |

## 9. การรีแบรนด์และระบบดีไซน์ Apple-Glass

ส่วนขยายถูกรีแบรนด์เป็น Teerathap sigenergy และปรับภาษาภาพให้เป็นแนว
Apple-Glass (frosted glass บนพื้นหลังโทนเข้ม ใช้สีเน้นอบอุ่นเพียงสีเดียว
ฟอนต์ระบบ และตัวเลข mono tabular)

### 9.1 จุดที่เปลี่ยนชื่อแบรนด์

| ไฟล์ | คีย์ | ค่าใหม่ |
|------|------|---------|
| `const.py` | `DASHBOARD_TITLE` | Teerathap sigenergy |
| `manifest.json` | `name` | Teerathap sigenergy |
| `config_flow.py` | `title` ทั้งสามขั้น | Teerathap sigenergy |
| `strings.json` / `translations/th.json` | หัวข้อขั้น user | Teerathap sigenergy |

### 9.2 ไอคอน

รูปต้นฉบับถูกครอปเป็นสี่เหลี่ยมจัตุรัสแล้วย่อเป็นหลายขนาดเพื่อใช้เป็น
ไอคอนของส่วนขยาย และวางสำเนาไว้ในโฟลเดอร์ที่ให้บริการเพื่อใช้ตกแต่งภายใน UI

| ปลายทาง | ขนาด | การใช้งาน |
|---------|------|-----------|
| `brand/icon.png` `brand/icon@2x.png` | 256 / 512 | ไอคอนใน Devices and Services |
| `brand/dark_icon.png` `brand/dark_icon@2x.png` | 256 / 512 | ไอคอนโหมดมืด |
| `icon.png` | 256 | ไอคอนส่วนขยาย |
| `frontend/images/teerathap-logo.png` | 512 | โลโก้สำหรับใช้ภายในหน้าจอ |
| `hacs_icon.png` | 256 | ไอคอนใน HACS |

หมายเหตุ แถบเมนูด้านข้างของ Home Assistant รองรับเฉพาะไอคอนชุด mdi
ไม่รองรับรูปภาพกำหนดเอง จึงยังใช้ไอคอน mdi สำหรับรายการในแถบเมนู

### 9.3 โทเคนระบบดีไซน์

```mermaid
%%{init: {'theme':'base','themeVariables':{'primaryColor':'#ffffff','primaryTextColor':'#000000','primaryBorderColor':'#000000','lineColor':'#000000','secondaryColor':'#f2f2f2','tertiaryColor':'#ffffff','fontFamily':'monospace'}}}%%
flowchart TD
    A["theme sigenergy_dark.yaml"] --> B["base deep slate<br/>bg 0f1216 card 1b212b"]
    A --> C["accent เดียว<br/>f5a623 warm amber"]
    A --> D["ink scale<br/>f4f6f8 ถึง 4a525c"]
    A --> E["semantic<br/>good info danger"]
    A --> F["font system<br/>SF Pro และ Noto Sans Thai"]
    G["sigenergy-dashboard.js"] --> H["_cardStyle<br/>gl-regular frosted"]
    G --> I["chrome accent<br/>teal เปลี่ยนเป็น amber"]
    H --> J["dual-edge lighting<br/>ไม่มี drop shadow"]
```

### 9.4 หลักการที่บังคับใช้

| หลักการ | การนำไปใช้ |
|---------|------------|
| สีเน้นเดียว | เปลี่ยนสีเน้น chrome จาก teal เป็น amber f5a623 คงสีเข้ารหัสข้อมูลไว้ |
| frosted glass | `_cardStyle` ใช้พื้นโปร่งแสง blur 28px saturate 1.6 |
| ไม่มี drop shadow บนการ์ด | ใช้ขอบ hairline กับแสง inset แทน |
| ตัวเลข mono tabular | บังคับ `tabular-nums` กับค่าตัวเลขรอง |
| ฟอนต์ระบบ | นำ SF Pro แล้วตามด้วย Noto Sans Thai |
| ไม่มีอิโมจิในโค้ด | ลบอิโมจิออกจากโค้ดและข้อความที่เขียนใหม่ |
