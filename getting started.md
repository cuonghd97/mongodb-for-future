# Mongodb

## Các khái niệm
`document` là một đơn vị cơ bản trong mongodb, tương đương với row trong cơ sở dự liệu quan hệ,
`collection` là một nhóm các `document` khi đó `collection` có thể được hiểu là table
Mỗi `document` có một key là `_id`, nó là duy nhất trong một `collection`

### Document
Là một nhóm các key-value
`{"greeting" : "Hello, world!"}`
`key`: là một string, gồm các ký tự utf-8, với một số ngoại lệ:
+ key không được chứa \0 (ký tự trống)
+ Ký tự `.` và `$` chỉ được dùng trong một số trường hợp đặt biệt
Mongodb phân biệt kiểu dữ liệu và chữ hoa-thường
```
{"foo" : 3}
{"foo" : "3"}
{"foo" : 3}
{"Foo" : 3}
```
các document trên hoàn toàn khác nhau
Mongodb không chứa các key trùng nhau
### Collection
`collection` là một tập hợp các `document`
### Dynamic Schemas
`document` trong một `collection` có thể là nhiều dạng không nhất thiết phải giống nhau `key`
```
{"greeting" : "Hello, world!"}
{"foo" : 5}
```
Hai `document` trên có thể ở trong cùng một `collection`
Đặt tên `collection` gồm các ký tự utf-8, không chứa ký tự `\0`, không được bắt đầu bằng `system.`, không được chứa ký tự `$`
### Database
`Database` gồm các `collection`
Tên `database` gồm các ký tự utf-8
Tên `database` không phải là chuỗi rỗng
Tên `database` không chứa các ký tự ` /, \, ., ", *, <, >, :, |, ?, $, (dấu cách), or \0`
Nên đặt tên database là các ký tự viết thường

## Shell
Chạy lệnh `mongo` hoặc `mongo -u <username> -p <password> --authenticationDatabase admin`
Ta có thể thực hiện các phép toán, hoặc định nghĩa một hàm javascript
## Các kiểu dữ liệu
+ null: {'x': null}
+ boolean: {'x': true}
+ number: {'n': 123}
+ string: {'str': 'hello world'}
+ date: {'date': new Date()}
+ regular expression: {'x': /foobar/i}
+ array: {'arr': [1, 2, 3]}
+ Một document khác: {'x': {'n': 100}}
+ object id {'x': ObjectId()}
+ code: {"x" : function() { /* ... */ }}

## Create, Update, Delete document
### insert và lưu
`db.foo.insert({"bar" : "baz"})`
#### batch insert
`Batch insert` cho phép bạn truyền một mảng các document vào database
`> db.foo.batchInsert([{"_id" : 0}, {"_id" : 1}, {"_id" : 2}])`
### Remove document
`db.foo.remove()` câu lệnh này xóa các `document` trong `collection` có tên là `foo`
Cũng có thể xóa các `document` theo điều kiện:
`db.foo.remove({'name': 'bar'})`
