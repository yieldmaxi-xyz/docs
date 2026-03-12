# Application State Machines & Diagrams

Tài liệu này mô tả vòng đời (lifecycle) và các trạng thái (states) của các luồng dữ liệu, tiến trình nền (cron jobs), và kiến trúc caching trong hệ thống **Crypto Yield Maxi**.

---

## 1. Sync Flow (Luồng đồng bộ)

Luồng này chịu trách nhiệm thu thập, tính toán và lưu trữ các cơ hội giao dịch (Delta Neutral, Arbitrage) từ hơn 20 sàn giao dịch. Nó được kích hoạt liên tục thông qua Cron Job.

```mermaid
stateDiagram-v2
    [*] --> T1_Triggered : Cron trigger (Mỗi 1 giờ)
    
    state "Tiến trình đồng bộ Real-Time" as SyncProcess {
        T1_Triggered --> Fetching_Delta : Gọi syncDeltaNeutral()
        
        state "Delta-Neutral Sync" as DeltaSync {
            Fetching_Delta --> Fetch_Exchanges : Query 20+ APIs song song
            Fetch_Exchanges --> Calculating_APR : Xử lý dữ liệu (Fees, Spread, Rate)
            Calculating_APR --> Filtering_Liquidity : Lọc Volume & OI (Spot+Perp)
            Filtering_Liquidity --> Save_Delta_DB : Lưu bảng funding_extractor
            Save_Delta_DB --> Snapshot_History : Lấy snapshot bảng history
        }
        
        Snapshot_History --> Fetching_Arbitrage : Gọi syncArbitrage()
        
        state "Arbitrage Sync" as ArbSync {
            Fetching_Arbitrage --> Load_History : Kéo bulk DB History (N+1 fixed)
            Load_History --> Find_Pairs : Tìm cặp sàn Long/Short tốt nhất
            Find_Pairs --> Filter_Arbitrage : Tính Spread, APR & Risk
            Filter_Arbitrage --> Save_Arb_DB : Lưu bảng arbitrage_opportunities
        }
        
    }
    
    Save_Arb_DB --> Idle : Hoàn tất chu kỳ
    Idle --> [*]
```

**Chi tiết luồng:**
1. Khởi động từ `cron/index.ts`.
2. **Delta Neutral** xử lý cập nhật dữ liệu Funding chung.
3. **Snapshot History** chạy ngay trong `syncDeltaNeutral()` để đảm bảo DB luôn lưu lại phiên cập nhật mới nhất.
4. **Arbitrage** tính toán mức chênh lệch giữa các cặp sàn.

---

## 2. External Data Sync Flow (Luồng đồng bộ từ bên thứ 3)

Luồng này phục vụ cho các mảng DeFi tĩnh hoặc ít thay đổi (Lending, Vaults, Listings sàn), chạy với chu kỳ dài hơn (Hàng giờ hoặc Hàng ngày).

```mermaid
stateDiagram-v2
    [*] --> Scheduled_Trigger : Cron trigger (1H, 24H)
    
    Scheduled_Trigger --> API_Request : Gọi API bên thứ 3 (VD: DefiLlama)
    
    state API_Request {
        [*] --> Rate_Limiter_Wait : Đợi tránh limit
        Rate_Limiter_Wait --> FetchData
        
        FetchData --> Retry_Loop : Lỗi Timeout/5xx
        Retry_Loop --> FetchData : Exponential Backoff
        
        FetchData --> Parsing_Data : Thành công HTTP 200
    }
    
    Parsing_Data --> Format_Transform : Ánh xạ sang schema nội bộ
    Format_Transform --> Database_Upsert : ON DUPLICATE KEY UPDATE (xóa cũ/thêm mới)
    
    Database_Upsert --> Clear_Cache : Xóa Memory/Redis liên quan
    Clear_Cache --> [*] : Hoàn tất cập nhật
```

**Các chu kỳ:**
* **Delta Neutral Sync:** Kích hoạt tại phút thứ 1 mỗi giờ.
* **Arbitrage Sync:** Kích hoạt tại phút thứ 3 mỗi giờ.
* **Lending Sync:** Kích hoạt tại phút thứ 7 mỗi giờ.
* **Vault Sync:** Kích hoạt tại phút thứ 9 mỗi giờ.
* **Exchange Metadata Sync:** Kích hoạt hàng ngày lúc 1:00 AM UTC.

---

## 3. Request Caching State Machine (Luồng phản hồi API cho UI)

Hệ thống có 3 tầng cache (Frontend ISR, Backend Memory, Backend Redis) để tối ưu độ phản hồi dưới 100ms.

```mermaid
sequenceDiagram
    participant U as User (Trình duyệt)
    participant F as Frontend (Next.js)
    participant N as Next.js ISR Cache
    participant B as Backend (Express)
    participant M as L1 Cache (Memory)
    participant R as L2 Cache (Redis)
    participant D as Database (MySQL)

    U->>F: Truy cập trang (VD: /delta-neutral)
    F->>N: Kiểm tra dữ liệu (next: {revalidate: 60})
    
    alt Có trong Next.js Cache & Còn hạn
        N-->>F: Trả về dữ liệu (HIT)
        F-->>U: Hiển thị ngay (0ms API delay)
    else Hết hạn (Stale) hoặc Cache Miss
        N->>B: Gửi HTTP GET /api/funding-extractor
        
        B->>M: Kiểm tra Memory Cache
        alt Có trong L1 Memory
            M-->>B: L1 Cache HIT (~0ms)
        else Không có L1
            B->>R: Kiểm tra Redis
            alt Có trong L2 Redis
                R-->>B: L2 Cache HIT (~5ms)
                B->>M: Lưu lại vào L1 Memory
            else Không có L2
                B->>D: Truy vấn MySQL
                D-->>B: Trả về kết quả
                B->>R: Lưu L2 Redis (TTL 60s)
                B->>M: Lưu L1 Memory (TTL 60s)
            end
        end
        
        B-->>N: Trả về JSON Response
        N->>N: Lưu lại vào ISR Cache (Dùng cho 60s tới)
        N-->>F: Lên UI
        F-->>U: Nội dung cập nhật
    end
```

**Chiến lược tối ưu:**
1. **Frontend không trực tiếp gọi API Backend per-request** nhờ cấu hình `next: { revalidate }`. Khách hàng thứ 1000 truy cập chung 1 phút sẽ nhận cùng 1 static HTML/JSON từ node server của Vercel/Next.js.
2. **Backend MemCache (L1) ngăn chập tải Redis:** Nếu Frontend cache bị miss, API request đập vào BE, BE lập tức trả ra từ biến Map() Memory cực nhanh.
3. **Redis (L2) chia sẻ liên server:** Nếu Backend có nhiều node hoặc Worker vừa chạy xong, kết quả tính toán có sẵn ở Redis chia cho mọi processes.

---

## 4. Single Opportunity Lifecycle (Vòng đời của cơ hội Arbitrage/Funding)

Máy trạng thái vòng đời của một kết quả cụ thể trong DB (ví dụ: Chênh lệch giá PURR).

```mermaid
stateDiagram-v2
    [*] --> Discovered : Cron quét thấy cơ hội mới
    
    Discovered --> Active : Vượt mốc thanh khoản & lợi nhuận
    
    Active --> Fluctuation : APR/Spread thay đổi (Cập nhật liên tục 15p/lần)
    Fluctuation --> Active : Cập nhật % Spread/Fee
    
    Active --> Unprofitable : Chênh lệch giá (Spread) quá lớn hoặc APR giảm
    Unprofitable --> Drop : Bot gỡ khỏi bảng DB sau chu kỳ sync tiếp theo
    
    Active --> Persisted_History : Chuyển sang bảng _history khi chốt giờ (8H)
    
    Persisted_History --> Aggregated : Tổng hợp thành bản ghi trung bình (24h/7D/30D)
    Aggregated --> [*]
    Drop --> [*]
```

1. **Sinh trưởng:** Một cơ hội xuất hiện mỗi vòng Sync (Khoảng 15p).
2. **Tuổi thọ thực (Active):** Thay đổi liên tục, "nhấp nháy" phần trăm lợi nhuận. Nếu đáp ứng tiêu chuẩn, được phơi lên UI.
3. **Dập tắt (Drop):** Ở lần sweep tiếp theo, MySQL sẽ tiến hành `ON DUPLICATE KEY UPDATE` hoặc xóa/tạo lại nếu tỷ giá biến mất, cơ hội hết vòng đời.
4. **Di sản (History):** Tại mốc chẵn giờ, nó được đưa vào `_history` để sau này phục vụ các tính toán chu kỳ dài.
