---
layout: post
title: Mối nguy hiểm tiềm tàng của các thuật toán Dynamic routing trong việc xây dựng Forwarding Table - Lý thuyết Đồ thị.
author: cuongbui
tags: [algorithm, bugs, dynamic, forwarding, greedy, OSI, routing]
categories: [security, algorithm]
---

Hôm nay tôi mới tìm ra nguyên nhân dẫn đến 1 con bug mà tôi đau đầu trong suốt mấy ngày. Con bug này xuất hiện trong chương trình mô phỏng sự hoạt động của các thuật toán định tuyến trong Mạng liên kết nói riêng và trong Lý thuyết đồ thị nói chung.
<!--more-->

Thật ra con bug này ko chỉ ảnh hưởng đến 1 thuật toán định tuyến, mà nó ảnh hưởng đến 1 họ các thuật toán định tuyến Grid mà tôi đã nghiên cứu cùng Sư Phụ nhiều năm. Và tôi cũng mới chỉ tìm ra được nguyên nhân dẫn đến con bug này và fix tạm thời một cách qua loa để chương trình chạy được; còn nếu muốn fix triệt để thì chắc sẽ cần mò mẫm sâu thêm.

### Routing vs Forwarding?

Trong lý thuyết Đồ thị, các Thuật toán định tuyến (nổi tiếng nhất chắc là Dijkstra shortest path) có tác dụng tìm đường đi từ một đỉnh nguồn đến 1 đỉnh đích. Mỗi thuật toán định tuyến khác nhau thì sẽ có cách thức định tuyến (routing rule) khác nhau (tùy từng trường hợp topo), dẫn đến đoạn đường định tuyến tìm được giữa nguồn và đích cũng sẽ có thể ko giống nhau.

Đó là tư tưởng của việc Routing! Còn Forwarding thì khác, nó là thủ tục hậu kỳ sau khi đã routing xong. Về bản chất, sau khi hoàn thành routing, đoạn đường định tuyến đã được tìm ra chính xác theo đúng routing rule của thuật toán định tuyến; tuy nhiên việc truyền tin trên đoạn đường đó lại là một vấn đề khác. Trong tầng Physical của mô hình OSI, các thiết bị chỉ biết đường đi chính xác (nối bằng dây cáp chẳng hạn) đi đến các thiết bị hàng xóm - được liên kết trực tiếp với nó, chứ còn ko biết đường đi chính xác đến hàng xóm của hàng xóm. Vậy không thể implement tư tưởng của routing vào ngữ cảnh này được. Tuy nhiên, dữ liệu lại bắt buộc phải đi qua tầng này, do vậy, để tinh gọn tư tưởng của routing nhằm áp dụng vào tầng này, chúng ta sinh ra khái niệm forwarding. Tư tưởng đơn giản hơn, chỉ là chuyển tiếp gói tin nhận được cho 1 thằng hàng xóm nào đó.

Với tư tưởng routing, nếu bạn muốn chuyển 1 gói hàng cho thằng X nào đó, thì bạn sẽ routing để biết rõ rằng đoạn đường định tuyến chính xác để đi đến X là: source - A - B - X. Tuy nhiên, bạn chỉ có thể vất gói hàng đó cho A, chứ ko nên vất nguyên cả đoạn đường đó cho A. Lý do thì có rất nhiều, đơn cử nhất là do phải tối ưu bộ nhớ (1 con chip đơn vị FPGA rất nhỏ, ko thể chứa hết all - pairs routing path được). Còn với forwarding thì đơn giản hơn nhiều. Mỗi đỉnh chỉ cần nhớ duy nhất 1 đỉnh tiếp theo để chuyển tiếp gói tin tới được đích là xong. Trong ví dụ trên, A chỉ cần nhớ rằng để chuyển gói hàng đến X thì nó phải đưa cho thằng B ngay gần nó nhất; chứ ko phải nhớ cả quãng đường "source - A - B - X".

Trong thực tế, Routing table được implement ở tầng Data Link, còn Forwarding table thì ở tầng Physical.

### Dynamic Routing là gì?

Thuật toán Dijkstra chắc là thuật toán định tuyến nổi tiếng nhất mà ai cũng biết đầu tiên rồi, và nó là 1 Greedy Routing Algorithm. Tham lam thể hiện ở việc nó sẽ duyệt toàn bộ đồ thị để tìm đường đi ngắn nhất; và đúng là đường đi mà nó tìm ra đúng là đường đi ngắn nhất thật!

Vậy Dynamic Routing Algorithm có j khác? Và tại sao người ta lại nghiên cứu cái này? Câu trả lời cũng vẫn là rất nhiều lý do, và đơn cử nhất là do phải tối ưu bộ nhớ! Duyệt toàn bộ đồ thị, quá tốn thời gian và bộ nhớ để chạy cũng như lưu lại kết quả hỗ trợ định tuyến. Vậy nên Dynamic routing ra đời, 1 phần sẽ giúp giảm tải được nhược điểm trên. 

Đoạn đường định tuyến được tìm ra bởi thuật toán Dynamic routing đương nhiên sẽ không là shortest như Dijkstra, nhưng nó sẽ có lợi thế hơn ít nhất là về thời gian chạy (nhiều hơn thì hơi sâu). Và điểm khác biệt còn nằm ở chỗ khốn kiếp sau: đoạn đường định tuyến tìm được bởi Greedy thì luôn cố định, nhưng còn đoạn đường định tuyến tìm được bởi Dynamic thì nó cũng cứ dynamic như cái tư tưởng của nó vậy! Tức là mỗi lần chạy thuật toán, Greedy sẽ cho ra cùng 1 kết quả với tất cả các lần chạy; còn Dynamic thì ko! Sự khác nhau giữa các kết quả của thuật toán Dynamic tùy thuộc vào routing rule của chúng. 

### Lý do dẫn đến bug và cách giải quyết tạm thời.

Quay trở lại con bug, thuật toán tôi code là 1 loại Dynamic routing. Routing rule của nó có sử dụng 1 khái niệm là cầu - mà các bạn có thể hiểu là 1 chuỗi liên tiếp các cạnh trong đồ thị. Và mỗi đỉnh sẽ nhận được thông tin về rất nhiều cầu mà nó có thể sử dụng để định tuyến. Đương nhiên nó sẽ chọn chỉ 1 cầu để định tuyến đến một vị trí mà thôi; ko có chuyện dùng nhiều hơn 1 cầu để tìm đường đi đến cùng 1 đích. 

Và cái óc chó ở đây là mỗi đỉnh khác nhau sẽ chọn ra được cầu khác nhau để đi tới đích (cho dù cùng cơ chế chọn cầu). Điều này lại dẫn đến 1 cái óc chó tiếp theo, đó là nếu chỉ routing thì mọi thứ vẫn rất bình thường, nhưng nếu dùng đoạn đường định tuyến đó để nhét vào dùng cho forwarding thì sẽ xảy ra hiện tượng infinity loop như sau.

A muốn đến X, A routing và sử dụng cầu B - C - W để rồi từ W - X. Từ đó A biết routing path là A - B - C - W - X, và A phải forwarding gói tin cho B để B chuyển tiếp gói tin đó đến X. Coi như là B cũng dùng cầu B - C - W để định tuyến đi, nhưng khi gói tin đến C, C lại sử dụng một cầu khác là D - F - B - Z để rồi từ Z đến X, thành ra C lại chuyển tiếp gói tin lần lượt rồi lại quay về B. 

Vậy đó, nếu chỉ đơn thuần là routing luôn 1 thể thì sẽ ko sao, thuật toán vẫn đúng, code cũng vẫn đúng, chạy vẫn đúng. Tuy nhiên, nếu muốn forwarding thì infinity loop ocho kia sẽ xuất hiện. Và vì tôi code theo tư tưởng clean - tách riêng các pha chương trình ra với nhau - nên tôi mới dùng forwarding, chứ ko nếu là 1 khối monolithic thì sẽ chẳng có pha nào là forwarding cả.

Và cách khắc phục tạm thời của tôi rất tạm bợ. Lý do xảy ra việc các đỉnh tìm ra được các cầu khác nhau để dùng, do đó tôi đã ép để cho cách thức tìm cầu của tất cả các đỉnh sẽ cho ra cùng 1 cầu duy nhất. Và cầu duy nhất đó ko j khác, là cầu ngắn nhất. Ban đầu, tôi lựa chọn cầu tốt nhất, nhưng chính vì mỗi cầu khác nhau sẽ có cost khác nhau ở từng đỉnh, nên việc tìm ra cầu ở các đỉnh khác nhau chưa chắc đã giống nhau. Cầu này có thể có cost nhỏ nhất nếu dùng cho đỉnh này, nhưng có thể lại không phải có cost nhỏ nhất nếu dùng cho đỉnh khác.


Vậy đó, khá là cân bằng.

![My Thanos](/assets/img/posts/cuongbui/thanos.png){: .center_image }

