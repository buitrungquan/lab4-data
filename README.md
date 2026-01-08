Chủ đề 1: Regression vs ARIMA – khi nào chọn cái nào?
<img width="553" height="182" alt="image" src="https://github.com/user-attachments/assets/27db8cd5-7752-42fd-907f-539b9b4d29a4" />


						Hình 1. PM2.5 toàn giai đoạn (2013–2017)
Diễn giải:
Chuỗi PM2.5 biến động rất mạnh trong toàn bộ giai đoạn nghiên cứu, với nhiều đỉnh cao xuất hiện rải rác theo thời gian. Không quan sát thấy xu hướng tăng hoặc giảm dài hạn rõ ràng, cho thấy ô nhiễm không khí không cải thiện ổn định theo năm. Điều này gợi ý rằng PM2.5 chịu ảnh hưởng chủ yếu bởi các yếu tố ngắn hạn như thời tiết và hoạt động con người hơn là xu hướng dài hạn.
<img width="554" height="182" alt="image" src="https://github.com/user-attachments/assets/8542ed18-eaa3-44fb-b355-1867f3d23c1f" />

						Hình 2. PM2.5 zoom 1–2 tháng
Diễn giải:
Khi quan sát trong khoảng thời gian ngắn, PM2.5 có thể tăng rất nhanh trong vài giờ hoặc vài ngày, tạo ra các đợt ô nhiễm đột ngột. Các đỉnh PM2.5 có biên độ lớn và xuất hiện không đều, gây khó khăn cho dự báo nếu mô hình phản ứng chậm. Điều này cho thấy dự báo ngắn hạn có vai trò quan trọng trong hệ thống cảnh báo sớm chất lượng không khí.

<img width="554" height="163" alt="image" src="https://github.com/user-attachments/assets/080935e4-0823-49fc-8ee9-271c8570dfa9" />


			          Hình 3. ACF của PM2.5
Diễn giải:
Hệ số tự tương quan giảm chậm theo độ trễ, cho thấy PM2.5 có mối liên hệ mạnh với các giá trị trong quá khứ gần. Điều này chứng tỏ chuỗi không phải là nhiễu ngẫu nhiên mà có cấu trúc phụ thuộc theo thời gian rõ ràng. Kết quả này giải thích vì sao các đặc trưng độ trễ có giá trị cao trong các mô hình dự báo.

<img width="554" height="163" alt="image" src="https://github.com/user-attachments/assets/49401dbd-2e71-47bd-84d0-0c2a5c910088" />


						Hình 4. PACF của PM2.5
Diễn giải:

PACF thể hiện các đỉnh đáng kể ở những độ trễ nhỏ, đặc biệt ở độ trễ đầu tiên. Điều này cho thấy PM2.5 hiện tại chịu ảnh hưởng trực tiếp mạnh từ các giá trị gần nhất trong quá khứ. Quan sát này phù hợp với việc lựa chọn các độ trễ nhỏ trong mô hình ARIMA và regression.
ARIMA (Forecast vs Actual)

<img width="553" height="220" alt="image" src="https://github.com/user-attachments/assets/0b060c26-4c28-4f83-b537-a1dba5922b7f" />

		    			Hình 5. Forecast vs Actual của mô hình ARIMA
Diễn giải:
Dự báo của ARIMA bám được xu hướng chung của PM2.5 nhưng có xu hướng làm mượt chuỗi. Trong các giai đoạn PM2.5 tăng đột ngột, mô hình phản ứng chậm và thường đánh giá thấp biên độ đỉnh. Khoảng dự báo rộng hơn trong các thời điểm biến động mạnh phản ánh mức độ bất định cao của chuỗi PM2.5.

Q1. Mô hình nào tốt hơn cho horizon = 1?
- Regression baseline tốt hơn ARIMA cho horizon = 1.
Dẫn chứng bằng số liệu :

Regression baseline: RMSE ≈ 25.33, MAE ≈ 12.32, R² ≈ 0.95
ARIMA (1,0,3): sai số cao hơn regression (thể hiện trên Forecast vs Actual)

Giải thích :

Dự báo rất ngắn hạn của PM2.5 bị chi phối mạnh bởi độ trễ gần nhất, đặc biệt là PM2.5_lag1. Regression baseline khai thác trực tiếp các đặc trưng lag (1, 3, 24) và time features nên bám sát biến động ngắn hạn tốt nếu feature engineering đúng. ARIMA có thể hoạt động tốt trong một số trường hợp nhưng phụ thuộc mạnh vào cấu trúc tự tương quan và quyết định sai phân, nên kém linh hoạt hơn cho horizon rất ngắn.

Q2. Mô hình nào ổn hơn khi có spike?

-  Regression baseline ổn hơn ARIMA khi xuất hiện spike PM2.5.
Phân tích:

Trên đoạn 1–3 ngày có đỉnh PM2.5 rõ, regression phản ứng nhanh hơn do sử dụng thông tin trễ gần nhất, trong khi ARIMA có xu hướng làm mượt và phản ứng chậm, dẫn đến việc đánh giá thấp biên độ đỉnh. Khi mô hình sai nặng ở một vài thời điểm spike, RMSE tăng mạnh hơn MAE, điều này giải thích vì sao ARIMA thường bị phạt nặng về RMSE trong các giai đoạn có spike.

Q3. Nếu triển khai thật, bạn chọn gì và vì sao?

- Chọn regression baseline cho hệ thống cảnh báo sớm.
Lý do:

Regression baseline dễ mở rộng khi bổ sung thêm đặc trưng (thời tiết, giao thông), dễ cập nhật và chạy nhanh trong môi trường vận hành thực tế. ARIMA có ưu thế về khả năng giải thích theo (p, d, q) và cung cấp khoảng tin cậy dự báo, nên phù hợp để phân tích xu hướng tổng thể, nhưng không phải lựa chọn tối ưu cho cảnh báo sớm trong điều kiện thời tiết và ô nhiễm biến động mạnh.
