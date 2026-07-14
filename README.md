# Network Fundamental Labs

Packet Tracer labs — IP addressing, subnetting, VLSM  
Course: Networking Fundamentals (David Bombal) · Week 1–2 of cloud engineering roadmap

---

## Lab 01 — Subnetting & VLSM (192.168.1.0/24)

### Requirements | โจทย์

**EN**
1. Break up `192.168.1.64/26` into as many subnets as possible, 8 hosts per subnet
2. Allocate the first new subnet to Site 3
3. Manually configure all devices — router uses the **last** IP in the subnet, hosts start from the **first** IP
4. Subnet the last remaining block from `192.168.1.64/26` with `/30` masks and assign to the serial links
5. Verify PCs can reach `cisco.com` and `facebook.com`

**TH**
1. แตก `192.168.1.64/26` ให้ได้ subnet มากที่สุด โดยแต่ละวงรองรับ 8 host
2. subnet แรกที่ได้ → Site 3
3. ตั้งค่าเองทุกอุปกรณ์ — router ใช้ IP **สุดท้าย**ของวง, host ไล่จาก IP **แรก**
4. เอาก้อนสุดท้ายจาก `192.168.1.64/26` มาซอย `/30` ให้ serial link
5. ตรวจสอบว่า PC เข้า `cisco.com` และ `facebook.com` ได้

---

### Subnet Plan | แผนการแบ่ง subnet

| Site / Link | Prefix | Network | Broadcast | Router IP | Host Range | Usable |
|---|---|---|---|---|---|---|
| Site 1 | /26 | 192.168.1.0 |192.168.1.63 |192.168.1.62 | .1 - .62| 62|
| Site 2 | /26 | 192.168.1.128 |192.168.1.191 |192.168.1.190 | .129 - .190| 62|
| Site 3 | /28 | 192.168.1.64| 192.168.1.79| 192.168.1.78|.65 - .78 | 14|
| R1 ↔ Internet | /30 |192.168.1.112 | 192.168.1.115|192.168.1.114 | .113-.114| 2|
| R2 ↔ Internet | /26 |192.168.1.192 | 192.168.1.255|192.168.1.254 | .193-.254|62 |
| R4 ↔ Internet | /30 | 192.168.1.116| 192.168.1.119|192.168.1.118 | .117-.118|2 |

> เติมของจริงจาก lab ให้ครบทุกช่อง

---

### Design Rationale | เหตุผลการออกแบบ

**Why /28 for an 8-host subnet | ทำไม 8 host ต้องใช้ /28**

**EN** — 8 usable hosts require `2^h − 2 ≥ 8`. `/29` gives `2^3 − 2 = 6` — not enough. `/28` gives `2^4 − 2 = 14` — the smallest prefix that fits. The network and broadcast addresses are reserved, which is why 2 is always subtracted.

**TH** — ต้องการ 8 host ใช้ได้จริง ต้อง `2^h − 2 ≥ 8` · `/29` ได้ `2^3 − 2 = 6` ไม่พอ · `/28` ได้ `2^4 − 2 = 14` คือ prefix เล็กสุดที่พอ · ที่ลบ 2 เพราะ network + broadcast จองไว้ ตั้งให้ host ไม่ได้

**Why /30 for serial links | ทำไม serial link ใช้ /30**

**EN** — A point-to-point link needs exactly 2 addresses. `/30` gives `2^2 − 2 = 2` — an exact fit with zero waste. Using a larger subnet here would strand usable addresses.

**TH** — link แบบ point-to-point ต้องการแค่ 2 IP · `/30` ให้ `2^2 − 2 = 2` พอดีเป๊ะ ไม่เหลือทิ้ง · ถ้าใช้วงใหญ่กว่านี้ = IP เสียเปล่า

**Why the router takes the last IP | ทำไม router ใช้ IP สุดท้าย**

**EN** — Required by the lab spec. In production, either the first or last usable address is a valid convention — what matters is that it is applied consistently across all subnets.

**TH** — โจทย์กำหนด · ในงานจริงใช้ IP แรกหรือสุดท้ายก็ได้ ประเด็นคือต้อง**ทำเหมือนกันทุกวง** ไม่ใช่สลับไปมา

**Why largest-first allocation | ทำไมต้องจัดวงใหญ่ก่อน**

**EN** — VLSM must allocate the largest block first; otherwise smaller subnets fragment the address space and leave gaps too small to hold the larger requirement.

**TH** — VLSM ต้องจัดวงใหญ่ก่อน ไม่งั้นวงเล็กจะไปคร่อมช่วงที่วงใหญ่ต้องใช้ เกิดช่องว่างที่ใช้ต่อไม่ได้

---

### Topology | ผังเครือข่าย

![topology](labs/Topology)

---

### Verification | การตรวจสอบ

**End-to-end connectivity from PC6**

![ping](labs/ping)

**EN** — PC6 successfully resolves and reaches `cisco.com`, `facebook.com`, and `8.8.8.8` with 0% loss. `TTL=126` confirms the packets crossed **2 routers** — the default TTL of 128 is decremented once per hop. This proves inter-site routing through the serial links is working, not just local connectivity.

**TH** — PC6 ping ถึง `cisco.com`, `facebook.com`, `8.8.8.8` ได้ครบ 0% loss · `TTL=126` ยืนยันว่า packet ผ่าน **router 2 ตัว** (TTL เริ่มที่ 128 ลด 1 ทุก hop) = routing ข้าม site ผ่าน serial link ทำงานจริง ไม่ใช่แค่ต่อได้ในวงตัวเอง

---
> **⚠️ Known deviation | จุดที่ยังไม่ตรงโจทย์**
>
> **EN** — R2's serial link was left as `/26` from the pre-configured topology and was not re-subnetted to `/30` as requirement #4 specifies. A point-to-point link uses 2 addresses, so this strands 60 usable addresses. To be corrected in a follow-up pass.
>
> **TH** — ขา serial ของ R2 ยังเป็น `/26` ตามที่ topology ตั้งมาให้ ไม่ได้ซอยเป็น `/30` ตามโจทย์ข้อ 4 · link แบบ point-to-point ใช้แค่ 2 IP → เสีย IP เปล่า 60 ตัว · จะกลับมาแก้รอบถัดไป

---

### Key Formulas | สูตรที่ใช้

```
block size = 256 − (non-255 octet of mask)
network    = largest multiple of block ≤ IP
broadcast  = next network − 1
usable     = network+1 → broadcast−1

subnet count = 2^(new prefix − old prefix)   ← no −2
host count   = 2^(32 − prefix) − 2           ← subtract 2
```

---

### Files | ไฟล์ในโปรเจกต์

```
labs/
    Subnetting lab 2 - intial.pkt
    Topology
    Ping
```
