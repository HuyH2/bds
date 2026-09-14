# KẾ HOẠCH MOCKUP APP ROLE HR
**Dự án:** BĐS Sales Suite — mở rộng sang phân hệ Nhân sự
**Ngày soạn:** 2026-09-14 • **Căn cứ:** `SPEC.md` + mockup `mobile.html` (role Sale, 6.796 dòng)

---

# PHẦN A — TỔNG HỢP MOCKUP ROLE SALE (HIỆN TRẠNG)

## A1. Kiến trúc file
| Hạng mục | Hiện trạng |
| :--- | :--- |
| File | `mobile.html` — single-file, ~400KB, tự chứa (HTML + CSS + JS + mock data) |
| Entry | `index.html` = màn đăng nhập (luôn là điểm bắt đầu) |
| CSS | Tailwind CDN (`cdn.tailwindcss.com`) + `tailwind.config` inline mở rộng brand colors |
| Icon | Material Symbols Outlined (Google Fonts) |
| Font | Inter 400/500/600/700/800 |
| Build | Không có build step — mở file là chạy |

## A2. Khung ứng dụng (App Shell)
```
#mobileAppFrame  (max-w-[430px], 100dvh, khung iPhone khi xem trên desktop)
├── #appHeader     — chỉ hiện ở tab "Dự án": avatar + lời chào + badge hạng + 🔔
├── #subHeader     — dùng chung mọi màn còn lại: [← Back | icon tab] + title + subtitle + action slot
├── #detailHeader  — riêng màn Chi tiết căn: back + tên căn + share + avatar
├── <main>         — router, chứa toàn bộ `.mobile-tab-view`
├── #dtActionBar   — thanh hành động nổi (chỉ màn chi tiết căn), neo trên bottom nav
└── #bottomNav     — 4 tab + 1 FAB tròn ở giữa
```
Quy tắc CSS đã chốt: chrome (`header`/`nav`) `position: absolute` trong frame, `z-index: 100`; `main` có `padding-top: 5rem` + `padding-bottom: 7rem`; `.mobile-tab-view { display: none !important }` để chống 4 màn chồng nhau lúc Tailwind CDN chưa nạp xong.

## A3. Điều hướng
- `SCREEN_REGISTRY` — object khai báo 16 màn: `{ viewId, title, tab?, isMainTab? }`
- `openScreen(name, options)` — ẩn tất cả view, hiện view đích, tự quản header/back/bottom nav highlight, `options.restore` để khôi phục trạng thái cây phân cấp
- `goBackScreen()` — stack `screenHistory[]`; tab chính reset stack, màn con push vào
- Quy tắc: **tab chính KHÔNG có nút Back** (hiện icon tab thay thế); màn con **luôn có Back**

**Bottom nav hiện tại:** `[Dự án] [Sự kiện] (◉ Chấm công) [Quản lí] [Cài đặt]`

## A4. Danh mục màn hình
| Screen key | View ID | Vai trò |
| :--- | :--- | :--- |
| `home` / `inventory` | `view-inventory` | Bảng hàng 4 cấp: Dự án → Phân khu → Tòa → Tầng (accordion) → Căn |
| `events` | `view-events` | Danh sách sự kiện & dự án (card ảnh trái / mô tả phải) |
| `checkin` | `view-checkin` | 2 panel: Chấm công (GPS + selfie watermark) / Lịch sử công |
| `my-units` | `view-my-units` | 2 sub-tab: Quản lí dự án (cây 4 cấp) / Cá nhân (deal của tôi) |
| `settings` | `view-settings` | Hồ sơ, QR cá nhân, toggle thông báo, tài khoản |
| `detail` | `view-detail` | Chi tiết căn: hero → giá → thông số → bóc tách chi phí → tiến độ TT → ghi chú |
| `event-detail` | `view-event-detail` | Chi tiết sự kiện / dự án + đăng ký + thiệp mời |
| `customer-detail` | `view-customer-detail` | Hồ sơ khách + stepper phễu + lịch sử tương tác |
| `mortgage` | `view-mortgage` | Máy tính vay ngân hàng |
| `saleskit` | `view-saleskit` | Kho tài liệu (chip lọc + share Zalo) |
| `explanation` | `view-explanation` | **Form đơn giải trình chấm công / xin nghỉ** (gửi lên TP duyệt) |
| `qr-scanner` | `view-qr-scanner` | Quét QR check-in sự kiện |
| `personal-qr` | `view-personal-qr` | QR điểm danh cá nhân |
| `notifications` | `view-notifications` | Trung tâm thông báo + chip lọc |
| `edit-profile` | `view-edit-profile` | Sửa hồ sơ (trường công ty cấp bị khóa) |

## A5. Design system đã chốt
```
--primary:       #0f3d64   (navy — nút chính, text nhấn, active tab)
--primary-light: #1e588a   (gradient pair)
--secondary:     #0284c7   (sky — trạng thái OK, link, badge live)
--surface:       #f8fafc   (nền app)
--card:          #ffffff + border-slate-200 + shadow-xs
Bo góc: rounded-xl (nút/input) · rounded-2xl (card) · rounded-3xl (card lớn/modal)
Chữ: text-[10px]/[11px] meta · text-xs body & nút · text-sm tiêu đề · font-bold→extrabold
Nút: h-11 · active:scale-95 · transition-all
Số liệu: font-mono
```
**Mã màu trạng thái căn:** trống `emerald-500` · giữ chỗ `amber-400` · đã cọc `orange-500` · đã bán `rose-500`

## A6. Pattern UI tái sử dụng được
1. **Drilldown 4 cấp** — `selectInventoryProject/Zone/Tower` + `goBackInventoryHierarchy()` + `setSubHeaderContext()`
2. **Accordion tầng** — `renderInventoryFloorsAccordion()`, `toggleAllInventoryFloors()`
3. **Sub-tab 2 cột** — `mgmtTabs` / `checkinTabs` (`MGMT_TAB_ACTIVE` / `MGMT_TAB_IDLE`)
4. **Chip lọc trạng thái** — `filterInventoryUnitStatus()`, `setMyFloorFilter()`
5. **Toggle Lưới / Danh sách** — `setInventoryRoomViewMode()`
6. **Bottom-sheet modal** — `openQuickUnitModal()`, modal ghi chú, modal đổi trạng thái, modal thiệp mời
7. **Toast** — `showToast(msg, icon)`
8. **Empty state** — `myEmptyStateHtml(message, actionHtml)`
9. **Form động + validate** — `openDealForm` / `validateDealForm` / `showDealFormErrors`
10. **Stepper phễu** — `renderCustomerStepper(currentStage)`
11. **Camera giả lập + watermark** — `takeCheckinPhoto()` vẽ đè khối thông tin GPS/giờ/nhân viên

## A7. Mô hình dữ liệu mock
| Const | Nội dung |
| :--- | :--- |
| `USER_PROFILE` | Hồ sơ người dùng hiện tại |
| `INVENTORY_PROJECTS_CONFIGS` | Cây Dự án → Phân khu → Tòa |
| `INVENTORY_TOWERS_DATABASE` | Tầng + căn chi tiết |
| `PROJECT_LISTINGS` / `EVENT_LISTINGS` / `EVENTS_DATA` | Thẻ danh sách + chi tiết |
| `CUSTOMERS_DATABASE` | Khách hàng CRM |
| `SEED_DEALS` + `DEAL_STORAGE_KEY` | Giao dịch — **persist qua localStorage** |
| `REGISTERED_STORAGE_KEY` | Dự án đã đăng ký |

Cơ chế: `readStorage/writeStorage` → seed nếu rỗng → mọi thao tác ghi lại → `refreshAfterDealChange()` render lại màn đang mở.

---

# PHẦN B — KẾ HOẠCH MOCKUP ROLE HR

## B1. Chân dung người dùng
**Chuyên viên / Trưởng phòng Nhân sự sàn BĐS.** Khác Sale ở chỗ: không bán căn, mà **chủ yếu quản lý công của Sale** — với đặc thù ngành BĐS là nhân sự biến động cực nhanh (CTV vào/ra liên tục), làm việc phân tán (văn phòng, nhà mẫu, sự kiện), và lương gắn chặt với ngày công + hoa hồng.

Nghiệp vụ chính, suy ra từ `SPEC.md` (mục 2.1, 2.2, 2.7) và từ những gì Sale *gửi lên* trong mockup hiện tại:
1. **Nhận & duyệt đơn** — Sale bấm "Gửi Đơn Cho Trưởng Phòng Phê Duyệt" ở `view-explanation` → hiện chưa có ai nhận. HR là đầu bên kia.
2. **Giám sát chấm công** — quân số real-time, đi muộn, quên chấm, đối soát ảnh selfie + GPS, xuất Excel tính lương.
3. **Hồ sơ nhân sự** — cây tổ chức, hợp đồng, hạng (F1 PRO...), điều chuyển team, off-board.
4. **Đào tạo & sự kiện nội bộ** — workshop CTV, onboarding, điểm danh QR.
5. **Lương & hoa hồng** — ngày công × hệ số + hoa hồng từ deal của Sale.
6. **HR cũng là nhân viên** — vẫn phải tự chấm công.

**Ngoài phạm vi:** tuyển dụng (tin đăng, CV, phễu ứng viên, onboarding) — đã bỏ khỏi app HR.

## B2. Nguyên tắc thiết kế
- **Không dùng khối tổng hợp / cảnh báo**: không có thẻ "Quân số hôm nay", ô số liệu tổng, danh sách cảnh báo hay ghi chú cảnh báo HĐ — các màn đi thẳng vào danh sách / dữ liệu chi tiết.
- **Giữ nguyên 100% app shell, design system, router, toast, modal** của `mobile.html`. HR phải trông như cùng một app, chỉ đổi nội dung nghiệp vụ.
- **Tái sử dụng pattern, không tái sử dụng dữ liệu**: cây 4 cấp bảng hàng → cây tổ chức; form deal → form duyệt đơn.
- **Mock có trạng thái**: dùng lại cơ chế `localStorage` để duyệt/từ chối đơn còn lưu được sau khi F5 — demo thuyết phục hơn nhiều.

## B3. Điều hướng đề xuất

**Bottom nav HR:** `[Nhân sự] [Duyệt phép] (◉ Chấm công) [Nghiệp vụ] [Cài đặt]`

| Tab | Screen key | Đối chiếu bên Sale |
| :--- | :--- | :--- |
| 1. Nhân sự | `staff` | ← `home` / `inventory` (drilldown 4 cấp) |
| 2. Duyệt phép | `approve` | ← `my-units` (`view-hr-ops`) |
| ◉ Chấm công | `checkin` | **giữ nguyên** |
| 3. Nghiệp vụ | `business` | ← nhóm lối tắt "Nghiệp vụ HR" trước đây ở Cài đặt |
| 4. Cài đặt | `settings` | **giữ nguyên** (đổi badge chức danh) |

`SCREEN_REGISTRY` của HR:
```js
'staff'            : { viewId: 'view-staff',        title: 'Nhân Sự',    tab: 'nhan-su',   isMainTab: true }
'approve'          : { viewId: 'view-hr-ops',       title: 'Duyệt Phép', tab: 'duyet-phep', isMainTab: true }
'checkin'          : { viewId: 'view-checkin',      title: 'Chấm Công',  tab: 'cham-cong', isMainTab: true }
'business'         : { viewId: 'view-business',     title: 'Nghiệp Vụ',  tab: 'nghiep-vu', isMainTab: true }
'settings'         : { viewId: 'view-settings',     title: 'Cài Đặt',    tab: 'cai-dat',   isMainTab: true }
'staff-detail'     : { viewId: 'view-staff-detail',     title: 'Hồ Sơ Nhân Sự' }
'request-detail'   : { viewId: 'view-request-detail',   title: 'Chi Tiết Đơn' }
'timesheet'        : { viewId: 'view-timesheet',        title: 'Bảng Công' }
'timesheet-detail' : { viewId: 'view-timesheet-detail', title: 'Bảng Công Cá Nhân' }
'training'         : { viewId: 'view-training',         title: 'Đào Tạo' }
'announce'         : { viewId: 'view-announce',         title: 'Gửi Thông Báo' }
'payroll'          : { viewId: 'view-payroll',          title: 'Bảng Lương' }
'qr-scanner' / 'personal-qr' / 'notifications' / 'edit-profile'  — giữ nguyên
```

## B4. Đặc tả từng màn

### TAB 1 — NHÂN SỰ (`view-staff`)
Drilldown 4 cấp, clone thẳng cơ chế bảng hàng:
```
Chi nhánh (Sala / Đồng Khởi / Hà Nội)
  → Phòng ban (Kinh doanh 1, Kinh doanh 2, Marketing, Back-office)
    → Team (Team Nam, Team Hương...)
      → Danh sách nhân viên  [Lưới avatar ⇄ Danh sách]
```
- Search bar sticky (clone `#invSearchHeader`): tìm theo tên / mã NV / SĐT.
- Thẻ nhỏ `Bảng công` ở đầu cấp chi nhánh → mở `view-timesheet`.
- **Chip lọc trạng thái hôm nay** thay cho 4 trạng thái căn:
  `Đã chấm công` 🟢 emerald · `Đi muộn` 🟡 amber · `Chưa chấm` 🟠 orange · `Nghỉ phép` 🔵 sky · `Vắng KP` 🔴 rose
- Header team hiển thị badge `28/32 Đã chấm công` (clone `#invFloorsAvailableBadge`).
- Accordion theo team, mỗi nhân viên là 1 ô avatar + tên + chấm màu trạng thái.
- Bấm 1 nhân viên → **bottom-sheet nhanh** (clone `openQuickUnitModal`): ảnh, chức danh, giờ check-in, vị trí, nút `Xem hồ sơ` / `Gọi` / `Nhắc chấm công`.

### `view-staff-detail` — Hồ sơ nhân sự
Clone bố cục màn Chi tiết căn (hero → khối → khối → action bar):
1. **Hero**: avatar lớn, họ tên, mã NV, chức danh, badge hạng, trạng thái làm việc (Thử việc / Chính thức / CTV / Đã nghỉ).
2. **Thông tin cơ bản**: ngày sinh, SĐT, email, CCCD (che giữa), địa chỉ, người liên hệ khẩn cấp.
3. **Công việc**: ngày vào làm, thâm niên, chi nhánh/phòng/team, quản lý trực tiếp, loại HĐ + ngày hết hạn, link sang `view-timesheet-detail`.
4. **Tài liệu**: HĐLĐ, CCCD, bằng cấp, chứng chỉ môi giới (clone card Sales Kit).
5. **Action bar**: `Điều chuyển team` · `Sửa hồ sơ` · `Kết thúc HĐ` (modal xác nhận).

### FAB — CHẤM CÔNG (`view-checkin`)
Giữ nguyên y hệt Sale (HR cũng là nhân viên). Chỉ đổi danh sách địa điểm sang các điểm HR hay đứng: trụ sở, chi nhánh, hội trường đào tạo.

### TAB 2 — DUYỆT PHÉP (`view-hr-ops`)

**Tab Duyệt phép** — *màn quan trọng nhất của HR*
- **Cấp 1 — Chi nhánh**: mỗi thẻ chi nhánh có tổng số đơn + nhãn `N chờ duyệt` (hoặc `Đã xử lý hết` / `Chưa có đơn`).
- **Cấp 2 — Đơn của chi nhánh** (có nút Back về cấp 1): chip lọc `Chờ duyệt (n)` · `Đã duyệt` · `Từ chối` · `Tất cả`, đếm trong phạm vi chi nhánh.
- Card đơn: avatar + tên + team | loại đơn (Giải trình quên chấm / Đi muộn / Nghỉ phép / Công tác / Điều chuyển) | thời gian phát sinh | trích lý do | badge màu trạng thái | 2 nút nhanh **✓ Duyệt** / **✕ Từ chối**.
- Badge số đơn chờ duyệt hiển thị trên bottom nav (clone `#navDealCount`).
- `view-request-detail`: full lý do, ảnh minh chứng, **đối chiếu dữ liệu chấm công thực tế** (giờ GPS, khoảng cách, ảnh selfie hôm đó), lịch sử vi phạm 3 tháng gần nhất, ô nhập ghi chú phê duyệt, nút Duyệt / Từ chối / Yêu cầu bổ sung.
- **Persist localStorage** (`hr.requests.v1`) — duyệt xong F5 vẫn giữ.

### MÀN BẢNG CÔNG (`view-timesheet`) — mục nhỏ trong tab Nhân sự
Mở từ thẻ `Bảng công` ở đầu tab Nhân sự, có nút Back.
- Chọn tháng + chọn phòng ban.
- **Lưới công tháng**: hiện **toàn bộ nhân sự** (lọc phòng ban + ô tìm theo tên, không phân biệt dấu); hàng = nhân viên, cột = ngày, ô = màu trạng thái; khung cuộn cả ngang lẫn dọc với thanh cuộn hiện rõ, dòng số ngày và cột tên cố định khi cuộn.
- Nút `Xuất Excel` (mock → toast).
- Bấm 1 nhân viên → `view-timesheet-detail`: lịch tháng cá nhân + list ảnh check-in kèm watermark GPS.

### TAB 3 — NGHIỆP VỤ (`view-business`)
Danh sách lối vào: `Bảng lương & hoa hồng` · `Đào tạo & sự kiện nội bộ` · `Thông báo nội bộ` · `Quét QR điểm danh` (chuyển từ nhóm "Nghiệp vụ HR" trong Cài đặt sang).

### TAB 4 — CÀI ĐẶT
Giữ nguyên, đổi: badge `F1 PRO` → `TRƯỞNG PHÒNG NS`, thêm nhóm **Cấu hình HR**: ca làm việc & giờ vào/ra, bán kính geofence từng địa điểm, hạn mức phép năm, người duyệt cấp 2.

### Màn phụ
- `view-payroll` — **Bảng lương & hoa hồng tháng**: lương cứng + ngày công thực tế + hoa hồng (đọc `SEED_DEALS`, hệ số `COMMISSION_RATE` 2%) + phụ cấp − khấu trừ (BHXH, thuế TNCN) → **thực nhận**. Sắp xếp giống tab Nhân sự: ô tìm kiếm (tên / mã NV / SĐT) ở cấp đầu + drilldown Chi nhánh → Phòng ban → Team → Nhân sự (mỗi thẻ nhóm có số nhân sự + tổng thực nhận; Back lùi từng cấp); không có khối tổng quỹ lương; nút `Xuất Excel` cạnh tiêu đề danh sách, xuất theo cấp đang xem (mock → toast). Bấm thẻ lương 1 nhân sự → `view-payroll-detail` (phiếu lương): thanh cuộn ngang chấm công từng ngày trong tháng (ô màu theo trạng thái, ngày chưa tới viền nét đứt, tự cuộn tới hôm nay) + lương cứng, ngày công → lương theo công, phụ cấp; hoa hồng bóc tách từng căn đã chốt (dự án, mã căn, ngày, giá trị, +2%); khấu trừ BHXH/BHYT/BHTN & thuế TNCN; tổng thu nhập → **thực nhận**; nút `Hồ sơ`, `Bảng công`, `Xuất PDF`, `Gửi nhân sự`.
- `view-training` — HR **tạo và sửa** đào tạo / sự kiện nội bộ qua `view-training-form` (loại, tên, ngày & giờ, địa điểm, số chỗ tối đa, nhãn, mô tả; kiểm tra thiếu thông tin, ngày/giờ kết thúc sau bắt đầu, số chỗ không ít hơn số đã đăng ký; lưu `hr.trainings.v1`). Mỗi thẻ: `Sửa` · `Mời` · `Điểm danh` (mở `qr-scanner`), thanh tỷ lệ đăng ký.
- **Thông báo nội bộ**: `view-announce-history` — danh sách thông báo **đã gửi** (mới nhất trước; tiêu đề, đối tượng nhận, thời gian, người gửi) + nút `Soạn thông báo mới` → `view-announce` (tiêu đề, nội dung, đối tượng, xem trước); gửi xong lưu vào lịch sử (`hr.announcements.v1`) và quay về danh sách. Bấm 1 thông báo → `view-announce-detail` (nội dung đầy đủ, gửi tới, thời gian, người gửi).
- `view-notifications` — đổi nhóm chip: `Đơn từ` · `Chấm công` · `HĐLĐ sắp hết hạn`.

## B5. Mô hình dữ liệu mock cần dựng
```js
HR_USER_PROFILE          // Trưởng phòng NS
HR_ORG_CONFIGS           // Chi nhánh → Phòng ban → Team   (clone INVENTORY_PROJECTS_CONFIGS)
HR_STAFF_DATABASE        // ~80-120 nhân viên: mã, tên, avatar, chức danh, team,
                         // ngày vào, loại HĐ, hạn HĐ, trạng thái hôm nay, công tháng, phép còn
HR_REQUESTS              // Đơn từ: loại, người gửi, thời gian, lý do, minh chứng, trạng thái, người duyệt
HR_TRAININGS             // Khoá đào tạo / sự kiện nội bộ
HR_PAYROLL               // Bảng lương tháng: lương cứng, công, hoa hồng, khấu trừ, thực nhận
HR_ATTENDANCE_MATRIX     // Ma trận công tháng (nhân viên × ngày)
HR_NOTIFICATIONS

// Persist
'hr.requests.v1'   — kết quả duyệt đơn
'hr.staff.v1'      — thay đổi hồ sơ / điều chuyển
```
Dữ liệu **phải khớp** với mockup Sale: `Nguyễn Quang Huy • NV-0012 • Chi nhánh Sala • F1 PRO` phải có mặt trong `HR_STAFF_DATABASE`, và đơn giải trình mẫu trong hộp duyệt phải đúng nội dung Sale gửi ở `view-explanation`.

## B6. Bảng ánh xạ hàm (tái sử dụng từ `mobile.html`)
| Hàm Sale | Hàm HR |
| :--- | :--- |
| `selectInventoryProject / Zone / Tower` | `selectBranch / selectDepartment / selectTeam` |
| `renderInventoryFloorsAccordion` | `renderTeamStaffAccordion` |
| `filterInventoryUnitStatus` | `filterStaffAttendanceStatus` |
| `setInventoryRoomViewMode` | `setStaffViewMode` (Lưới avatar ⇄ Danh sách) |
| `openQuickUnitModal` | `openQuickStaffModal` |
| `openUnitDetail` | `openStaffDetail` |
| `openDealForm / validateDealForm` | `openRequestReviewForm / validateRequestForm` |
| `setMgmtMode` | — (Duyệt phép và Bảng công là 2 màn riêng) |
| `refreshDealCounters` | `refreshPendingRequestCounter` |
| `myEmptyStateHtml` | giữ nguyên |
| `showToast` / `openScreen` / `goBackScreen` | giữ nguyên |

## B7. Phương án file — **khuyến nghị**
Tạo **`hr.html`** riêng, copy app shell + design system + router từ `mobile.html`, và đổi `index.html` thành **màn chọn vai trò** (2 thẻ: Sale / HR).

*Lý do:* `mobile.html` đã 400KB/6.796 dòng; nhồi thêm 2 role vào một file sẽ khó sửa và dễ vỡ mockup Sale đang chạy ổn. Mockup không cần DRY — cần an toàn và dễ demo song song 2 cửa sổ.

*Phương án thay thế* (nếu muốn 1 file duy nhất, có nút đổi role ngay trong app): tách `SCREEN_REGISTRY` + `bottomNav` theo `CURRENT_ROLE`, chi phí refactor ~1 ngày và có rủi ro cho luồng Sale.

### B7.1. Đổi vai trò ngay trong app (bấm avatar) — vẫn làm được với 2 file
2 file **không** cản trở việc đổi role. Đổi role = `location.href = 'hr.html'` → tải lại trang, gần như tức thì với file local, và `localStorage` giữ nguyên nên không mất dữ liệu mock.

**Luồng đề xuất — bấm avatar mở bottom-sheet "Tài khoản & Vai trò":**
```
┌─────────────────────────────────┐
│ [avatar] Nguyễn Quang Huy       │
│          NV-0012 • CN Sala      │
├─────────────────────────────────┤
│ 👤 Chỉnh sửa hồ sơ           ›  │
├─────────────────────────────────┤
│ ĐANG ĐĂNG NHẬP VỚI VAI TRÒ      │
│ 🏢 Sale / CTV               ✓   │  ← role hiện tại
│ 👥 Nhân sự (HR)                 │  ← bấm → chuyển hr.html
└─────────────────────────────────┘
```
Giữ được cả 2 việc: avatar vẫn vào được hồ sơ (như hiện tại), đồng thời là chỗ đổi role.

**Cần thêm:**
| Hạng mục | Nội dung |
| :--- | :--- |
| `ROLE_STORAGE_KEY` | `'bds.role.v1'` — nhớ role vừa chọn |
| `switchRole(role)` | `writeStorage(ROLE_STORAGE_KEY, role)` → `location.href = role === 'hr' ? 'hr.html' : 'mobile.html'` |
| `openRoleSheet()` | Bottom-sheet, clone khung modal sẵn có (`quickCustomerStatusModal`) |
| `index.html` | Màn đăng nhập: luôn hiện form + 2 tài khoản demo; vào trang là xoá phiên cũ |
| Badge header | Thay `F1 PRO` bằng badge vai trò (`SALE` / `HR`) để luôn biết đang ở role nào |

**Chi phí:** ~1 giờ, code giống hệt nhau ở cả 2 file (copy-paste 1 block ~40 dòng). Đưa vào **P0**.

*Lưu ý:* khi chuyển file, state trong RAM (cây phân cấp đang mở, tab con đang chọn) bị reset về mặc định. Dữ liệu đã lưu (`bds.deals.v1`, `hr.requests.v1`...) **không mất**. Với mockup thì chấp nhận được; nếu muốn giữ nguyên cả vị trí đang đứng thì mới cần gộp 1 file.

### B7.2. Đăng nhập (cập nhật 2026-09-14)
`index.html` là màn đăng nhập. Mockup không có máy chủ nên dùng 2 tài khoản demo, mật khẩu chung `demo123`:

| Tài khoản | Người dùng | Vai trò → file |
| :--- | :--- | :--- |
| `huy.nguyen` | Nguyễn Quang Huy • NV-0012 • CN Sala | Sale / CTV → `mobile.html` |
| `huong.tran` | Trần Thị Thanh Hương • NS-0004 • TP Nhân sự | Nhân sự (HR) → `hr.html` |

- Bấm thẻ tài khoản demo → tự điền form và đăng nhập luôn; nhập tay sai → báo lỗi ngay trên form.
- Phiên lưu ở `localStorage['bds.session.v1']` = `{ username, role, name, loggedInAt }`. Flow luôn bắt đầu từ đăng nhập: mở `index.html` sẽ xoá phiên cũ và hiện màn đăng nhập (không tự vào thẳng app).
- `mobile.html` / `hr.html` kiểm tra phiên ngay trong `<head>`: chưa đăng nhập hoặc sai vai trò → về `index.html`.
- `Cài đặt → Đăng xuất` xóa phiên và về màn đăng nhập. Đổi vai trò ở bottom-sheet avatar = đăng nhập sang tài khoản demo của vai trò kia.

## B8. Lộ trình triển khai
| Phase | Nội dung | Kết quả |
| :--- | :--- | :--- |
| **P0 — Khung** | `hr.html`: shell + design system + router + bottom nav 4 tab; `index.html` chọn role + nhớ role; **bottom-sheet đổi vai trò khi bấm avatar (thêm vào cả `mobile.html`)**; `view-checkin` + `view-settings` + `edit-profile` bê nguyên | Bấm được 5 tab, chấm công chạy, đổi qua lại Sale ⇄ HR bằng avatar |
| **P1 — Lõi HR** | `HR_ORG_CONFIGS` + `HR_STAFF_DATABASE`; Tab Nhân sự drilldown 4 cấp + chip lọc + quick sheet; `view-staff-detail` | Xem được toàn bộ cây tổ chức |
| **P2 — Duyệt phép** | `HR_REQUESTS` + localStorage; Tab Duyệt phép; `view-request-detail`; badge đếm trên nav; nối đúng đơn Sale gửi | **Demo mạnh nhất** — duyệt/từ chối có lưu |
| **P3 — Bảng công** | Màn Bảng công (mục trong tab Nhân sự): 4 ô tổng quan, ma trận công tháng cuộn ngang, danh sách cảnh báo, `view-timesheet-detail` | Giám sát chấm công đầy đủ |
| ~~P4 — Tuyển dụng~~ | **Đã bỏ** — HR chỉ quản lý công của Sale | — |
| **P5 — Bổ sung** | `view-payroll` (lương + hoa hồng), `view-training` + QR điểm danh, `view-announce`, `view-notifications` HR | Đủ bộ để nghiệm thu |

**Ước lượng:** P0 ~0,75 ngày · P1 ~1,5 · P2 ~1 · P3 ~1 · P5 ~1 → **~5,25 ngày công**.

## B9. Quyết định đã chốt (2026-09-14)
| # | Vấn đề | Chốt |
| :--- | :--- | :--- |
| 1 | Cấu trúc file | **File `hr.html` riêng**; `index.html` đổi thành màn chọn vai trò (Sale / HR). Không đụng vào `mobile.html`. |
| 2 | Bộ tab | `[Nhân sự] [Duyệt phép] (◉ Chấm công) [Nghiệp vụ] [Cài đặt]` — bỏ Tuyển dụng; Bảng công là mục nhỏ trong Nhân sự, tab Nghiệp vụ thay chỗ |
| 6 | Tuyển dụng | **Bỏ hẳn** — HR chỉ quản lý chủ yếu công của Sale |
| 3 | Phạm vi dữ liệu | **HR xem được doanh số & hoa hồng.** Giữ màn `view-payroll` (lương cứng + công + hoa hồng → thực nhận). |
| 4 | Role Team Leader | **Để sau**, không nằm trong đợt này. |
| 5 | Đổi vai trò trong app | **Có** — bấm avatar mở bottom-sheet "Tài khoản & Vai trò", chọn Sale / HR để nhảy file. Chi tiết ở B7.1, làm trong P0. |

## B10. Trạng thái triển khai (cập nhật 2026-09-14)

**Đã hiện thực xong P0 → P3 và P5** (P4 Tuyển dụng đã bỏ). File bàn giao:

| File | Nội dung |
| :--- | :--- |
| `hr.html` | Mockup HR đầy đủ — 17 màn, ~3.000 dòng, single-file như `mobile.html` |
| `index.html` | **Màn đăng nhập**: form tên đăng nhập / mật khẩu + 2 tài khoản demo (bấm là đăng nhập ngay) |
| `mobile.html` | Bổ sung bottom-sheet đổi vai trò (bấm avatar) + mục trong Cài đặt |

Đã kiểm chứng: cú pháp JS của cả 3 file, lồng thẻ HTML (0 lỗi), và 24 kịch bản runtime trên `hr.html` (drilldown toàn bộ cây tổ chức, mở hồ sơ 85 nhân sự, duyệt/từ chối đơn có lưu, bảng lương, chấm công) — tất cả chạy sạch.

**Chưa làm (ngoài phạm vi đã chốt):** role Team Leader.
