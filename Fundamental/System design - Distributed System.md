1. **API GATEWAY**
	- Is a single point of entry to the clients of an application
	- some functions:
		- authentication and security policy enforcements (authentication là xác minh danh tính ví dụ như login. authorization là kiểm tra bạn có quyền hay không). authentication xảy ra sau authorization
		- load balancing
		- service discovery
		- monitoring, logging, analytics
		- caching
		- rate limiter
		- validate request
		- data loader
	- should be deployed to multiple regions to improve availability. For many cloud provider offerings, the API gateway is deployed across the world close to the clients.
2. **System Design Interview**
	- Hỏi user behaviors (User có thể làm gì)
	- Hỏi Total user
	- Hổi DAU (daily active users)
	- Hỏi về QPS = query per second ~= how many request per second (API call per second)
	- TÍnh toán các constraint để xác định các kiểu dữ liệu => lựa chọn db
3. **Rate limiter**:
	1. Một số thuật toán:
		- Token bucket:
			- Tạo một cái bucket bỏ vô n token. một request sẽ lấy mất m token. mỗi lần có request, check xem số token trong bucket >= m hay không. nếu không thì không thực hiện request. Cứ sau một time sẽ refill lại token
		- Leaky bucket:
			- Có thể xem là 1 cái queue. limit số lượng request và FIFO để serve. đảm bảo số lượng request tới server nhưng không control lượng tải tới server như token bucket.
		- Fixied window
			- Limit số lượng request trong một khoảng thời gian. hạn chế là brust số lượng request vượt limit.
		- Sliding Window log: 
			- ý tưởng giống fixed window. nhưng window này sẽ tính từ lúc request trở về trước 1 khoảng thời gian nên gọi là sliding. tuy nhiên hạn chế là cần lưu log và query nhiều lần mỗi lần có request để count. gây chậm, tốn tài nguyên
		- Sliding Window:
			- Cải tiến từ sliding window log. Nhận thấy hạn chế của sliding window, nên giải thuật này không lưu log, do không lưu log nên có một công thức tính toán số lượng request trong slide window. đánh đổi chính xác để cải thiện tốc độ và bộ nhớ.
4. CQRS:
	- Chia ra thành read side, write side - command, query
	- tầng dtos để aggregate data cho query
	- tầng entities để khi dữ liệu ở cmd.
	- handle nhận vào command và execute command
	- có thể dùng chung với event sourcing
	- **Independent scaling:** CQRS cho phép xử lý read và write có thể scale độc lập, hạn chế lock contentions.
	- **Optimized data schemas:** Read side có thể sử dụng schema tối ưu cho query, trong khi write side sử dụng schema tối ưu cho update.
	- **Separation of concerns:**  Phân tách read side và write side giúp cho model dễ dàng bảo trì và linh hoạt.
	- **Simpler queries:** Ứng dụng tránh việc thực hiện các query phức tap.
5. Consistent hashing:
	- là giải thuật để distributed data trong horizontal scaling
	- bài toán đặt ra là có thể có hot node, hoặc khi remove một node thì cần chia lại data nhiều.
	- giải thuật:
		- hash server name và data chung một hash function.
		- kết nối hash range thành 1 ring 
		- hash server name hoặc ip server, đưa các server vào ring.
		- hash object key, theo chiều kim đồng hồ, key nào gần server nào thì được đưa vào server đó.
		- khi add thêm một server hoặc remove một server, chỉ các key ở gần các server đó mới cần remapped 
		- sẽ gặp issue hot server. khi đó môi server sẽ có các virtual node trên ring để server đó có thể xuất hiện nhiều vị trí hơn trong ring
6. Caching
	- Chiến lược ghi cache:
		- Write through: ghi đồng thời DB và cache
		- Write-back: ghi cache xong mới async ghi db -> có thể dẫn đến inconsistency
		- Write-around: ghi db, lần sau đọc cache miss mới ghi vào cache
	- Chiến lược clear cache vì mem trên cache là limit
		- LRU: least recently used: xoá những cái đã cache quá lâu. VD như xoá email từ lâu rồi
		- LFU: least frequenly used: xoá những cái có tần suất sử dụng ít. VD: cache thông tin Sơn Tùng. xoá cache thông tin của tao
7. Redis:
	- Vì sao redis lại nhanh: 
		- vì nó là in memory database nên tốc độ đọc ghi nhanh hơn đĩa.
		- nó tận dụng low-level data structure để lưu vào ram mà k cần quan tâm đến việc ghi xuống đĩa
		- single thread nên k có lock hay switch context. handle multiple request bằng ghép kênh, multi plexing. thông qua select or poll system calla hoặc dùng epoll
	- Nhược điểm của redis:
		- vì là in mem nên dễ mất data.
		- k lưu trữ được nhiều như hdd hay ssd 
		- vì k phải là nơi để lưu data persistency nên dễ mất data. job với data quan trọng không nên dùng redis để implement queue
		- dùng redis pub/sub thay kafka khi:
			- lượng dữ liệu cần xử lý k lớn.
			- yêu cầu tốc độ
			- message được delivery đến comsumer ngay lập tức
			- gửi xong k cần lưu