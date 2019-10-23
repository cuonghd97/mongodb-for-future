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
`$limit`: đưa ra `n` document vào pipeline trong đó `n` là số lượng chỉ định.  
`$lookup`: thực hiện phép left join tới một collection khác trong cùng database.  
`$match`: lọc luồng các document và chỉ cho phép document thỏa mãn điều kiện được truyền qua giai đoạn tiếp theo. Với mỗi document đưa vào dữ liệu trả về là một document trong trường hợp thỏa mãn, hoặc không document trong trường hợp không thỏa mãn.  
`$merge`: viết các kết quả document tổng hợp vào một collection. Giai đoạn này có thể kết hợp(chèn document mới, gôp document, thay thế document, sao lưu những document đã tồn tại, `fail the operation`, xử lý document với pipeline cập nhật) kết quả đầu ra là một collection. `$merge` phái là giai đoạn cuối cùng. `$merge` được đưa ra tại bản mongo 4.2  
`$out`: các document kết quả được tổ chức thành collection. `$out` phải là giai đoạn cuối cùng. `$out` được đưa ra ở bản mongo 2.6  
`$project`: định hình lại mỗi document trong luồng, như: thêm trường mới, xóa bỏ trường đã tồn tại. Với mỗi document đầu vào chỉ có một document được trả về.  
`$redact`: định hình lại từng document trong luồng bằng cách giới hạn nội dung cho từng document, dựa trên những thuộc tính cả document đó. Nó là sự kết hợp của `$match` và `$project`.  
`$replaceRoot`: thay thế một doument với document mới. Hành động này có thể thay thế toàn bộ các trường đang tồn tại trong document đầu vào, gồm cả trường id.  
`$replaceWith`: là một bí danh của `$replaceRoot` hoạt động như `$replaceRoot`  
`$sample`: chọn ngẫu nhiên số lượng document được chỉ định từ các document đầu vào.  
`$set`: thêm trường mới vào document. Giống như `$project`, `$set` cũng định hình lại mỗi document trong luồng. Cụ thể là thêm trường mới vào document đầu ra mà có cả các trường đang tồn tại. `$set` mà một bí danh của `$addField`.  
`$skip`: bỏ qua `n` document đầu tiên.  
`$sort`: sắp xếp lại các document theo một key chỉ định.  
`$sortByCount`: gom nhóm các document theo mọt trường cụ thể và đếm số lần xuất hiện các giá trị trong trường đó.  
`$unset`: xóa bỏ trường đã tồn tại.  
`$unwind`: chia một document có value là mảng thành nhiều document.  

`$literal`: trả về giá trị mà không cần phân tích cú pháp. Vì mongodb phân tích cú pháp bắt đầu bằng ký tự `$`. Để tránh trường hợp mongodb không phân tích giá trị nào ta dùng `$literal`.  

# Các biểu thức toán tử
`$abs`: giá trị tuyệt đối.  
`$add`: cộng thêm giá trị số, đối với kiểu dữ liệu `date` thì số được cộng vào được coi là mili giây.  
`$ceil`: trả về số nguyên nhỏ nhất lớn hơn hoặc bằng một số cụ thể.  
`$devine`: trả về kết quả của phép chia.  
`$exp`: tăng `e` lên số mũ quy định.  
`$floor`: trả về số nguyên lớn nhất nhỏ hơn hoặc bằng một số cụ thể.  
`$ln`: tính log tự nhiên của một số.  
`$log10`: log cơ số 10 của một số.  
`$mod`: trả về phần dư.  
`$multiply`: tích của 2 số.  
`$pow`: tính số mũ.  
`$round`: làm tròn số thực đến phần thập phân cụ thể.  
`$sqrt`: phép khai căn.  
`$subtract`: phép trừ, nếu 2 giá trị là `date` trả về kết quả là mili giây, nếu số trừ kiểu date số bị trừ là số kết qua sẽ là `date` mới.  
`$trunc`: cắt một số thành số nguyên hoặc đến một vị trí thập phân được chỉ định.  
# Các biểu thức với mảng
`$arrayElemAt`: trả về giá trị từ một vị trí cụ thể.  
`$arrayToObject`: chuyển một mảng các cặp key-value thành document.  
`$concatArrays`: nối mảng, và trả về mảng đã được nối.  
`$filter`: đưa ra mảng con thỏa mãn điều kiện cho trước.  
`$in`: trả về kiểu boolean, kiểm tra giá trị có tồn tại trong mảng.  
`$indexOfArray`: nếu giá trị xuất hiện trọng mảng sẽ trẩ về vị trí của nó, ngược lại trả về -1.  
`$isArray`: kiểm tra xem đó có phải là một mảng hay không.  
`$map`: tạo ra một mảng mới các giá trị phần tử được thay đổi theo các xử lý trong `$map`.  
`$objectToArray`: chuyển một document thành mảng.  
`$range`: đƯa ra một mảng gồm danh sách các số nguyên theo yêu cầu được xác định trước.  
`$reduce`: ấp dụng một biểu thức trong cho mỗi phần tử trong mảng và kết hợp chúng thành giá trị duy nhất.  
`$reverseArray`: trả về mảng mới với thứ tự các phần tử ngược lại.  
`$size`: trả về số lượng phần tử của một mảng.  
`$slice`: cắt mảng.  
`$zip`: gộp 2 mảng.  
# Toán tử biểu thức boolean
`$and`, `$not`, `$or`  
# Các toán tử so sánh
Các toán tử so sánh để trả về `boolean` ngoại trừ `$cmp`  
`$cmp`: trả về 0 nếu 2 giá trị bằng nhau, 1 nếu giá trị thứ nhất lớn hơn giá trị thứ 2, -1 nếu giá trị thứ nhất nhơ hơn.  
`$eq`: trả về `true` nếu 2 giá trị bằng nhau.  
`$gt`: trả về `true` nếu giá trị đầu tiên lớn hơn giá trị thứ 2.  
`$gte`: trả về `true` nếu giá trị đầu tiên lớn hơn hoặc bằng giá trị thứ 2.  
`$lt`: trả về `true` nếu giá trị đầu tiên nhỏ hơn giá trị thứ 2.  
`$lte`: trả về `true` nếu giá trị đầu tiên nhỏ hơn hoặc bằng giá trị thứ 2.  
`$ne`: trả về `true` nếu 2 giá trị khác nhau.  
# Các toán tử biểu thức có điều kiện
`$cond`: giống câu lệnh `if-else`  
`$ifNull`: trả về kết quả khác null của biểu thức đầu tiên hoặc kết quả của biểu thức thứ 2 nếu biểu thức đầu tiên trả về null.  
`$switch`: giống với khối lệnh `switch` trong lập trình  
# Các toán tử với ngày tháng
`$dateFromParts`: xây dựng một đối tượng ngày bson với các phần cấu thành ngày.  
`$dateFromString`: chuyển đổi `date` từ dạng `string` sang `date object`.  
`$dateToParts`: trả về document gồm các thành phần cấu thành của một ngày.  
`$dateToString`: trả về ngày tháng năm dạng `string`.  
`$dayOfMonth`: trả về ngày trong tháng trong khoảng 1-31.  
`$dayOfWeek`: trả về các ngày trong tuần 1(Chủ nhật) - 7(Thứ 7).  
`$dayOfYear`: trả về ngày trong năm từ 1-366.  
`$hour`: trả về thời gian trong ngày từ 0-23.  
`$isoDayOfWeek`: trả về thứ tự ngày trong tuần theo chuẩn ISO 8601. 1(thứ 2) - 7(chủ nhật).  
`$milisecond`: trả về mili giây từ 0 - 999.  
`$minute`: trả về phút từ 0-59.  
`$month`: trả về tháng từ 1 - 12.  
`$second`: trả về giây từ 0 - 60.  
`$toDate`: chuyển đổi giá trị thành `date`.  
`$week`: trả về số thứ tự của tuần từ 0 - 53.  
`$year`: trả về năm dưới dạng số.  
# Các toán tử với set
`$allElementsTrue`: trả về `true` nếu không có giá trị nào bằng `false`.  
`$anyElementTrue`: trả về `true` nếu có bất cứ giá trị nào bằng `true`, ngược lại trả về `false`.  
`$setEquals`: trả về `true` nếu set đầu vào chỉ có các phần tử khác nhau.  
`$setIntersection`: trả về 1 set là giao của 2 set.  
`$setIsSubset`: trả về `true` nếu tất cả phần tử của set thứ nhất thuộc set thứ 2.  
`$setUnion`: trả về một set với toàn bộ phần tử của các set được đưa vào.  
# Các toán tử với chuỗi
`$concat`: thực hiện phép nối chuỗi.  
`$dateFromString`: chuyển ngày tháng dạng chuỗi thành date object.  
`$dateToString`: trả về ngày đã được định dạng thành chuỗi.  
`$indexOfBytes`: trả về chỉ số byte utf8 của chuỗi con nếu chuỗi con đó xuất hiện. Ngược lại trả về -1.  
`$indexOfCP`: trả về chỉ số utf8 của chuỗi con đó nếu xuất hiện, ngược lại trả về -1.  
`$ltrim`: xóa các khoảng trắng, hoặc các ký tự được chỉ định từ đầu chuỗi.  
`$regexFind`: Áp dụng biểu thức chính quy (regex) cho chuỗi và trả về thông tin trên chuỗi con phù hợp đầu tiên.  
`$regexFindAll`: Áp dụng biểu thức chính quy (regex) cho chuỗi và trả về thông tin trên tất cả các chuỗi con phù hợp.  
`$regexMatch`: Áp dụng biểu thức chính quy (regex) cho chuỗi và trả về giá trị boolean tương ứng với có chuỗi con thỏa mãn hay không.  
`$rtrim`: xóa các khoảng trắng, hoặc các ký tự được chỉ định từ cuối chuỗi.  
`$split`: cắt chuỗi dựa trên một dấu phân cách, kết qủa trả về là một mảng các chuỗi được cắt.  
`$strLenBytes`: trả về số byte mã hóa của chuỗi.  
`$strLenCP`: trả về số mã utf8 trong một chuỗi.  
`$strcasecmp`: thực hiện so sánh chuỗi và không phân biệt hoa thường: 0 nếu hai chuỗi bằng nhau, 1 nếu chuỗi thứ nhất lớn hơn chuỗi thứ 2, còn lại là -1.  
`$substrBytes`: trả về một chuỗi con bắt đầu bằng vị trí của ký tự utf8 dưới dạng byte và tiếp tục cho số byte được chỉ định.  
`$substrCP`: trả về một chuỗi con. Bắt đầu với ký tự được chỉ định trong chuỗi và số lượng ký tự được chỉ định.  
`$toLower`: chuyển chuỗi thành chuỗi toàn chữ thường.  
`$toString`: chuyển một giá trị thành chuỗi.  
`$trim`: Xóa các ký tự trắng thừa và ký tự được chỉ định trong cả chuỗi.  
`$toUpper`: chuyển chuỗi thành chuỗi toàn ký tự in hoa.  