# Khám Phá: Hội Chứng "Mất Trí Nhớ" State (SPA State Amnesia)

## The Insight
Trong các ứng dụng Single Page App (đặc biệt là Next.js/React), khi điều hướng sâu (từ màn hình Danh sách chứa Search/Filter sang xem Chi tiết một mục) rồi ấn nút Back, ứng dụng thường xuyên bị "quên" mất trạng thái đã lọc. Bản chất của vấn đề là do developer sử dụng Local State (`useState`) thay vì **URL-based State**. Khi component unmount để chuyển trang, rồi mount lại khi thao tác Browser Back, các `useState` sẽ bị reset về giá trị default ban đầu (ví dụ: Tab tự trả về mặc định, ô Search/Pagination bị xóa sạch).

## Why This Matters
Đây là một "lỗi thường thức" (UX Logic Bug) nghiêm trọng. Người dùng sẽ cảm thấy cực kỳ ức chế vì luồng làm việc bị ngắt quãng. Hãy tưởng tượng họ đang duyệt danh sách hồ sơ ở Trang 5, bấm vào đọc chi tiết 1 hồ sơ, rồi ấn Back ra ngoài và hệ thống ném họ bị động về lại Trang 1.

## Recognition Pattern
Để tìm ra bug này trên bất kỳ dự án React/Next.js/Vue nào trong tương lai, hãy luôn thực hiện **"The Amnesia Test"**:
1. Thay đổi một trạng thái ở trang list (VD: gõ Search "ME35", chuyển qua Tab "ALL", bấm sang Page "2").
2. Click một item để điều hướng sâu (chuyển sang trang Route Detail).
3. Nhấn **Browser Back** (`window.history.back()` hoặc nút `<=` trên trình duyệt).
4. **Dấu hiệu mắc bệnh:** Mọi tùy chỉnh hiển thị đã làm ở bước 1 biến mất, UI trở về giao diện mặc định.

## The Approach
Khi phát hiện ra lỗi này (hoặc khi thiết kế tính năng mới), hệ tư duy của Antigravity phải như sau:
- Bất kỳ state nào quyết định dữ liệu đang hiển thị (Filters, Search Query, Pagination, Active Tabs) MẶC ĐỊNH phải được "neo" (sync) lên thanh địa chỉ (URL Parameters).
- Phải chuyển dịch từ Local State ẩn sang URL-Driven State. 
- URL phải là "Single Source of Truth". Trình duyệt quay về đúng URL `/dashboard?tab=all&search=me35&page=2` thì danh sách cũng tự động render y như vậy.

## Example
**❌ Anti-pattern phổ biến (Gây lỗi Amnesia):**
```tsx
// Lỗi! State bay màu khi component bị unmount
const [activeTab, setActiveTab] = useState('PENDING'); 
const [searchStr, setSearchStr] = useState('');
```

**✅ Tự phục hồi với URL-State:**
```tsx
import { useSearchParams, useRouter, usePathname } from 'next/navigation';

export default function Dashboard() {
  const searchParams = useSearchParams();
  const router = useRouter();
  const pathname = usePathname();

  // Đọc state từ URL (Single source of truth)
  const activeTab = searchParams.get('tab') || 'PENDING';
  const searchStr = searchParams.get('search') || '';

  // Hàm update state -> thay đổi trực tiếp lên URL
  const updateTab = (newTab: string) => {
    const params = new URLSearchParams(searchParams);
    params.set('tab', newTab);
    router.replace(`${pathname}?${params.toString()}`);
  }
}
```
