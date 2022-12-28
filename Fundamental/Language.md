1. Golang
	1. **Architecture**
		- When golang values constructed on the heap:
			- when a value could possily be referenced after the function that constructed the value returns
			- when the compiler determines is too large to fit on the stack
			- when the compiler doesn't know the size of value at complie time 
		- Some more specific kind of values allocated in the heap:
			- values share with pointer
			- variables stored in interface variables
			- backing data for maps channels, slices, and strings
		- Struct padding in go
			- max field's byte of struct is size of large field in struct nhưng phải less than or equal machine word
			- can grouped more field to fit machine word.
	2. GC:
		- go GC là non-generational, non-compacting, non-moving GC 
		- nó dùng mark & sweep.
		- mark phase:
			+ setup: đầu tiên nó sẽ dừng hết các goroutine đang chạy (stop the world). bọn này được dừng ở function call. việc làm này để k có goroutine nào xin cấp thêm mem. nó đánh dấu 1 cái barrier để scan trong vùng barrier đó.
			+ marking: sau khi barrier turn on hoàn tất, các goroutine được chạy lại. nhưng thằng GC sẽ chiếm lấy processor (OS thread) để đi marking biến nào vẫn đang được dùng, có thằng trỏ tới nó. theo lý thuyết nó sẽ chiếm 25% số lượng OS thread. để làm việc này thì nó sẽ đi qua stack của các goroutine đang chạy, tìm đến mem nào mà các biến trong stack đang trỏ đến, đánh dấu nó. thông qua heap graph để mark thêm các mem được trỏ đến từ mem trong heap được đánh dấu khi nãy. tuy nhiên có 1 vấn đề nếu GC nhận thấy một mình nó k đủ resource để mark thì nó sẽ yêu cầu thêm các goroutine khác phụ nó gọi là Mark Assits
			+ termination: nó STW thêm lần nữa để tính toán lần chạy GC tiếp theo. nó remove cái barrier ở trên đi (turn off barrier). và nó sẽ có 1 cái collection các mem trên heap k được đánh dấu, cần dọn dẹp. rurn off hoàn tất, nó trả lại OS thread và tất cả OS thread được dùng để chạy goroutine như bth.
		- sweep: đi dọn dẹp các biến trong collection thu được ở bước trên. gọi là dọn dẹp, nhưng thật ra vùng mem không được đánh dấu sẽ được tái sử dụng cho những biến mới được tạo ra trong goroutine.
	3. Goroutine:
		- tạo ra ở user space nên có khả năng tạo rất nhiều so với os thread. 1 G khi mới khởi tạo có heap size chỉ 2KB. max 64 bit 1GB, 32 bit 250 MB
		- vì tạo ở user space nên developer có thể schedule task được.
		- giao tiếp thường thông qua channel
	4. Props
		- Static type. Complie code ra mã máy nhanh gọn. tận dụng lợi thế của cpu multiple core dễ dàng. giao tiếp giữa các thread đơn giản bằng channel
		- goroutine là lighweight thread nên rất gọn nhẹ, chi phí khởi tạo và run rất ít so với thead thông thường.
		- có GC 