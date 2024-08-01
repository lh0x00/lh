## Hướng dẫn quy chuẩn khi làm việc với Golang

Tài liệu này tổng hợp các quy chuẩn khi viết code Golang theo hướng dẫn của Google. Nội dung tập trung vào các tình huống thường gặp, giúp code dễ đọc, dễ bảo trì và dễ sử dụng. Bài viết được chia thành các phần chính với phân tích sâu và ví dụ minh họa.

### 1. Đặt tên (Naming)

Tên biến, hàm, gói đóng vai trò quan trọng trong việc thể hiện ý nghĩa và mục đích của code. Đặt tên rõ ràng, nhất quán giúp code dễ hiểu và dễ bảo trì.

#### 1.1. Hàm và phương thức (Function and Method Names)

##### 1.1.1. Tránh lặp lại (Avoid Repetition)

Tên hàm và phương thức nên ngắn gọn, tập trung vào hành động hoặc kết quả trả về. Tránh lặp lại thông tin đã có sẵn từ tên gói, kiểu dữ liệu, kiểu con trỏ, tên biến truyền vào, tên và kiểu dữ liệu trả về.

**Ví dụ:**

```go
// Sai: Lặp lại tên gói "yamlconfig" và kiểu dữ liệu "Config"
package yamlconfig

func ParseYAMLConfig(input string) (*Config, error)

// Đúng: Ngắn gọn, tập trung vào hành động "Parse"
package yamlconfig

func Parse(input string) (*Config, error) 
```

**Phân tích:**

* Tên gói `yamlconfig` đã thể hiện rõ chức năng liên quan đến cấu hình YAML.
* Kiểu dữ liệu trả về `*Config` cũng đã được xác định trong khai báo hàm.
* Việc lặp lại thông tin này trong tên hàm gây dài dòng, khó đọc.

##### 1.1.2. Quy ước đặt tên (Naming Conventions)

Ngoài việc tránh lặp lại, cần tuân theo một số quy ước đặt tên sau:

* **Hàm trả về giá trị:** Dùng danh từ để thể hiện kết quả trả về. Ví dụ: `JobName`, `UserData`.
* **Hàm thực hiện hành động:** Dùng động từ để thể hiện hành động của hàm. Ví dụ: `WriteTo`, `ProcessData`.
* **Hàm giống nhau chỉ khác kiểu dữ liệu:** Thêm tên kiểu dữ liệu vào cuối tên hàm để phân biệt. Ví dụ: `ParseInt`, `ParseInt64`, `ConvertToString`, `ConvertToInt`.
* **Hàm chính:** Có thể bỏ qua tên kiểu dữ liệu nếu hàm là phiên bản chính, được sử dụng phổ biến nhất. Ví dụ: `Marshal`, `Unmarshal`.

**Ví dụ:**

```go
// Hàm trả về giá trị:
func (c *Config) JobName() (string, error)

// Hàm thực hiện hành động:
func (u *User) UpdateProfile(data *ProfileData) error

// Hàm giống nhau, khác kiểu dữ liệu:
func ParseInt(input string) (int, error)
func ParseInt64(input string) (int64, error)

// Hàm chính:
func (d *Data) Marshal() ([]byte, error)
```

#### 1.2. Gói và kiểu dữ liệu giả lập kiểm thử (Test Double Packages and Types)

Kiểm thử đóng vai trò quan trọng trong đảm bảo chất lượng code. Việc sử dụng các test double (stub, fake, mock, spy) giúp kiểm tra code độc lập với các thành phần khác. 

##### 1.2.1. Tạo gói hỗ trợ kiểm thử (Creating Test Helper Packages)

Gói chứa test double nên được đặt trong gói riêng biệt với gói sản phẩm, thường bằng cách thêm hậu tố `_test` vào tên gói.

**Ví dụ:**

Gói sản phẩm: `creditcard`

Gói kiểm thử: `creditcard_test`

##### 1.2.2. Đặt tên test double (Naming Test Doubles)

* **Đơn giản (Simple):** Nếu chỉ có một kiểu dữ liệu cần giả lập, dùng tên đơn giản như `Stub`, `Fake`. 
* **Theo hành vi (Behavioral):** Nếu có nhiều hành vi cần giả lập, đặt tên theo hành vi mong muốn. Ví dụ: `AlwaysCharges`, `AlwaysDeclines`, `ReturnsError`.
* **Rõ ràng (Explicit):** Nếu có nhiều kiểu dữ liệu cần giả lập, đặt tên rõ ràng theo kiểu dữ liệu. Ví dụ: `StubService`, `StubStoredValue`, `FakeDatabase`.

**Ví dụ:**

```go
// Gói creditcard_test
package creditcard_test

// Stub đơn giản cho creditcard.Service
type Stub struct{}

func (Stub) Charge(*creditcard.Card, money.Money) error { return nil }

// Stub với hành vi cụ thể:
type AlwaysDeclines struct{}

func (AlwaysDeclines) Charge(*creditcard.Card, money.Money) error {
    return creditcard.ErrDeclined
}

// Stub cho nhiều kiểu dữ liệu:
type StubService struct{}

func (StubService) Charge(*creditcard.Card, money.Money) error { return nil }

type StubStoredValue struct{}

func (StubStoredValue) Credit(*creditcard.Card, money.Money) error { return nil }
```

##### 1.2.3. Biến cục bộ trong kiểm thử (Local Variables in Tests)

Khi sử dụng test double trong hàm kiểm thử, nên thêm tiền tố vào tên biến để phân biệt với kiểu dữ liệu sản phẩm.

**Ví dụ:**

```go
// Gói payment_test
package payment_test

func TestProcessor(t *testing.T) {
    // Sử dụng tiền tố "spy" cho biến test double
    var spyCC creditcard_test.Spy 
    proc := &Processor{CC: spyCC}

    // ...
}
```

#### 1.3. Ghi đè và Che khuất (Stomping and Shadowing)

Golang cho phép gán giá trị mới cho biến đã khai báo (ghi đè). Khi dùng toán tử `:=` trong phạm vi mới, một biến mới cùng tên được tạo ra, che khuất biến cũ (che khuất).

**Ghi đè (Stomping):**

```go
// Giá trị của i được ghi đè
i := 10
i := 20 
```

**Che khuất (Shadowing):**

```go
x := 10

if true {
  x := 20 // Biến x mới được tạo ra, che khuất biến x bên ngoài
  fmt.Println(x) // In ra 20
}

fmt.Println(x) // In ra 10
```

**Phân tích:**

* Ghi đè giúp code ngắn gọn hơn khi giá trị cũ không còn cần thiết.
* Che khuất có thể gây nhầm lẫn, cần cẩn thận khi sử dụng.

#### 1.4. Gói tiện ích (Util Packages)

Gói tiện ích chứa các hàm, kiểu dữ liệu dùng chung cho nhiều gói khác. Nên tránh đặt tên chung chung như `util`, `helper`, `common`, thay vào đó dùng tên miêu tả chức năng cụ thể của gói.

**Ví dụ:**

```go
// Sai: Tên gói không rõ ràng
package util

// Đúng: Tên gói thể hiện chức năng
package stringutil
package timeutil
```

#### 1.5. Kích thước gói (Package Size)

Nên chia nhỏ gói theo chức năng để dễ quản lý và sử dụng. Mỗi gói chỉ nên tập trung vào một chức năng cụ thể, tránh gói quá lớn và chứa nhiều chức năng không liên quan.

### 2. Nhập khẩu (Imports)

Phần `import` khai báo các gói được sử dụng trong file hiện tại. Nên tuân theo quy ước đặt tên, thứ tự và nhóm nhập khẩu để code dễ đọc và dễ bảo trì.

#### 2.1. Proto và stub (Protos and Stubs)

Khi làm việc với Protobuf, các gói được tạo ra cần được đổi tên để phù hợp với quy ước đặt tên trong Golang.

**Quy ước đặt tên:**

* `pb`: Hậu tố cho gói được tạo bởi `go_proto_library`.
* `grpc`: Hậu tố cho gói được tạo bởi `go_grpc_library`.

**Tiền tố:**

* Dùng tiền tố ngắn (1-2 chữ cái) để phân biệt các gói. Ví dụ: `fspb`, `fsgrpc`.
* Bỏ qua tiền tố nếu chỉ dùng 1 proto hoặc gói liên quan chặt chẽ với proto.
* Dùng tiền tố từ ngắn nếu tên proto chung chung hoặc khó hiểu. Ví dụ: `mapspb`.

**Ví dụ:**

```go
// Nhập khẩu gói proto và gRPC
import (
    fspb "path/to/package/foo_service_go_proto"
    fsgrpc "path/to/package/foo_service_go_grpc"
)
```

#### 2.2. Thứ tự nhập khẩu (Import Ordering)

Nên nhóm và sắp xếp các gói theo thứ tự sau:

1. **Gói thư viện chuẩn:** Ví dụ: `fmt`, `os`, `io`.
2. **Gói dự án:** Ví dụ: `github.com/user/myproject`, `internal/mypackage`.
3. **(Tùy chọn) Gói Protobuf:** Ví dụ: `pb "path/to/foo_go_proto"`.
4. **(Tùy chọn) Gói tác dụng phụ:** Ví dụ: `_ "path/to/package"`.

**Ví dụ:**

```go
import (
    "fmt"
    "os"

    "github.com/user/myproject"

    pb "path/to/foo_go_proto"

    _ "path/to/package"
)
```

#### 2.3. Nhập khẩu "trống" (Blank Imports)

Nhập khẩu "trống" (`import _`) chỉ nên dùng trong gói `main` hoặc trong kiểm thử. Tránh dùng trong thư viện để kiểm soát phụ thuộc và dễ dàng kiểm tra.

**Ví dụ:**

```go
// Trong gói main, nhập khẩu image/jpeg để sử dụng tác dụng phụ
package main

import (
    _ "image/jpeg"
)
```

#### 2.4. Nhập khẩu "chấm" (Dot Imports)

Nhập khẩu "chấm" (`import .`) cho phép sử dụng tên hàm, biến từ gói khác mà không cần tiền tố. Nên tránh dùng vì khó xác định nguồn gốc của hàm, biến.

### 3. Xử lý lỗi (Error Handling)

Golang sử dụng `error` để báo hiệu lỗi trong quá trình thực thi. Nên tuân theo quy ước trả về lỗi, tạo chuỗi lỗi và xử lý lỗi để code dễ đọc và dễ bảo trì.

#### 3.1. Trả về lỗi (Returning Errors)

* Dùng `error` là tham số trả về cuối cùng của hàm để báo hiệu hàm có thể lỗi.
* Trả về `nil error` để báo hiệu hàm thực thi thành công.
* Tránh trả về kiểu lỗi cụ thể, nên dùng `error` để tránh lỗi con trỏ `nil` trong interface.

**Ví dụ:**

```go
// Hàm trả về lỗi:
func ReadFile(filename string) ([]byte, error)

// Hàm trả về nil error:
func ConnectToDatabase() error
```

#### 3.2. Chuỗi lỗi (Error Strings)

* Không viết hoa chuỗi lỗi, trừ khi bắt đầu bằng tên viết hoa, danh từ riêng hoặc từ viết tắt.
* Không kết thúc chuỗi lỗi bằng dấu câu.

**Ví dụ:**

```go
// Sai: Viết hoa và kết thúc bằng dấu chấm
return fmt.Errorf("Error: file not found.")

// Đúng: Không viết hoa và không kết thúc bằng dấu câu
return fmt.Errorf("file not found") 
```

#### 3.3. Xử lý lỗi (Handling Errors)

Khi gặp lỗi, nên xử lý theo một trong các cách sau:

* Xử lý lỗi ngay lập tức.
* Trả về lỗi cho hàm gọi.
* Trong trường hợp ngoại lệ, dùng `log.Fatal` hoặc `panic`.

**Ví dụ:**

```go
// Xử lý lỗi ngay lập tức:
data, err := ReadFile("data.txt")
if err != nil {
    fmt.Println("Error reading file:", err)
    return
}

// Trả về lỗi cho hàm gọi:
func ProcessData(data []byte) error {
    // ...
    if err != nil {
        return fmt.Errorf("error processing data: %w", err)
    }
    // ...
}

// Dùng log.Fatal:
if err := ConnectToDatabase(); err != nil {
    log.Fatal("Error connecting to database:", err)
}
```

#### 3.4. Lỗi trong băng tần (In-Band Errors)

Tránh sử dụng giá trị đặc biệt (ví dụ: -1, `nil`, chuỗi rỗng) để báo hiệu lỗi. Thay vào đó, nên trả về thêm một giá trị để báo hiệu kết quả trả về có hợp lệ.

**Ví dụ:**

```go
// Sai: Trả về -1 khi không tìm thấy khóa
func Lookup(key string) int

// Đúng: Trả về giá trị boolean để báo hiệu tìm thấy khóa
func Lookup(key string) (string, bool)
```

#### 3.5. Thụt lề luồng lỗi (Indent Error Flow)

Xử lý lỗi trước khi tiếp tục thực thi code. Thụt lề luồng xử lý lỗi giúp code dễ đọc và dễ hiểu.

**Ví dụ:**

```go
// Sai: Luồng xử lý lỗi bị thụt lề, khó đọc
if err != nil {
    // Xử lý lỗi
} else {
    // Luồng xử lý bình thường
}

// Đúng: Xử lý lỗi trước, luồng xử lý bình thường không bị thụt lề
if err != nil {
    // Xử lý lỗi
    return
}

// Luồng xử lý bình thường
```

### 4. Ghi log (Logging)

Ghi log là cách hữu hiệu để theo dõi và gỡ lỗi chương trình. Nên tuân theo quy ước ghi log, mức độ log và xử lý lỗi khi ghi log.

#### 4.1. Tránh lặp lại (Avoid Duplication)

Chỉ nên ghi log lỗi ở hàm gọi, tránh ghi log nhiều lần ở các hàm con. 

**Ví dụ:**

```go
// Sai: Ghi log lỗi ở cả hàm con và hàm gọi
func processData(data []byte) error {
    if err != nil {
        log.Error("Error processing data:", err)
        return err
    }
}

func main() {
    // ...
    if err := processData(data); err != nil {
        log.Error("Error processing data:", err)
        // ...
    }
}

// Đúng: Chỉ ghi log lỗi ở hàm gọi
func processData(data []byte) error {
    if err != nil {
        return err
    }
}

func main() {
    // ...
    if err := processData(data); err != nil {
        log.Error("Error processing data:", err)
        // ...
    }
}
```

#### 4.2. Bảo mật (PII)

Không nên ghi thông tin cá nhân (PII) vào log. 

#### 4.3. Mức độ log (Log Levels)

* Dùng `log.Error` hạn chế: Ghi log mức `ERROR` tốn kém hơn các mức khác.
* Mức độ log tùy chỉnh: Sử dụng `log.V` để ghi log chi tiết cho phát triển và gỡ lỗi.

#### 4.4. Khởi tạo chương trình (Program Initialization)

Khi gặp lỗi trong quá trình khởi tạo chương trình, nên ghi log bằng `log.Exit` và giải thích rõ cách khắc phục lỗi.

#### 4.5. Kiểm tra chương trình và `panic` (Program Checks and Panics)

* Ưu tiên trả về lỗi thay vì `panic` để xử lý lỗi.
* Dùng `log.Fatal` để dừng chương trình khi gặp lỗi nghiêm trọng.
* Tránh phục hồi `panic` để tránh lan truyền trạng thái lỗi.

### 5. Tài liệu (Documentation)

Tài liệu code đóng vai trò quan trọng trong việc giúp người khác hiểu và sử dụng code.

#### 5.1. Quy ước (Conventions)

* Tài liệu rõ ràng, đầy đủ, dễ hiểu.
* Sử dụng ví dụ chạy được để minh họa cách sử dụng code.
* Chỉ tài liệu cho các trường, tham số dễ gây lỗi hoặc khó hiểu.
* Tài liệu rõ ràng về hành vi của `context`, `concurrency`, `cleanup` và `errors`.
* Xem trước tài liệu Godoc để kiểm tra định dạng.

#### 5.2. Định dạng Godoc (Godoc Formatting)

* Dùng dòng trống để tách các đoạn văn.
* Đặt ví dụ chạy được trong file kiểm thử.
* Thụt lề 2 khoảng trắng để định dạng nguyên văn code.
* Dòng bắt đầu bằng chữ hoa được định dạng thành tiêu đề.

### 6. Khai báo biến (Variable Declarations)

#### 6.1. Khởi tạo (Initialization)

* Dùng `:=` khi khởi tạo biến với giá trị khác 0.
* Dùng `var` khi khởi tạo biến với giá trị 0.

#### 6.2. Literal phức hợp (Composite Literals)

* Dùng literal phức hợp khi biết giá trị ban đầu của biến.
* Tránh dùng literal phức hợp cho giá trị rỗng, thay vào đó sử dụng giá trị 0.

#### 6.3. Gợi ý kích thước (Size Hints)

* Dùng `make` với gợi ý kích thước khi biết trước kích thước của slice hoặc map.
* Hầu hết code không cần gợi ý kích thước, cho phép runtime tự động tăng kích thước.

### 7. Hướng kênh (Channel Direction)

* Chỉ rõ hướng kênh (`<-chan` hoặc `chan<-`) để compiler phát hiện lỗi và thể hiện rõ mục đích sử dụng.

### 8. Danh sách tham số hàm (Function Argument Lists)

* Hạn chế số lượng tham số hàm để dễ đọc và dễ nhớ.
* Dùng `option struct` hoặc `variadic options` khi hàm có nhiều tham số.

### 9. Giao diện dòng lệnh (CLI)

* Sử dụng thư viện `cobra` hoặc `subcommands` để tạo CLI phức tạp.

### 10. Kiểm thử (Testing)

#### 10.1. Hàm hỗ trợ kiểm thử và khẳng định (Test Helpers and Assertion Helpers)

* Hàm hỗ trợ kiểm thử thực hiện thiết lập hoặc dọn dẹp, dùng `t.Helper` để báo cáo lỗi.
* Tránh dùng hàm hỗ trợ khẳng định, thay vào đó dùng `cmp` và `fmt`.

#### 10.2. Thông báo lỗi (Error Messages)

* Thông báo lỗi cần rõ ràng, đầy đủ, bao gồm:
    * Nguyên nhân lỗi
    * Đầu vào
    * Kết quả thực tế
    * Kết quả mong đợi
* Xác định hàm và đầu vào trong thông báo lỗi.
* Hiển thị giá trị thực tế (`got`) trước giá trị mong đợi (`want`).

#### 10.3. So sánh (Comparisons)

* So sánh cấu trúc đầy đủ bằng `cmp.Equal` hoặc `cmp.Diff`.
* Tránh so sánh kết quả phụ thuộc vào đầu ra không ổn định.
* Tiếp tục kiểm tra sau khi gặp lỗi, dùng `t.Error` thay vì `t.Fatal`.

#### 10.4. Kiểm thử con (Subtests)

* Dùng `t.Run` để tạo kiểm thử con.
* Đặt tên kiểm thử con rõ ràng, dễ đọc, dễ lọc.

#### 10.5. Kiểm thử bảng (Table-Driven Tests)

* Dùng kiểm thử bảng khi có nhiều trường hợp kiểm thử tương tự.
* Tách hàm kiểm thử khi logic kiểm tra khác nhau.
* Tách cấu hình kiểm thử khỏi logic kiểm tra.

### 11. Nối chuỗi (String Concatenation)

* Dùng `+` khi nối ít chuỗi.
* Dùng `fmt.Sprintf` khi định dạng chuỗi phức tạp.
* Dùng `strings.Builder` khi nối chuỗi từng phần.
* Dùng backticks (``) cho literal chuỗi nhiều dòng.

### 12. Trạng thái toàn cục (Global State)

* Tránh sử dụng trạng thái toàn cục vì khó kiểm tra, dễ gây lỗi và xung đột.
* Dùng giá trị instance để cho phép client tạo và sử dụng giá trị riêng biệt.

### 13. Thư viện thông dụng (Common Libraries)

* **`flag`**: Dùng snake case cho tên flag, camel case cho tên biến.
* **`glog`**: Thư viện ghi log của Google, dùng `log.Fatal` và `log.Exit` để dừng chương trình.
* **`context`**: Tham số đầu tiên của hàm, dùng `context.Background()` trong hàm entrypoint.

### 14. Kết luận

Tuân theo các quy chuẩn này giúp code Golang dễ đọc, dễ bảo trì và dễ sử dụng, đồng thời giúp dự án phát triển ổn định và hiệu quả hơn. 

**Tham khảo:**

* [Go Style Guide](https://google.github.io/styleguide/go/guide)
* [Go Style Best Practices](https://google.github.io/styleguide/go/best-practices)
* [Go Style Decisions](https://google.github.io/styleguide/go/decisions)
