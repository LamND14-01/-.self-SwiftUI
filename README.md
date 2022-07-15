<h1>Why does \.self work for ForEach?</h1>

Khi bắt đầu với SwiftUI, Ta đã thấy có rất nhiều cách khác nhau mà `ForEach` sử dụng để tạo dynamic views, Nhưng tất cả chúng đều có 1 điểm chung: SwiftUI cần xác định từng dynamic view duy nhất để có thể animate hoặc update ui 1 cách chính xác.

Nếu một object conforms to `Identifiable` protocol thì Swiftui sẽ tự động sử dụng `id` property cho việc [uniquing](https://ue.termwiki.com/UE/uniquing). Nếu ta không sử dụng `Identifiable` thì ta có thể sử dụng 1 keypath cho 1 property mà ta biết là duy nhất, chẳng hạn như số ISBN của một cuốn sách. Nhưng nếu ta không conform to `Identifiable` hoặc ta không có 1 keypath riêng biệt, ta có thể sử dụng `\.self`.

Ta đã quen sử dụng `\.self` cho các type cơ bản như `Int` and `String`, like this:

```Swift
    List {
        ForEach([2, 4, 6, 8, 10], id: \.self) {
            Text("\($0) is even")
        }
    }
```

Với Core Data ta có thể sử dụng 1 identifier duy nhất nếu ta muốn, nhưng ta cũng có thể sử dụng `\.self` nó cũng một phần hoạt động tốt.

Khi sử dụng `\.self` như một identifier, có nghĩa là “the whole object”, Nhưng điều đó không có nghĩa nhiều - a struct is a struct, vì vậy nó không có bất kỳ thông tin nhận dạng cụ thể nào ngoài content của nó. Vì vậy bản chất ở đây là Swift đã tính toán  *hash value* của struct, nghĩa là cách thể hiện của dữ liệu phức tạp trong *fixed-size values*, sau đó sử dụng hash như một identifier.

Hash values có thể được tạo theo bất kỳ cách nào, nhưng ý tưởng của tất cả hash-generating đều giống nhau:

1. Bất kể kích thước đầu vào như thế nào đầu ra phải phải có cùng kích thước cố định.
2. Tính toán cùng một hash cho một object 2 lần liên tiếp sẽ trả về cùng một giá trị.

Ví dụ: nếu ta lấy hash của “Hello World” và hash của tác phầm hoàn chỉnh của Shakespeare, cả 2 sẽ có cùng 1 kích thước. Điều này có nghĩa là không thể covert hash về giá trị bạn đầu của nó – ta không thể convert 40 như chữ cái thập phân và chữ số về tác phẩm hoàn chỉnh của Shakespeare.

Hash thường được sử dụng cho việc xác minh dữ liệu. Ví dụ, nếu bạn download 8GB zip file, bạn có thể check xem nó có đúng không bằng cách so sánh với local hash của tệp đó với trên sever – nếu nó giống nhau, nghĩa là tệp zip đó giống hệt nhau. Hashe cũng được sử dụng với dictionary keys and sets.

Tất cả điều trên đều quan trọng vì khi xcode tạo ra class để quản lý object, nó làm class conform to `Hashable`, protocol giúp Swift có thể tạo hash value cho nó, điều đó có nghĩa là chúng ta có thể sử dụng `\.self` để định danh cho class. Đây là lý do tại sao `String` và `Int` hoạt động được với `\.self`: nó cũng conform to `Hashable`.

`Hashable` có một chút giống `Codable`: Nếu chúng ta muốn tạo custom type conform to `Hashable`, thì mọi thứ nó chứa cũng conforms to `Hashable`, ta không cần làm thêm gì nữa. Để chứng minh, ta có thể tạo custom struct that conforms to `Hashable` thay vì `Identifiable`, và sử dụng `\.self` để xác định nó:

```Swift
    struct Student: Hashable {
        let name: String
    }
    
    struct ContentView: View {
        let students = [Student(name: "Harry Potter"), Student(name: "Hermione Granger")]
    
        var body: some View {
            List(students, id: \.self) { student in
                Text(student.name)
            }
        }
    }
```

Ta tạo struct `Student` conform to `Hashable` vì tất cả các properties của nó đã conform to `Hashable`, Swift sẽ tính toán hash values của từng property sau đó kết hợp chúng thành một hash đại diện cho toàn bộ struct. Tất nhiên, nếu chúng ta kết thúc với hai sinh viên có cùng tên, chúng ta sẽ gặp vấn đề, giống như nếu chúng ta có một array String với hai String hệt nhau.

Bây giờ, bạn có thể nghĩ rằng điều này dẫn đến một vấn đề: nếu ta tạo 2 Core Data objects với value giống nhau, nó sẽ tạo ra hash giống nhau, ta sẽ gặp sự cố khi làm việc với animation. Tuy nhiên, Core Data khá là thông mình: các objects mà nó tạo cho chúng ta thực ra là một lựa chọn khác của properties ngoài tầm mà ta định nghĩa trong our data model, bao gồm một thứ được gọi là object ID – một identifier duy nhất của object, bất kể nó chứa properties nào. IDs đó tương tự như một `UUID`, Core Data tạo ra chúng tuần tự khi chúng ta tạo ra các object.

Vì thế, `\.self` hoạt động với bất cứ thứ gì conforms to `Hashable`, vì Swift sẽ tạo hash value cho object và sử dụng nó để định danh nó. Nó cũng hoạt động Core Data’s objects vì nó cũng conform to `Hashable`. vì vậy, Nếu bạn muốn sử dụng một identifier cụ thể điều đó rất tốt, nhưng bạn không cần tạo vì `\.self` cũng là một option tốt.

**Warning:** Mặc dù việc tính toán cùng hash cho 1 object 2 lần một lúc có thể trả về value giống nhau, tính toán nó giữa hai lần chạy ứng dụng của bạn (vd, tính toán hash, tắt ứng dụng, khởi động lại ứng dụng, tính toán lại hash lại lần nữa) có thể hash giữa hai lần sẽ khác nhau.

<h2>Identifiable</h2>
Ngoài việc sử dụng `\.self` ta cũng có thể sử dụng thêm protocol `Identifiable` thay thế cho việc đăng ký id. Tất cả những gì ta cần làm đó là add `Identifiable` vào list protocol mà struct của ta cần conform theo. Đây là một protocol của Swift, giúp nó xác định được type của struct là duy nhất. Nó chỉ có một yêu cầu đó là phải có một property có tên là `id` chứa một định danh duy nhất. ví dụ:

```Swift
struct ExpenseItem: Identifiable {
    let id = UUID()
    let name: String
    let type: String
    let amount: Int
}
```
Bây giờ, bạn có thể tự hỏi tại sao chúng tôi phải thêm điều đó, bởi vì code đã hoạt động tốt trước đây rồi. Bởi vì *ExpenseItem* giờ dây đã được đảm là có thể xác định duy nhất, ta sẽ không cần phải báo cho `List` property nào sẽ được dùng để định danh, nó sẽ biết có một property `id` và nó là duy nhất. Đó là mấu chốt của protocol `Identifiable`.
Do đó ta có thể viết lại như sau:

```Swift
List(expenseItems) { item in
    Text(item.name)
}
```