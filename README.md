# A/B Testing in Optimizing Ads

> **Đồ án môn học "Python cho Khoa học dữ liệu", 2024 — 4 thành viên:** Lâm Gia Bảo (22110023), Trần Duy An (22110008), Đậu Quang Anh (22110014), Trần Quốc Danh (22110035).
> **Về lịch sử commit:** toàn bộ commit trong repo này đứng tên tôi (Trần Duy An) vì tôi là người archive dự án lên GitHub một năm sau khi học xong. **Lịch sử commit không phản ánh phân công công việc.**
> ⚠️ **Cảnh báo về nội dung:** đọc lại năm 2026, phần suy luận thống kê trong notebook có lỗi — xem mục [Hạn chế đã biết](#-hạn-chế-đã-biết) ở cuối. Đừng dùng kết luận trong notebook làm tham khảo.

Phân tích một thử nghiệm A/B của nhà bán lẻ giày trực tuyến: hai phiên bản quảng cáo (**nhóm A** và **nhóm B**) được chạy trên bốn nền tảng (**Google, Facebook, Twitter, Email**) suốt bảy ngày trong tuần. Mục tiêu là xác định phiên bản nào và nền tảng nào cho tỷ lệ nhấp chuột cao hơn, và khẳng định điều đó **có ý nghĩa thống kê** hay chỉ là dao động ngẫu nhiên.

## 📂 Dữ liệu

`data/ad_clicks.csv` — **1.654 người dùng**, không có dòng trùng lặp. Trong đó **565 người có nhấp chuột** (`ad_click_timestamp` khác rỗng) → tỷ lệ nhấp chung **34,2%**.

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
| 1 | Tỷ lệ chuyển đổi có khác nhau giữa 4 nền tảng? | **Chi-square test of independence** (`scipy.stats.chi2_contingency`) | p = **0,4133** → chưa đủ cơ sở bác bỏ H₀. ⚠️ Notebook in ra **0,7963** — con số đó **SAI** do lỗi code ở cell 24 (`Crosstable_with_percentage = Xtab` là gán tham chiếu, làm cột `click_percentage` lọt vào bảng chi-square, dof 6 thay vì 3). Kết luận không đổi |
| 2 | Tỷ lệ truy cập có khác nhau giữa quảng cáo A và B? | **Chi-square test of independence** | p = **0,0051** → **bác bỏ H₀**, có khác biệt |
| 3 | Facebook có thật sự hơn từng nền tảng còn lại? | **Two-proportion z-test** từng cặp (`statsmodels.stats.proportions_ztest`) | ⚠️ **KHÔNG có ý nghĩa thống kê** — cả ba p-value đều > 0,05 (0,2338 · 0,8401 · 0,1942). Kết luận "Facebook cao hơn" trong notebook là **SAI** |
| 4 | Số lượt nhấp có thay đổi theo ngày trong tuần? | **One-way ANOVA** (`scipy.stats.f_oneway`) | p = **0,6053** → không có khác biệt giữa các ngày. *(Ghi chú: `f_oneway` trên biến nhị phân là không chuẩn; chi-square 7×2 cho p = 0,6043 — lệch 0,001, không đổi kết luận.)* |
| 5 | A có hơn B theo **từng ngày** không? | ⚠️ **KHÔNG CÓ KIỂM ĐỊNH NÀO ĐƯỢC CHẠY** | Kết luận cuối notebook khẳng định *"có ý nghĩa thống kê... vào hầu hết các ngày trong tuần (trừ Thứ Ba)"* nhưng cell 89–96 **chỉ vẽ biểu đồ**. Chạy lại đủ 7 z-test: **chỉ 1/7 ngày** có p < 0,05 (Thursday p = 0,0118), và **0/7** sau hiệu chỉnh Bonferroni. Claim này **SAI** |

Mức ý nghĩa dùng thống nhất α = 0,05.

## 📊 Kết luận

- **Quảng cáo nhóm A** thu hút nhiều lượt nhấp hơn nhóm B, và khác biệt này **có ý nghĩa thống kê** (kiểm định 2): A 37,48% vs B 30,83% — chênh **+6,65 điểm phần trăm**, 95% CI **[+2,09; +11,21]**. Đây là **kết luận duy nhất trong notebook được chống đỡ vững**. ⚠️ Nó đúng ở mức **tổng thể**, **không** bảo chứng cho claim "hơn vào hầu hết các ngày trong tuần" ở cuối notebook (xem kiểm định 5).
- **Facebook** có tỷ lệ nhấp chuột cao nhất *về mặt số học*, nhưng ⚠️ **chênh lệch này KHÔNG đạt ý nghĩa thống kê** (cả ba z-test từng cặp đều p > 0,05; chi-square trên bốn nền tảng cũng cho p = 0,4133). Không kết luận được nền tảng nào hơn. **Google** mang lại nhiều lượt *hiển thị* nhất, nhưng số lượt xem không đồng nghĩa với tỷ lệ chuyển đổi.
- **Không** có bằng chứng cho thấy hiệu quả quảng cáo thay đổi theo ngày trong tuần (kiểm định 4).

## ⚠️ Hạn chế đã biết

- **Notebook** chỉ báo cáo p-value, không kèm **effect size** hay **khoảng tin cậy**, nên chỉ nói được *"có khác biệt"* chứ không định lượng *"khác biệt bao nhiêu"*. *(Mục Kết luận ở trên đã bổ sung CI cho kiểm định 2 khi rà lại năm 2026.)*
- Các so sánh z-test từng cặp ở kiểm định 3 **chưa hiệu chỉnh đa so sánh** (Bonferroni / Holm), nên xác suất sai lầm loại I gộp cao hơn 0,05. ⚠️ Tuy nhiên ở đây **hiệu chỉnh không làm đổi kết luận nào** — p nhỏ nhất trong cả sáu cặp là 0,1942, mà Bonferroni chỉ làm ngưỡng khắt khe hơn. **Đây KHÔNG phải nguyên nhân của kết luận Facebook sai**; nguyên nhân là hiểu "không bác bỏ được H₀" thành "H₀ đã được chứng minh" (*affirming the null*), cộng với việc H₀ ở cell 56 bị viết thành đối thuyết.
- Kiểm định 3 còn dùng **two-sided test cho một tuyên bố một phía** ("Facebook *nhiều hơn*") — thiếu `alternative='larger'`. Chạy lại đúng chiều vẫn không cứu được: p = 0,1169 · 0,4200 · 0,0971.
- Dữ liệu chỉ bao gồm hiển thị và nhấp chuột, **không có dữ liệu mua hàng** — "conversion" ở đây là nhấp chuột, không phải doanh thu.
- Cỡ mẫu 1.654 người trong một tuần; chưa kiểm tra tính ổn định qua nhiều tuần.

## 📁 Cấu trúc

```
data/    ad_clicks.csv                       dữ liệu thô
report/  AB_Testing_in_optimizing_ads.ipynb  toàn bộ phân tích + biểu đồ + kết luận
```

## 📚 Tham khảo

- Project tham khảo: [pedramsafaeifar/Ad_Clicks___AB_Testing](https://github.com/pedramsafaeifar/Ad_Clicks___AB_Testing).
