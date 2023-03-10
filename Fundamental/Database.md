1. ACID:
	- Atomicity - Tính nguyên tử: Một transaction chỉ thành công khi tất cả các hành động trong transaction đều thành công. khi có một hành động không thành công, những hành động trước đó sẽ phải roll back lại trạng thái ban đầu.
	- Consistency - tính nhất quán: dữ liệu luôn giữ được trạng thái hợp lệ dù ở bất kỳ thời điểm nào của transaction. ví dụ column type int thì không thể insert vào text hay varchar. constraint gender là 'male', 'female' thì không thể insert nam, nữ
	- Isolation - tính cô lập: nếu 2 transaction diễn ra trong cùng một thời điểm trên cùng một dữ liệu thì 2 transaction này sẽ không ảnh hưởng đến nhau. ví dụ khi transaction diễn ra nhưng không lock. transaction A trừ tiền trong tk thì transaction B đang read data sẽ vẫn thấy số tiền khi chưa bị trừ. nếu có lock, transaction A hoặc B sẽ thực hiện trước, sau đó mới đến transaction còn lại.
	- Durability - tính bền vững: data đã commit thành công thì dù có bất kì sự cố nào như shutdown server, thì khi khởi động lại server data vẫn như cũ, không bị mất data.
2. Index:
	1. Khái niệm
		- Hiểu đơn giản như muc lục của quyển sách. nhờ vào mục lục sẽ giúp chúng ta tìm kiếm các trang sách muốn đọc một cách nhanh chóng hơn.
		- Indexing là chuyển đổi một hoặc nhiều column sang table mới do db system sẽ quản lý table này.
		- thay vì scanning trên table chính, ta scanning trên table index, giảm lượng data read từ disk. table index được sắp xếp nên scan đơn giản hơn, thay ví scan toàn bộ, sử dụng binary tree để scan, phổ biến và thông dụng nhất chính là B-tree index. (Balanced). ngoài ra còn có hash index, bitmap index.
		- nhược điểm: 
			- tạo thêm bảng mới để lưu index. càng nhiều index càng nhiều bảng => insert cần re-balance => chậm write, dung lượng lưu trữ tăng.
			- với điều kiện where không hợp lý với index, tốn cost scan trên table index rồi lookup sang table chính
	2. Các loại index:
		- b-tree index:
			- sử dụng khi số lượng các giá trị không lặp lại của cột nhiều (high cardinality)
			- mỗi node có thể có nhiều hơn 2 node con.
			- tự cân bằng khi dữ liệu column index thay đổi
			- độ phức tạp bằng chiều cao cây. O(logn)
			- các node của cây có link trực tiếp với nhau nên phù hợp với order by
			- vì sao không đánh trên những cột low cardinality: vì vẫn phải seq scan trên nhiều value mà vừa tốn thêm disk space để lưu cây, vừa tốn thời gian lookup sang table chính
			- phù hợp với:
				- **match full value**: tìm kiếm với giá trị chính xác. Sử dụng với các điều kiện so sánh =
				- **match a column prefix**: composite index với nhiều column. index scan chỉ được sử dụng khi điều kiện column đầu tiền là match value, ngược lại không áp dụng được, vì tất cả dều được sắp xếp theo column đầu tiên, sau đó đến column thứ hai.
				- match a range of full values: tìm kiếm với rất nhiều giá trị chính xác. cụ thể là điều kiện IN hoặc các điều kiện so sánh (>, <, >=, <=)
			- không phù hợp với: tìm kiếm text điều kiện LIKE.
			- với composite index, where column cần match full value hoặc match mostleft column
		- hash index:
			- phù hợp so sánh =. không phù hợp so sánh khoảng hoặc lớn bé
			- không phù hợp với composite index. vì 2 bảng có size m, n thì hash của m * n khá to, dễ vượt qua max size của int32, int64.
		- bitmap index:
			-  Phù hợp với các column **low cardinality**.
			-  Lưu bit cho mỗi giá trị nên giảm dung lượng lưu trữ cần dùng.
			-  Chỉ hiệu quả với tìm kiếm **full match value**.
			-  Kết hợp với nhiều index khác để tăng tốc độ với OR, AND.
	1. Clustered index - Non-clustered index:
		- Clustered index:
			- Bản chất là build b-tree trên chính physical data stored. Mỗi table chỉ nên có 1 clustered index, vì node leaf lưu full thông tin nên tốn khá nhiều disk space, và vì nó build cây trên chính physical data storged nên không thể nào order theo nhiều tiêu chí khác nhau được ở những index khác nhau được. Tuy nhiên, clustered index có thể là composite clustered index.
			- Đảm bảo data ordered
			- Thường được đánh trên primary key. nếu không phải primary key thì phải trên column unique, not null.
			- Nên đánh trên column auto increment vì khi đó, nó chỉ cần append vào cuối.
			- Mỗi node leaf lưu trữ full thông tin record. vì vậy, khi tìm kiếm trên clustered index, không cần trỏ tới table chính như btree index thông thường. Nên nó cải thiện đáng kể performance I/O bound workloads.
			- không nên update giá trị trên clustered index, vì phải build lại cây, lại còn phải cập nhật data trên physical stored, dẫn đến chia nhiều page hơn => full scan tốn thêm cost để load data từ disk lên
			- Implement b+tree, khi node trung gian chỉ chứa giá trị index.
		- Non-clustered index:
			- Logical data stored
			- Index ở những cột còn lại không phải cột được chọn để đánh clustered index
			- Node leaf lưu trữ thông tin giá trị clustered index của record đó. Từ đó khi tìm kiếm, nó quét trên index của non-clustered index, sau đó sẽ lấy giá trị ở node leaf để search trên b-tree của clustered index.
			- implement b-tree
	2. Merge index:
		- Khi where trên các column được đánh index riêng lẻ, không phải composite index thì sẽ diễn ra index_merge
		- Using intersect(...): Merge với điều kiện AND. Scan trên các col index điều kiện, sau đó merge lại rồi scan trên bảng này. nếu column điều kiện là PK thì nó dùng để filter sau khi đã kiểm tra các điều kiện khác
		- Using uinon(...): Merge với điều kiện OR.
		- Using sort_union(...): Merge với điều kiện OR. khác với union, sort_union fetch row ids for all row -> sort -> return
1. JOIN:
		- inner join: lấy các record mà điều kiện thoả ở cả 2 bảng
		- left join: lấy full record table bên trái kết hợp với các phần tử phù hợp ở table join
		- right join: lấy các record phù hợp ở table from và full record ở table join
		- cross join, full join: lấy tất cả record ở cả 2 bảng 
		- cơ chế join:
			- nested loop join:
				- sử dụng 2 vòng lặp ở 2 table drive table và join table
				- dễ hiểu, dễ implement
				- phù hợp với table có số lượng record ít.
			- hash join:
				- phase 1 - hash phase: build hash table với table có số lượng record nhỏ hơn. hash join FK ra giá trị lưu vào hash table. mỗi record trong hash table sẽ mapping với các record trong table chính
				- phase 2 - probe phase: duyệt qua các record của table chính, tính toán hash value của FK. tìm hash value vừa tính trong hash table lưu ở phase 1 => lookup O(1)
			- merge join:
				- sort 2 table theo join FK key
				- tận dụng lợi thế của table sau khi sort để join.
2. Các loại lock:
	- Read lock: thằng khác có thể yêu cầu lock để đọc. nhưng không thằng nào yêu cầu write được
	- Write lock: thằng nào mà chiếm cái lock này rồi thì không thằng nào read hoặc write được 
	- Pessimisstic lock: 
		- transaction T(1) lock thành công thì các T(x) phải chờ T(1) xong thì mới acquire lock được
		- cons: dễ gây tắt nghẽn vì T(x) phải chờ.
		- props: không gây conflict data.
		- nên dùng với những transaction có tỉ lệ conflict cao để đảm bảo consistence cũng như giảm thiểu retry. 
	- Optimistic lock:
		- Transaction T(x) chỉ acquire lock khi thực hiện commit. thằng nào update mà đúng với version hiện có thì mới update thành công. không là phải retry tới khi nào được thì thôi.
		- Bản chất của lock khi commit là kiểm tra version của record mà thằng T(x) lúc update có giống với version hiện tại hay không.
		- Props: Giảm cost cho việc blocking xo với pessimistic lock
		- Cons:
			- retry nhiều lần
			- lỡ update -> trigger 1 hành động đéo nào đấy -> commit lỗi -> rollback. mất công vl
			- có thể dẫn đến inconsistence data. haft-written
		- nên dùng với transaction tỉ lệ conflict thấp để giảm retry.
		- quản lý version có thể dùng:
			- Implicit hidden column: tạo một column mới do database quản lý để lưu version.
			- time based update detection: dựa vào cột updated_at
3. MVCC:
	- ý tưởng: có 2 record A, B y hệt nhau. read ở A, write ở B. không cần lock. khi có write thành công thì pointer trỏ tới record mới. record cũ sẽ được dọn dẹp sau.
	- Mục đích chính của **multi-version concurrency control** để quản lý các truy cập read/write đồng thời đến database mà không cần **locking**. Reading không block writing và ngược lại
4. Transaction isolation level: 
	- Read uncommited: 
		- T(2) đọc được data ở T(1) mặc dù chưa commit
		- Dễ bị dirty read: đọc xong, làm gì đó rồi, bên kia rollback -> thế là kết quả bị sai
		- Phù hợp cho việc thống kê vì performance tốt do không có lock
	- Read commited :
		- T(2) chỉ đọc được data ở T(1) khi T(1) đã commit
		- Phantom row / read: lúc T(1) chưa commit, T(2) đang đọc ra 5 record, T(1) commit, T(2) đọc ra 6 record. Dữ liệu không được nhất quán
	- Repeatable read:
		- Kiểu tạo 1 snapshot. T(2) read trên snapshot này, T(1) update không ảnh hưởng đến snapshot
	- Seriablizable:
		- Level này sẽ thực hiện tuần tự nối tiếp cho tất cả các transaction, gần như chẳng có gì có thể diễn ra đồng thời.
5. Partitioning
	- Horizontal partitioning: 
		- Chúng ta chia table lớn thành nhiều table nhỏ hơn, các table nhỏ hơn gọi là **partition table**, kế thừa toàn bộ cấu trúc của parent table, từ column cho đến kiểu dữ liệu. Việc chia nhỏ này được gọi là
		- Giới hạn vùng dữ liệu phải scan trên table trong một vài trường hợp. Nếu ta cần tìm một học sinh tên John Doe không phân biệt giới tính thì việc partition như ví dụ trên không đem lại hiểu quả.
		- **Partition table** cũng giống như một table thường nên ta có thể thực hiện index cho nó. Dẫn đến việc tốn ít cost hơn để maintain index table, do số lượng record ít hơn.
		- Ngoài ra, việc xóa các dữ liệu trên **partition table** sẽ nhanh hơn và không ảnh hưởng đến các **partition** khác.
		- Áp dụng với các table rất lớn. Thường là quá size của memory.
		- Việc partition trên điều kiện nào phải dựa vào tính chất và tần suất của các query.
	- Vertical partitioning:
		- nên được chú ý ngay từ khi design database vì việc này ảnh hưởng trực tiếp đến cách query và vận hành hệ thống do phải chia thành các table thực
		- Về cơ bản, các records được lưu thành một khối dữ liệu có độ lớn gần tương tự như nhau được gọi là block. Do đó, nếu một table chứa số lượng column ít đồng nghĩa với việc tăng đương số lượng records lưu trữ trên một block. Như vậy nếu query các column trong cùng block, các xử lý tính toán I/O sẽ giảm đi phần nào dẫn tới việc tăng performance
	- Các loại partition
		- By list: việc phân chia ra các partition dựa trên key được định nghĩa dưới dạng list of value. Ví dụ với table **ENGINEER**, các engineer có title **Backend Engineer**, **Frontend Engineer**, **Fullstack Engineer** nhóm vào thành một partition; **BA** và **QA** một partition; còn lại là **default partition**
		- By hash: 
			- Thực hiện hash partition key ra hash value.
			-  Modulus để tìm partition cho record. Ví dụ record có partition key hash value = 5, tổng số lượng partition là 3 (0, 1, 2), lấy 5 % 3 = 2. Vậy record đó nằm ở partition thứ ba.
		- By range: Khi thực hiện **range partition** ta quan tâm đến giá trị min và max của mỗi **partition** để thực hiện việc phân chia. Ví dụ table **ENGINEER** có thông tin về ngày bắt đầu làm việc (start_date), ta có thể dựa trên column này để phân chia **partition** theo từng quý
	- Lợi ích:
		- Biến một logical table thành nhiều physical table, giảm không gian tìm kiếm, tăng performance cho query trong trường hợp tận dụng được điều kiện WHERE với **partition key**.
		-  Xóa bỏ các data cũ một cách dễ dàng, nhanh chóng so với cách truyền thống. Ví dụ cần xóa tất cả các record có giới tính nam, nếu chia partition theo giới tính từ đầu. Chỉ cần TRUNCATE/DROP **partition** đó là ok. Không cần seq scan để DELETE record với WHERE condition.
	- Bất lợi:
		- Unique constraint, PK constraint không thể thực hiện trên table chính mà phải thực hiện ở **partition table**.
		- Các **partition table** không thể có column nào khác mà không khai báo ở parent table. Nói cách khác, nó kế thừa toàn bộ column và data type của parent table.
		-  Một vài vấn đề liên quan đến trigger. Ví dụ BEFORE ROW trigger ON INSERT. Nó thực hiện trigger function/procedure... trước khi row được insert vào table.
6. Materialized view:
	- Tạo một view ảo aggregate toàn bộ data. move cache từ application xuống db