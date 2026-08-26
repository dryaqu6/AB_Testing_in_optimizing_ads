# A/B Testing in Optimizing Ads

> **Đồ án môn Python cho Khoa học dữ liệu**, lớp 22TTH, Khoa Toán – Tin học, Trường ĐH Khoa học Tự nhiên, ĐHQG-HCM.
> Nhóm thực hiện: **Lâm Gia Bảo** (22110023) · **Trần Duy An** (22110008) · **Đậu Quang Anh** (22110014) · **Trần Quốc Danh** (22110035).
> Giảng viên bộ môn: **ThS. Hà Văn Thảo**.

Phân tích một thử nghiệm A/B của nhà bán lẻ giày trực tuyến: hai phiên bản quảng cáo (**nhóm A** và **nhóm B**) được chạy trên bốn nền tảng (**Google, Facebook, Twitter, Email**) suốt bảy ngày trong tuần. Mục tiêu là xác định phiên bản nào và nền tảng nào cho tỷ lệ nhấp chuột cao hơn, và khẳng định điều đó **có ý nghĩa thống kê** hay chỉ là dao động ngẫu nhiên.

## 📂 Dữ liệu

`data/ad_clicks.csv` — 1.654 người dùng, không có dòng trùng lặp.

| Cột | Ý nghĩa |
| :--- | :--- |
| `user_id` | định danh người dùng (duy nhất) |
| `utm_source` | nền tảng hiển thị quảng cáo (4 giá trị) |
| `day` | ngày trong tuần (7 giá trị) |
| `ad_click_timestamp` | thời điểm nhấp chuột — **khuyết nếu không nhấp** |
| `experimental_group` | nhóm quảng cáo A hoặc B |

Biến mục tiêu `is_click` được suy ra từ việc `ad_click_timestamp` có khuyết hay không.

## 🧪 Các kiểm định đã thực hiện

| # | Câu hỏi | Phương pháp | Kết quả |
| :--- | :--- | :--- | :--- |
| 1 | Tỷ lệ chuyển đổi có khác nhau giữa 4 nền tảng? | **Chi-square test of independence** (`scipy.stats.chi2_contingency`) | p = **0,7963** → chưa đủ cơ sở bác bỏ H₀ |
| 2 | Tỷ lệ truy cập có khác nhau giữa quảng cáo A và B? | **Chi-square test of independence** | p = **0,0051** → **bác bỏ H₀**, có khác biệt |
| 3 | Facebook có thật sự hơn từng nền tảng còn lại? | **Two-proportion z-test** từng cặp (`statsmodels.stats.proportions_ztest`) | Facebook cao hơn có ý nghĩa |
| 4 | Số lượt nhấp có thay đổi theo ngày trong tuần? | **One-way ANOVA** (`scipy.stats.f_oneway`) | p = **0,6053** → không có khác biệt giữa các ngày |

Mức ý nghĩa dùng thống nhất α = 0,05.

## 📊 Kết luận

- **Quảng cáo nhóm A** thu hút nhiều lượt nhấp hơn nhóm B, và khác biệt này **có ý nghĩa thống kê** (kiểm định 2).
- **Facebook** cho tỷ lệ nhấp chuột cao nhất trong bốn nền tảng, dù **Google** mới là nền tảng mang lại nhiều lượt *hiển thị* nhất — số lượt xem nhiều không đồng nghĩa với tỷ lệ chuyển đổi cao.
- **Không** có bằng chứng cho thấy hiệu quả quảng cáo thay đổi theo ngày trong tuần (kiểm định 4).

## ⚠️ Hạn chế đã biết

- Kiểm định 1 và 2 dùng chi-square trên bảng chéo; chưa báo cáo **effect size** hay **khoảng tin cậy** cho chênh lệch tỷ lệ, nên chỉ kết luận được *"có khác biệt"* chứ chưa định lượng *"khác biệt bao nhiêu"*.
- Các so sánh z-test từng cặp ở kiểm định 3 **chưa hiệu chỉnh đa so sánh** (Bonferroni / Holm), nên xác suất sai lầm loại I gộp cao hơn 0,05.
- Dữ liệu chỉ bao gồm hiển thị và nhấp chuột, **không có dữ liệu mua hàng** — "conversion" ở đây là nhấp chuột, không phải doanh thu.
- Cỡ mẫu 1.654 người trong một tuần; chưa kiểm tra tính ổn định qua nhiều tuần.

## 📁 Cấu trúc

```
data/    ad_clicks.csv                       dữ liệu thô
report/  AB_Testing_in_optimizing_ads.ipynb  toàn bộ phân tích + biểu đồ + kết luận
```

## 📚 Tham khảo

- Bài giảng môn *Python cho Khoa học dữ liệu* — ThS. Hà Văn Thảo.
- Bài giảng môn *Xử lý số liệu thống kê* — TS. Tô Đức Khánh.
- Project tham khảo: [pedramsafaeifar/Ad_Clicks___AB_Testing](https://github.com/pedramsafaeifar/Ad_Clicks___AB_Testing).
