# UBCK Data Model — LLD Repository

> **Mục đích:** Lưu trữ và quản lý các file thiết kế LLD (Low-Level Design) dạng Logical, phục vụ quy trình thiết kế Data Model trên kiến trúc Medallion (Silver layer).

---

## Cấu trúc thư mục

```
ubck_atomic_design/
├── Silver/
│   ├── hld/                                        # HLD — sơ đồ quan hệ Source → Silver
│   │   ├── DCST_relationship_diagram.md
│   │   ├── FIMS_relationship_diagram.md
│   │   ├── FMS_relationship_diagram.md
│   │   └── NHNCK_relationship_diagram.md
│   └── lld/
│       ├── DCST/                                   # Source system: DCST (Dữ liệu Cơ quan Thuế)
│       │   ├── attr_DCST_THONG_TIN_DK_THUE.xlsx
│       │   ├── attr_DCST_TTKDT_NGUOI_DAI_DIEN.xlsx
│       │   ├── attr_DCST_DN_RUI_RO_CAO.xlsx
│       │   ├── attr_DCST_TCT_BAO_CAO.xlsx
│       │   ├── attr_DCST_TCT_BAO_CAO_CHI_TIET.xlsx
│       │   ├── attr_DCST_TCT_TT_CUONG_CHE_NO.xlsx
│       │   ├── attr_DCST_TT_XLY_VI_PHAM.xlsx
│       │   ├── attr_DCST_TCT_TTCCN_HOA_DON.xlsx
│       │   └── attr_DCST_HOA_DON_CHI_TIET.xlsx
│       ├── NHNCK/                                  # Source system: NHNCK (Người Hành Nghề Chứng Khoán)
│       │   └── attr_NHNCK_<TABLE>.csv
│       ├── manifest.csv                            # Danh sách tổng hợp tất cả LLD entities
│       └── ref_shared_entity_classifications.csv   # Phân loại các scheme, code value trong Classification value
├── Source/                                         # Source table definitions
│   ├── DCST_Source_Tables.xlsx
│   ├── FIMS_Source_Tables.xlsx
│   ├── FMS_Source_Tables.xlsx
│   ├── NHNCK_Source_Tables.xlsx
│   └── GAP/                                        # (Mở rộng)
├── system/
│   ├── common/                                     # (Tài nguyên dùng chung — mở rộng)
│   └── templates/
│       └── attr_template.csv                       # Template chuẩn cho file LLD
├── instructions/                                   # (Legacy — không dùng nữa)
└── README.md
```

---

## Quy ước file LLD

### Tên file

Format: `attr_<SOURCE>_<TÊN_BẢNG_NGUỒN>.<ext>`

- DCST: `attr_DCST_THONG_TIN_DK_THUE.xlsx`
- NHNCK: `attr_NHNCK_Professionals.csv`

### Cấu trúc file

**DCST (.xlsx):** Mỗi file chứa 1 hoặc nhiều sheet, mỗi sheet = 1 Silver entity được thiết kế từ bảng nguồn đó.

Ví dụ `attr_DCST_THONG_TIN_DK_THUE.xlsx` gồm 4 sheet:

| Sheet | Silver Entity | Loại |
|---|---|---|
| Registered Taxpayer | Registered Taxpayer | Fundamental (SCD4A) |
| IP Postal Address | Involved Party Postal Address | Shared |
| IP Electronic Address | Involved Party Electronic Address | Shared |
| IP Alt Identification | Involved Party Alternative Identification | Shared |

**NHNCK (.csv):** Mỗi file = 1 Silver entity.

### Các cột trong mỗi file/sheet

| Cột | Mô tả |
|---|---|
| `attribute_name` | Tên attribute trên Silver (tiếng Anh, logical name) |
| `description` | Mô tả gốc từ CSDL nguồn + mô tả bổ sung trên model (nếu có) |
| `data_domain` | 1 trong 12 Data Domain chuẩn |
| `nullable` | true / false |
| `is_primary_key` | true / false |
| `status` | draft / reviewed / approved |
| `source_columns` | Trường nguồn. Format: `<SOURCE>.dbo.<TABLE>.<COLUMN>`. Nhiều nguồn phân cách bằng `\|` |
| `comment` | Ghi chú thiết kế, logic mapping, điểm cần xác nhận SME |

---

## Trạng thái thiết kế (status)

| Status | Ý nghĩa |
|---|---|
| `draft` | Mới thiết kế, chưa review |
| `reviewed` | Đã review với team, có thể còn điểm cần xác nhận SME |
| `approved` | Đã duyệt, sẵn sàng chuyển Engineer |

---

## Workflow

```
1. Design trong Claude chat
   → Mentor hỗ trợ thiết kế từ bảng nguồn → xuất file .csv

2. Review & commit
   → Download file → review → commit vào repo (status = draft)
   → Sau khi review với team → update status = reviewed/approved

3. Sync vào Claude project
   → Bấm "Sync now" trên project knowledge
   → ⚠️ Luôn sync TRƯỚC KHI bắt đầu chat mới

4. Design bảng tiếp theo
   → Claude đọc file LLD đã duyệt để đảm bảo nhất quán FK, naming, pattern
```

---

## Quy tắc thiết kế (tóm tắt)

Chi tiết đầy đủ nằm trong project instruction. Dưới đây là tóm tắt nhanh:

- **LLD Logical** — không bao gồm technical fields (Record Status, Record Insert Date, v.v.)
- **Shared entity** — không có PK surrogate riêng
- **Classification Value** — chỉ có Code, không tạo cặp Id + Code
- **Currency** — bảng SCD1, chỉ có Code
- **FK đến Fundamental entity** — pattern Id + Code
- **Source System Code** — giá trị chi tiết đến tên bảng (ví dụ: `DCST.THONG_TIN_DK_THUE`)
- **PK nguồn** — map vào Entity Code (BK)
- **Description** — giữ nguyên mô tả gốc nguồn + bổ sung mô tả model
- **Data Domain** — dùng đúng 12 domain chuẩn (Currency Amount, không viết tắt Amount)

---

## Danh sách source systems

### DCST — Dữ liệu Cơ quan Thuế

| Bảng nguồn | Silver Entity chính | Status |
|---|---|---|
| THONG_TIN_DK_THUE | Registered Taxpayer | ✅ Designed |
| TTKDT_NGUOI_DAI_DIEN | Taxpayer Representative | ✅ Designed |
| DN_RUI_RO_CAO | High Risk Taxpayer Assessment | ✅ Designed |
| TCT_BAO_CAO | Tax Financial Statement | ✅ Designed |
| TCT_BAO_CAO_CHI_TIET | Tax Financial Statement Item | ✅ Designed |
| TCT_TT_CUONG_CHE_NO | Tax Debt Enforcement Order | ✅ Designed |
| TT_XLY_VI_PHAM | Tax Violation Penalty Decision | ✅ Designed |
| TCT_TTCCN_HOA_DON | Tax Invoice Enforcement Order | ✅ Designed |
| HOA_DON_CHI_TIET | Tax Invoice Enforcement Item | ✅ Designed |

### NHNCK — Nhân Hành Nghề Chứng Khoán

| Bảng nguồn | Silver Entity chính | Status |
|---|---|---|
| Applications | Applications | ✅ Designed |
| Professionals | Professionals | ✅ Designed |
| Organizations | Organizations | ✅ Designed |
| (xem `Silver/lld/manifest.csv` để đầy đủ) | | |

### FIMS, FMS

> Source tables đã có tại `Source/`. LLD chưa thiết kế.

---

## Tham chiếu

| Tài liệu | Vị trí |
|---|---|
| HLD Relationship Diagrams | `Silver/hld/` |
| LLD files | `Silver/lld/<SOURCE>/` |
| Manifest (tổng hợp entities) | `Silver/lld/manifest.csv` |
| Shared Entity Classifications | `Silver/lld/ref_shared_entity_classifications.csv` |
| Data Dictionary template | `system/templates/attr_template.csv` |
| Tài liệu thiết kế CSDL (D02) | Claude project knowledge |
| Data Dictionary tổng hợp (Excel) | Claude project knowledge |
| Modules đào tạo (M1–M12) | Claude project knowledge |
