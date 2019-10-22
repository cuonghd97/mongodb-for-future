# Aggeration Pipeline
Aggeration framework của mongodb được mô hình hóa dựa trên khái niệm đường ống xử lý dữ liệu. Các document được đưa vào đường ống qua nhiều giai đoạn biến đổi thành kết quả cuối cùng.  
Ví dụ:  
```
db.orders.aggregate([
   { $match: { status: "A" } },
   { $group: { _id: "$cust_id", total: { $sum: "$amount" } } }
])
```  
Giai đoạn đầu: `$match` lọc các document theo trường `status` và chuyển sang giai đoạn tiếp theo với những document thỏa mãn điều kiện `status` bằng `A`.  
Giai đoạn tiếp theo: `$group` nhóm các document theo trường `cust_id` để tính tổng theo mỗi `cust_id`  
Hầu hết các giai đoạn cung cấp các filter thực hiện như các câu lệnh query và sửa đổi hình thức của document đầu ra.  
Một số pipeline cung cấp các công cụ cho việc nhóm và sắp xếp các document theo trường chỉ định cũng như công cụ tổng hợp nội dung của một mảng, gồm mảng các document. Hơn nữa, các giai đoạn pipeline có thể thực hiện các công việc như tính trung bình hay nối chuỗi.  
Aggregation pipeline có thể thực hiện trên cả sharded document.  
Aggregation có thể sử dụng index để tăng hiệu suất trong một vài giai đoạn. Thêm vào đó aggregation có giai đoạn tối ưu hóa nội bộ.  

# Quick reference
Tại phương thức `db.collection.aggregate` các giai đoạn pipline được đặt trong một mảng. Document được truyền qua các giai đoạn theo trình tự. Ngoại trừ `$out`, `$merge`, `$geoNear` có thể xuất hiện nhiều lần trong pipeline.  
`db.collection.aggregate( [ { <stage> }, ... ] )`  
Các giai đoạn:  
`$addField`: Thêm một trường mới vào document. Giống `$project`, `$addField` định hình lại document luồng; đặc biệt, bằng cách thêm trường mới vào document đầu ra mà có thể bao gồm những trường đã tồn tại từ document đầu vào và những trường mới được thêm vào.  
> `$set` là một bí danh của `$addField`  

`$bucket`: phân loại các document thành các nhóm, được gọi là các bucket, dựa trên các biểu hiện cụ thể và các ranh giới bucket.  
`$bucketAuto`: phân loại các document thành các nhóm với số lượng nhóm cụ thể, được gọi là bucket, dựa trên các biểu hiện cụ thể. Ranh giới các bucket được tính toán tự động trong một nỗ lực để đồng đều các document vào một số lược bucket được chỉ định.  
`$collStats`: trả về số liệu thống kê liên quan đến collection hoặc view.  
`$count`: trả về số lượng của document trong giai đoạn đó trong aggregation pipeline.  
`$facet`: Xử lý nhiều pipeline aggregation trong một giai đoạn trên cùng một bộ document đầu vào.  
`$geoNear`: Trả về một luồng document the thứ tự dựa trên sự gần gũi với một điểm trong không gian địa lý. Kết hợp với `$match`, `$sort`, `$limit` cho dữ liệu không gian địa lý. Dữ liệu đầu ra bao gồm một trường khoảng cách bổ sung và có thể bao gồm trường định danh vị trí.  
`$graphLookup`: Thực hiện tìm kiếm đệ quy trên một collection. Với mỗi document đầu ra thêm một trường mảng mới chứa kết quả truyền tải của tìm kiếm đệ quy trong document đó.  
`$group`: nhóm các document đầu vào bởi một biểu thức định danh được chỉ định và áp dụng các biểu thức tích lũy.  
`$indexStats`: trả về số liệu thống kê liên quan đến việc sử dụng từng index cho collection.  
