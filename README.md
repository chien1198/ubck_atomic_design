# UBCK Data Model — LLD Repository

> **Mục đích:** Lưu trữ và quản lý các file thiết kế LLD (Low-Level Design) dạng Logical, phục vụ quy trình thiết kế Data Model trên kiến trúc Medallion (Silver layer).

---

## Cấu trúc thư mục

```
ubck-data-model/
├── instructions/
│   └── DCST_relationship_diagram.md      # HLD — sơ đồ quan hệ Source → 
Source
├── DCST_Source_Tables.xlsx
├── (source khác)/                        # Mở rộng khi có thêm source 
Silver
├── lld/
│   ├── DCST/                             # Source system: DCST (Dữ liệu Cơ quan Thuế)
│   │   ├── attr_THONG_TIN_DK_THUE.xlsx
│   │   ├── attr_TTKDT_NGUOI_DAI_DIEN.xlsx
│   │   ├── attr_DN_RUI_RO_CAO.xlsx
│   │   ├── attr_TCT_BAO_CAO.xlsx
│   │   ├── attr_TCT_BAO_CAO_CHI_TIET.xlsx
│   │   ├── attr_TCT_TT_CUONG_CHE_NO.xlsx
│   │   ├── attr_TT_XLY_VI_PHAM.xlsx
│   │   ├── attr_TCT_TTCCN_HOA_DON.xlsx
│   │   └── attr_HOA_DON_CHI_TIET.xlsx
│   └── (source khác)/                    # Mở rộng khi có thêm source system
├── templates/
│   └── attr_template.xlsx                # Template chuẩn cho file LLD
└── README.md
```

---

## Quy ước file LLD

### Tên file

Format: `attr_<TÊN_BẢNG_NGUỒN>.xlsx`

Ví dụ: `attr_THONG_TIN_DK_THUE.xlsx`

### Cấu trúc file Excel

Mỗi file chứa **1 hoặc nhiều sheet**, mỗi sheet = 1 Silver entity được thiết kế từ bảng nguồn đó.

Ví dụ `attr_THONG_TIN_DK_THUE.xlsx` gồm 4 sheet:

| Sheet | Silver Entity | Loại |
|---|---|---|
| Registered Taxpayer | Registered Taxpayer | Fundamental (SCD4A) |
| IP Postal Address | Involved Party Postal Address | Shared |
| IP Electronic Address | Involved Party Electronic Address | Shared |
| IP Alt Identification | Involved Party Alternative Identification | Shared |

### Các cột trong mỗi sheet

| Cột | Mô tả |
|---|---|
| `attribute_name` | Tên attribute trên Silver (tiếng Anh, logical name) |
| `description` | Mô tả gốc từ CSDL nguồn + mô tả bổ sung trên model (nếu có) |
| `data_domain` | 1 trong 12 Data Domain chuẩn |
| `nullable` | true / false |
| `is_primary_key` | true / false |
| `status` | draft / reviewed / approved |
| `source_columns` | Trường nguồn. Format: `DCST.dbo.<TABLE>.<COLUMN>`. Nhiều nguồn phân cách bằng `\|` |
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
   → Mentor hỗ trợ thiết kế từ bảng nguồn → xuất file .xlsx

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

## Danh sách bảng nguồn DCST

| Bảng nguồn | Silver Entity chính | Status |
|---|---|---|
| THONG_TIN_DK_THUE | Registered Taxpayer | ✅ Designed |
| TTKDT_NGUOI_DAI_DIEN | Taxpayer Representative | ⬜ Pending |
| DN_RUI_RO_CAO | High Risk Taxpayer Assessment | ⬜ Pending |
| TCT_BAO_CAO | Tax Financial Statement | ⬜ Pending |
| TCT_BAO_CAO_CHI_TIET | Tax Financial Statement Item | ⬜ Pending |
| TCT_TT_CUONG_CHE_NO | Tax Debt Enforcement Order | ⬜ Pending |
| TT_XLY_VI_PHAM | Tax Violation Penalty Decision | ⬜ Pending |
| TCT_TTCCN_HOA_DON | Tax Invoice Enforcement Order | ⬜ Pending |
| HOA_DON_CHI_TIET | Tax Invoice Enforcement Item | ⬜ Pending |

---

## Tham chiếu

| Tài liệu | Vị trí |
|---|---|
| HLD Relationship Diagram | `instructions/DCST_relationship_diagram.md` |
| Data Dictionary template | `templates/attr_template.xlsx` |
| Tài liệu thiết kế CSDL (D02) | Claude project knowledge |
| Data Dictionary tổng hợp (Excel) | Claude project knowledge |
| Modules đào tạo (M1–M12) | Claude project knowledge |
