1. Array: 
	- danh sách các phần tử có cùng cấu trúc, kiểu dữ liệu xếp liền mạch nhau.
	- mem size is fixed, cannot change during the run time. be assigned at compile time
	- sử dụng bộ nhớ không hiệu quả
	- độ phức tạp các thao tác cơ bản: 
		- insert: 
			- cuối mảng: O(1)
			- giữa mảng: O(n)
			- trung bình là O(n) vì cần shift các phần tử về đúng vị trí. chưa kể trường hợp mảng hết capacity, cần relocate mảng ở vị trí khác trong bộ nhớ, copy mảng cũ qua mảng mới.
		- access: O(1)
		- delete:
			- cuối mảng: O(1)
			- giữa mảng O(n)
		- search: O(n)
2. Linked list: 
	- Các phần tử gọi là các node, tốn vùng nhớ hơn phần tử mảng vì mỗi phần tử cần lưu thêm thông tin con trỏ trỏ đến các phần tử đứng trước hoặc đứng sau tuỳ vào tính chất linked list
	- tối ưu bộ nhớ hơn array, tránh tình trạng phân mảnh vùng nhớ.
	- size không fixed, thay đổi trong run time. thích hợp cho việc lưu trữ chưa biết trước số lượng phần tử
	- độ phức tạp các thao tác cơ bản: 
		- insert: 
			- cuối, đầu: O(1). còn tuỳ vào cách implement 
			- giữa: O(n). nhưng đỡ tốn time hơn vì không cần shift phần tử như array, chỉ cần thay đổi liên kết các con trỏ ở 1 số node
		- access: O(n)
		- delete
			- cuối, đầu: O(1)
			- giữa: O(n), nhưng đỡ tốn time hơn vì không cần shift phần tử như array, chỉ cần thay đổi liên kết các con trỏ ở 1 số node
		- search: O(n)
3. Hashmap:
	- Là một CTDL, mỗi phần tử là một cặp key - value.
	- hash là quá trình khởi tạo một giá trị khoá (32 or 64 bit) từ 1 phần dữ liệu
	- dựa vào giá trị hash, dữ liệu được đưa vào các bucket.
	- gọi n là số phần tử cần lưu, k là số bucket. giá trị n/k gọi là load factor. khi load factor <= 1 ~ 1, giá trị hàm hash phân bố đều, độ phức tạp của các thao tác trên hash table là O(1)
	- khi đụng độ có 2 chiến lược xử lý:
		- separate chaining: mỗi bucket là một linked list hoặc array.
		- open addressing: khi đụng độ, lưu vào vị trí trống tiếp theo của bảng băm. khi đó, khi tìm kiếm phần tử không chứa trong bảng băm, ta phải gặp được bucket trống từ bucket của giá trị hash, khi đó ta mới chắc chắn giá trị hash không có trong hash table. load factor n/k cần nhỏ hơn 1 mới thực hiện được open addressing.
				- khi hash ra được 1 index, access vào giá trị đó như access giá trị của mảng nên sẽ nhanh O(1). nhưng khi đụng độ sẽ > O(1), trường hợp xấu nhất là O(n)
4. Heap:
	- Min heap: phần từ thứ i phải nhỏ hơn phần tử thứ 2*i và 2*i+1 với i >= 1. Min là arr[1]
	- Max heap: phần từ thứ i phải lớn hơn phần tử thứ 2*i và 2*i+1 với i >= 1. Max là arr[1]