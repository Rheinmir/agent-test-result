# QA Test Log — dms.giatbh.io.vn
**Date:** 2026-04-08  
**Tester:** [Your name]  
**URL:** https://dms.giatbh.io.vn  

---

## 1. ACCOUNT INFO
| Field | Value |
|-------|-------|
| Username | dms |
| Password | [ghi lại] |
| Role | [ghi lại] |
| Email | [ghi lại] |
| Full Name | [ghi lại] |

---

## 2. CONSOLE ERROR TRACKING
> Mở F12 > Console, paste đoạn JS này để bắt lỗi tự động:

```javascript
// Paste vào Console rồi Enter
window.__qaLog = {errors: [], networkFails: []};
window.addEventListener('error', e => window.__qaLog.errors.push({type:'JS_ERROR', msg: e.message, file: e.filename, line: e.lineno, time: new Date().toISOString()}));
window.addEventListener('unhandledrejection', e => window.__qaLog.errors.push({type:'PROMISE_REJECT', msg: String(e.reason), time: new Date().toISOString()}));

// Override fetch để bắt lỗi network
const origFetch = window.fetch;
window.fetch = async (...args) => {
  const res = await origFetch(...args);
  if (!res.ok) window.__qaLog.networkFails.push({url: args[0], status: res.status, time: new Date().toISOString()});
  return res;
};
console.log('%c✅ QA Error Tracking ACTIVE', 'color:green;font-weight:bold');
```

> Sau khi test xong, lấy log bằng:
```javascript
console.log(JSON.stringify(window.__qaLog, null, 2));
```

---

## 3. PAGES TO TEST

### Dashboard — `/dashboard`
- [ ] Trang hiển thị đúng dữ liệu
- [ ] Filter "pending" hoạt động
- [ ] Filter "all" hoạt động  
- [ ] Các card/widget hiển thị số liệu chính xác
- [ ] Click vào từng card/stat có dẫn đến đúng trang không?
- **Ghi chú issues:**

---

### Documents/Tài liệu
- [ ] Vào trang danh sách tài liệu
- [ ] Search/tìm kiếm tài liệu hoạt động
- [ ] Filter/lọc hoạt động
- [ ] Sort cột (click header bảng) hoạt động
- [ ] Pagination hoạt động
- [ ] Click xem chi tiết tài liệu
- [ ] Nút tạo mới (Create/New/+) hoạt động
- [ ] Form tạo mới — validate đúng không (để trống rồi submit)
- [ ] Upload file hoạt động
- [ ] Download file hoạt động
- [ ] Sửa/Edit tài liệu
- [ ] Xoá/Delete tài liệu (có confirm popup không?)
- **Ghi chú issues:**

---

### Người dùng / User Management (nếu có)
- [ ] Xem danh sách users
- [ ] Tạo user mới
- [ ] Sửa user
- [ ] Phân quyền
- **Ghi chú issues:**

---

### Reports / Báo cáo (nếu có)
- [ ] Xem báo cáo
- [ ] Export báo cáo (PDF/Excel)
- [ ] Filter theo ngày
- **Ghi chú issues:**

---

### Settings / Cài đặt (nếu có)
- [ ] Cài đặt hồ sơ cá nhân
- [ ] Đổi mật khẩu
- [ ] Cài đặt chung
- **Ghi chú issues:**

---

### Notifications / Thông báo (nếu có)
- [ ] Thông báo hiển thị
- [ ] Đánh dấu đã đọc
- **Ghi chú issues:**

---

## 4. UI/UX ISSUES FOUND

| # | Trang | Mô tả vấn đề | Mức độ |
|---|-------|-------------|--------|
| 1 | | | Low/Medium/High |
| 2 | | | |
| 3 | | | |

---

## 5. FUNCTIONAL BUGS

| # | Trang | Bước tái hiện | Kết quả thực tế | Kết quả mong đợi | Mức độ |
|---|-------|--------------|-----------------|-----------------|--------|
| 1 | | | | | Low/Medium/High |
| 2 | | | | | |

---

## 6. CONSOLE ERRORS (paste output từ JS ở trên)

```json
// Paste kết quả JSON.stringify(window.__qaLog) ở đây
```

---

## 7. NETWORK ERRORS (F12 > Network tab, lọc "Errors")

| URL | Status Code | Method |
|-----|-------------|--------|
| | | |

---

## 8. PERFORMANCE NOTES

| Trang | Thời gian load | Ghi chú |
|-------|---------------|---------|
| Dashboard | | |
| | | |

---

## 9. RESPONSIVE / MOBILE

> Test bằng cách nhấn F12 > Toggle device toolbar (Ctrl+Shift+M)

- [ ] Layout OK ở 375px (iPhone SE)
- [ ] Layout OK ở 768px (iPad)
- [ ] Menu mobile hoạt động
- **Ghi chú:**

---

## TỔNG KẾT

- **Tổng số issues:** 
- **High severity:** 
- **Medium severity:**
- **Low severity:**
- **Kết luận:**
