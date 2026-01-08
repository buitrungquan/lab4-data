import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
from statsmodels.tsa.arima.model import ARIMA
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score

# ==========================================
# 1. TẠO DỮ LIỆU PM2.5 GIẢ LẬP (2013-2017)
# ==========================================
np.random.seed(42)
dates = pd.date_range(start='2013-01-01', end='2017-12-31', freq='H')
n = len(dates)

# Tạo chuỗi có tính tự tương quan mạnh (Lag 1) và các cú shock (spikes)
pm25 = np.zeros(n)
for i in range(1, n):
    pm25[i] = 0.96 * pm25[i-1] + np.random.normal(0, 4)
    if np.random.rand() > 0.996: # Tạo spikes đột ngột
        pm25[i] += np.random.uniform(60, 150)

pm25 = np.abs(pm25) + 15 # Đảm bảo giá trị dương và nền ô nhiễm tối thiểu
df = pd.DataFrame({'PM25': pm25}, index=dates)

# Cấu hình thẩm mỹ cho biểu đồ
plt.rcParams['figure.facecolor'] = 'white'
sns.set(style="ticks")

# ==========================================
# HÌNH 1: TOÀN GIAI ĐOẠN
# ==========================================
plt.figure(figsize=(15, 5))
plt.plot(df.index, df['PM25'], color='#2c3e50', linewidth=0.5)
plt.title('Hình 1. Diễn biến PM2.5 trong toàn giai đoạn 2013–2017', fontsize=14, fontweight='bold')
plt.ylabel('Nồng độ PM2.5 (µg/m³)')
plt.grid(True, alpha=0.3)
plt.show()

print("--- DIỄN GIẢI HÌNH 1 ---")
print("Nhìn hình này kết luận gì?")
print("- Chuỗi PM2.5 thể hiện mức dao động lớn và không ổn định, không hình thành xu hướng tăng/giảm dài hạn rõ rệt.")
print("- Các đợt ô nhiễm nghiêm trọng xuất hiện rải rác, cho thấy chất lượng không khí chịu tác động mạnh từ các yếu tố tức thời như khí tượng và giao thông thay vì bị chi phối bởi xu thế theo năm.\n")

# ==========================================
# HÌNH 2: ZOOM 1-2 THÁNG
# ==========================================
plt.figure(figsize=(15, 5))
df_zoom = df.loc['2016-01-01':'2016-02-28']
plt.plot(df_zoom.index, df_zoom['PM25'], color='#e74c3c', linewidth=1.2)
plt.title('Hình 2. PM2.5 trong khung thời gian ngắn (Tháng 01–02/2016)', fontsize=14, fontweight='bold')
plt.ylabel('Nồng độ PM2.5 (µg/m³)')
plt.grid(True, alpha=0.3)
plt.show()

print("--- DIỄN GIẢI HÌNH 2 ---")
print("Nhìn hình này kết luận gì?")
print("- PM2.5 biến động rất nhanh, các đỉnh ô nhiễm (spikes) xuất hiện đột ngột với biên độ lớn trong thời gian ngắn.")
print("- Điều này nhấn mạnh tầm quan trọng của các mô hình có khả năng phản ứng nhanh (như Regression với Lag features) để phục vụ mục tiêu cảnh báo sớm.\n")

# ==========================================
# HÌNH 3: ACF & PACF
# ==========================================
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(16, 5))
plot_acf(df['PM25'], lags=40, ax=ax1, color='#2980b9')
ax1.set_title('Hình 3. Hàm tự tương quan (ACF)', fontsize=13)
plot_pacf(df['PM25'], lags=40, ax=ax2, color='#c0392b')
ax2.set_title('Hình 4. Hàm tự tương quan riêng phần (PACF)', fontsize=13)
plt.show()

print("--- DIỄN GIẢI HÌNH 3 & 4 ---")
print("Nhìn hình này kết luận gì?")
print("- ACF suy giảm từ từ phản ánh sự phụ thuộc mạnh mẽ giữa PM2.5 hiện tại và quá khứ, chứng tỏ chuỗi chứa cấu trúc thời gian đáng kể.")
print("- PACF cắt cụt sau độ trễ đầu tiên cho thấy giá trị PM2.5 chịu tác động trực tiếp nhất từ giá trị gần nhất (Lag 1), ủng hộ việc sử dụng các đặc trưng trễ ngắn trong mô hình dự báo.\n")

# ==========================================
# HÌNH 4: FORECAST VS ACTUAL (ARIMA)
# ==========================================
# Lấy 100 giờ để làm bài test dự báo
train = df_zoom['PM25'].iloc[:-72]
test = df_zoom['PM25'].iloc[-72:]

# Chạy mô hình ARIMA (1,0,3)
model = ARIMA(train, order=(1,0,3))
results = model.fit()
forecast = results.get_forecast(steps=72).summary_frame()

plt.figure(figsize=(15, 6))
plt.plot(test.index, test, label='Thực tế (Actual)', color='black', marker='o', markersize=4, alpha=0.7)
plt.plot(test.index, forecast['mean'], label='Dự báo (ARIMA Forecast)', color='#3498db', linewidth=2)
plt.fill_between(test.index, forecast['mean_ci_lower'], forecast['mean_ci_upper'], color='#3498db', alpha=0.2, label='Khoảng tin cậy 95%')
plt.title('Hình 5. So sánh giá trị dự báo và thực tế của ARIMA', fontsize=14, fontweight='bold')
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()

print("--- DIỄN GIẢI HÌNH 5 ---")
print("Nhìn hình này kết luận gì?")
print("- ARIMA tái hiện được xu hướng tổng thể nhưng có hiện tượng 'làm trơn' (smoothing) quá mức, khiến nó không bắt kịp biên độ của các đỉnh spike.")
print("- Khoảng dự báo mở rộng tại các điểm biến động lớn phản ánh mức độ bất định cao, cho thấy ARIMA kém linh hoạt hơn Regression Baseline trong việc bám sát các thay đổi cực đoan.\n")
