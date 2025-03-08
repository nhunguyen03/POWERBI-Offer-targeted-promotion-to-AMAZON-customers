# [POWER BI] TARGETED PROMOTION: REACH THE AMAZON'S CUSTOMERS EFFECTIVELY
## I. INTRODUCTION
### 1. About project
This project uses **hypothetical data from AMAZON Global Superstore Sales** from 2011 to 2014, covering transactions on the AMAZON e-commerce platform, focusing on analyzing AMAZON's customers and answering the question: **"How to offer targeted promotion to right customers?"**

![image](https://github.com/user-attachments/assets/96db8b72-9a1e-4efb-9335-e28cf37d9d15)

This project emphasizes data processing and visualization **using PBI**

**The analysis results provide insights into:**
- Who the segment primarily are?
- Where they live (country, city)?
- Most preferred product categories?
- The average spending per order (_AOV_) over time? Which strategy in months have the lowest spending?
- Most preferred ship mode? Prioritize cost savings or fast delivery?

[Link raw dataset here](https://drive.google.com/drive/folders/1GdTp3wWn3f_ZvvlmgOuhtGwHEIHDzRRd?usp=drive_link)

### 2. Explain dataset

Raw dataset consists of 3 data tables:
  + Table 1: **Orders**: Fact order
  + Table 2: **People**: Information of Sales person
  + Table 3: **Returns**: Information of Returned orders

**A. _Table Orders_: Primary key - Order ID & Order Date & Customer ID**
- **Order ID**: mã ID đơn hàng
  
_**NOTE**: Mã này có thể bị trùng lặp. Mỗi dòng trong dữ liệu đại diện cho một sản phẩm trong đơn hàng, nghĩa là nếu một đơn hàng có nhiều sản phẩm, mã đơn hàng đó sẽ xuất hiện nhiều lần_

- **Order Date**: ngày xuất đơn hàng. MM/DD/YYYY
- **Ship Date**: ngày vận chuyển đơn hàng. MM/DD/YYYY
- **Ship Mode**: phương thức vận chuyển. Có 4 phương thức: _First Class, Same Day, Second Class, Standard Class_
- **Customer ID**: mã khách hàng
- **Customer Name**: tên khách hàng
- **Segment**: loại khách hàng. Có 3 loại: Consumer - KH cá nhân; Corporate - doanh nghiệp; Home Office - freelancer/doanh nhân cá nhân/công ty nhỏ

_**NOTE**: Phân biệt Segment này với RFM segment_

-  **City**: thành phố của địa chỉ giao hàng
-  **State**: bang của địa chỉ giao hàng
-  **Country**: quốc gia của địa chỉ giao hàng
-  **Postal Code**: mã postal
-  **Market**: thị trường bán lẻ. Có 7 thị trường: APAC, EU, US, LATAM, EMEA, Africa, Canada
-  **Region**: khu vực của thị trường
-  **Product ID**: mã sản phẩm
-  **Category**: loại hàng hóa. Có 3 loại: Office Supplies, Technology, Furrniture
-  **Sub-category**: các nhánh hàng hóa
-  **Product Name**: tên sản phẩm
-  **Sales**: doanh số
-  **Quantity**: số lượng sản phẩm
-  **Profit**: lợi nhuận sản phẩm

**B. _Table People_: thông tin sales person**
- **Person**: tên người bán hàng
- **Region**: khu vực bán hàng

**C. _Table Returns_: thông tin đơn hàng trả lại**
- **Order ID**: mã đơn hàng
- **Returned** - Yes: đơn hàng được trả lại

## II. PROCESS DATA

[Processing table Orders](https://console.cloud.google.com/bigquery?sq=279214207888:b080545939214a63a63bcf7415d59c5f)

⇒ **Result**: [clean_sales_orders2](https://console.cloud.google.com/bigquery?ws=!1m5!1m4!4m3!1snhu-hoc-data!2sUnigap_dataset!3sclean_sales_orders2)

[Processing table Returns](https://console.cloud.google.com/bigquery?sq=279214207888:b80cdb5b62d049248ce9896f8fa7f244)

⇒ **Result**: [clean_return_orders](https://console.cloud.google.com/bigquery?ws=!1m5!1m4!4m3!1snhu-hoc-data!2sUnigap_dataset!3sclean_return_orders)

[Processing table People](https://console.cloud.google.com/bigquery?sq=279214207888:55dd3c0320f54baf9f0f960e1d77c7a3)

⇒ **Result**: [clean_sales_staff](https://console.cloud.google.com/bigquery?ws=!1m5!1m4!4m3!1snhu-hoc-data!2sUnigap_dataset!3sclean_sales_staff)

### 1. Clean data
Table Returns và People không có lỗi sai, chỉ đổi tên cột. Do đó, ở bước đây mô tả cách làm sạch table Orders

- Cột **Order ID**: nhiều duplicates, không có primary key của bảng. Hướng giải quyết: kết hợp Order ID & Order Date & Customer ID thành một cột mới, cột này là primary key của bảng

![image](https://github.com/user-attachments/assets/26b8551c-b1ae-4b02-a4a3-1eb5e73ed375)


- Cột **Product ID**: một Product ID liên kết với nhiều Product Name. Hướng giải quyết: đánh số thứ tự cho các sản phẩm có cùng **Product ID**, sau đó kết hợp **STT + ProductID** (VD: 2xxxx) để tạo thành **New Product ID**. Mỗi **New Product ID** sẽ liên kết duy nhất với một **Product Name**

![image](https://github.com/user-attachments/assets/7b49515a-6010-409c-b57b-316051870d03)


### 2. Enrich data
Để thuận tiện cho việc visualization ở bước sau, ta cần thêm một số cột vào bảng Orders như sau:

- Cột **customer_type**: phân loại khách hàng active và new theo từng năm

![image](https://github.com/user-attachments/assets/bb81971e-b159-4c1f-a384-3a72b3a58073)

- Cột **delivery_days**: khoảng thời gian từ ngày đặt hàng (**order_date**) đến ngày giao hàng (**ship_date**)

- Cột **return**: Yes/Blank. Phân loại đơn hàng có bị trả lại không (kéo dữ liệu từ bảng Returns vào bảng Orders)

- Cột **year**: trích năm từ ngày đặt hàng (**order_date**)

Ngoài ra, tạo bảng **dim_date** chứa ngày từ thời điểm bắt đầu đến thời điểm kết thúc của bảng **Orders**. Mục đích tạo bảng này để đảm bảo các chart theo thời gian có dữ liệu date liên tục, không bị ảnh hưởng bởi dữ liệu date của bảng **Orders**

**DATA MODEL**

![image](https://github.com/user-attachments/assets/203fc590-93b4-4bb7-bdf2-a1743d42150d)

## III. VISUALIZE DATA

[Link Visualize Data here](https://github.com/user-attachments/files/19142217/project.pdf)

### A. Page **Sales & Profit Summary**: tổng quan về tình hình kinh doanh của AMAZON

Doanh thu được break down theo 3 dimensions:
- Theo thời gian (**_Revenue Growth Trend (MoM)_**
- Theo market (**_Revennue & Profit by Market_**)
- Theo category (**_Revenue & Profit by Category_**)

Xu hướng lựa chọn dịch vụ vận chuyển (**_Transaction of Ship Mode by Time_**)

### B. Page **Customer Analysis**: tổng quan về tình hình khách hàng của AMAZON
Tổng số lượng khách hàng qua thời gian (**_Total customer Over Time_**)

Tỷ lệ giữ chân khách hàng qua từng năm (**_Active & New Customer (YoY)_**)

_**NOTE**: Đã xác định cách phân loại customer type ở trên_

RFM segment được xét theo 4 khía cạnh:
- Theo đóng góp doanh thu, lợi nhuận (**_Revenue & Profit & Number by Segment_**)
- Theo category (**_Revenue of Category by Segment_**)
- Theo chi tiêu và lợi nhuận mang lại trên mỗi cá nhân (**_Spending & Profit per person_**)
- Theo khoảng thời gian vận chuyển (**_Delivery Days by Segment_**)

### C. Page **Loyal**: hành vi và xu hướng chi tiêu cụ thể của nhóm
Nơi ở của nhóm khách hàng (Country - city)

Cơ cấu segment trong nhóm (**_Num of Customer by Segment_**) và tỷ lệ đóng góp doanh thu (**_Revenue by Segment_**)

_**NOTE**: Phân biệt Segment này với RFM segment_

Xét về category:
- Số lượng category được bán ra (**_Total Product Sold of Category by Time_**) nhằm nắm được loại category KH thường mua
- Doanh thu & lợi nhuận & tỷ lệ trả hàng của từng category (**_Revenue, Profit & Return rate by Category_**) nhằm xác định xu hướng bất thường hay theo chu kỳ, từ đó có chiến lược phù hợp theo từng khoảng thời gian 
- Liệt kê top 10 sub-category bán chạy (**_Top 10 best-selling Sub Category_**)
- Liệt kê top 5 highest/loss profit sub-category (**_Top 5 Highest/Loss Profit_**) nhằm biết chính xác các sub category lợi nhuận nhất và kém lợi nhuận nhất

Xét về AOV (**_AOV by Time_**) (chi tiêu trung bình trên 1 đơn hàng) được break down ra để giải thích chi tiết những tháng có AOV bất thường (giảm đột ngột hoặc tăng đột ngột):
- Doanh thu & lợi nhuận qua thời gian (**_Revenue & Transactions by Time_**)
- Tỷ trọng sản phẩm bán ra trong tháng (**_Num of Product Sold by Category_**)

_**NOTE**: thông thường AOV thấp là do: 1. KH chi tiêu ít đi trong 1 đơn hàng hoặc 2. số lượng đơn giảm khiến doanh số sụt giảm, mức chi tiêu vẫn như cũ hoặc 3. tỷ trọng mặt hàng technology & furrniture bán ra giảm đột biến khiến doanh thu giảm sút (mặt hàng technology & furniture đóng góp gần 80% doanh thu trong 1 đơn hàng (**_Revenue of Cateogry per Transaction_**))_

Xét về Ship Mode (**_% Transaction with Ship Mode_**) và xét Ship Mode theo từng category (**_Ship Mode by Category_**) để nắm được loại hình vận chuyển thường dùng cho từng category

### Tương tự phân tích các metrics trên cho 4 nhóm còn lại (Potential loyalist, Champion, At-risk, New customer)

### V. INSIGHTS & RECOMMENDATION
Để biết thêm chi tiết về insights và recommendations, tham khảo tại đây: [AMAZON_PBI_Test.pdf](https://github.com/user-attachments/files/19142353/AMAZON_PBI_Test.pdf)

![image](https://github.com/user-attachments/assets/58536948-5bdc-4ee2-a222-56efd4bb26f3)

![image](https://github.com/user-attachments/assets/b67f9a7d-1ff8-4aa8-9e8b-876ae56dfeef)

![image](https://github.com/user-attachments/assets/9281e374-fb4d-40dd-b328-0284ffc5a623)

![image](https://github.com/user-attachments/assets/0aba2adc-0406-4f35-8063-ad0941c7232d)

![image](https://github.com/user-attachments/assets/bb6c335f-5ddb-40c5-ac01-66ab7af64258)
