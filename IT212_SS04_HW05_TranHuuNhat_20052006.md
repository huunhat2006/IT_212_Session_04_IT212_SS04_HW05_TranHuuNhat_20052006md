BÀI 5 - Thiết kế Quy trình & Prompt Kiểm tra Giao dịch Đáng ngờ (Fraud Detector)
1. Ý đồ thiết kế quy trình 2 bước

Quy trình được chia thành 2 bước nhằm mô phỏng quy trình làm việc thực tế của kỹ sư phần mềm:

Bước 1 - Code Generation

Yêu cầu AI sinh mã nguồn Java đúng nghiệp vụ trước.

Mục tiêu:

Đúng chức năng.
Xử lý đủ các quy tắc phát hiện gian lận.
Kiểm tra dữ liệu đầu vào.
Dễ đọc, dễ bảo trì.

Ở bước này AI thường ưu tiên tính đúng đắn nên có thể sinh thuật toán dùng vòng lặp lồng nhau O(N²).

Bước 2 - Performance Audit

Yêu cầu một AI khác (hoặc cùng AI với prompt mới) đóng vai trò chuyên gia tối ưu thuật toán.

Mục tiêu:

Phân tích độ phức tạp.
Tìm điểm nghẽn hiệu năng.
Đề xuất thuật toán tốt hơn.
Tối ưu từ O(N²) xuống O(NlogN).

Đây chính là mô hình:

Generate → Audit → Optimize

được sử dụng phổ biến trong các dự án FinTech thực tế.

BƯỚC 1 - CODE GENERATION PROMPT

Bạn là một Java Senior Developer đang phát triển hệ thống chống gian lận giao dịch (Fraud Transaction Detector) cho ứng dụng FinTech SmartCheck.

Nhiệm vụ:

Hãy xây dựng lớp FraudDetector với phương thức:

List<Transaction> detectFraud(List<Transaction> transactions)

Mô hình dữ liệu:

class Transaction {
    private String transactionId;
    private String cardId;
    private double amount;
    private String location;
    private LocalDateTime transactionTime;
}

Quy tắc nghiệp vụ:

Nếu giá trị giao dịch > 100,000,000 VND thì giao dịch đó bị đánh dấu đáng ngờ.
Nếu xuất hiện hai giao dịch liên tiếp của cùng một cardId:
Khác location
Khoảng cách thời gian dưới 10 phút

thì cả hai giao dịch đều bị đánh dấu đáng ngờ.

Yêu cầu kỹ thuật:

Sử dụng Java 8.
Sử dụng LocalDateTime và Duration.
Kiểm tra dữ liệu đầu vào.
Trả về danh sách các giao dịch đáng ngờ.
Không phát sinh NullPointerException.

Xử lý dữ liệu biên:

transactions = null → trả về danh sách rỗng.
transactions rỗng → trả về danh sách rỗng.
location null hoặc rỗng → bỏ qua quy tắc kiểm tra vị trí.
cardId null → bỏ qua giao dịch đó.

Định dạng đầu ra:

Giải thích ngắn về thuật toán.
Phân tích độ phức tạp thời gian.
Sinh đầy đủ mã nguồn Java hoàn chỉnh.
BƯỚC 2 - PERFORMANCE AUDIT PROMPT

Bạn là một Principal Software Engineer chuyên tối ưu hiệu năng hệ thống FinTech.

Dưới đây là mã nguồn FraudDetector đã được sinh ra ở bước trước.

Hãy thực hiện Code Review và Performance Audit.

Yêu cầu:

Phân tích độ phức tạp thời gian hiện tại.
Xác định các đoạn sử dụng vòng lặp lồng nhau O(N²).
Đánh giá tác động khi dữ liệu đạt 10,000 giao dịch.
Giải thích vì sao thuật toán hiện tại có thể trở thành bottleneck.
Đề xuất chiến lược tối ưu.

Ràng buộc:

Vẫn giữ nguyên nghiệp vụ.
Không làm thay đổi kết quả đầu ra.
Tối ưu bằng cách:
Gom nhóm giao dịch theo cardId.
Sắp xếp theo transactionTime.
Chỉ so sánh các giao dịch lân cận trong cùng nhóm.

Mục tiêu:

Đưa độ phức tạp về O(NlogN).
Giảm số lượng phép so sánh không cần thiết.

Định dạng đầu ra:

Phân tích điểm nghẽn.
So sánh Big-O trước và sau tối ưu.
Mã nguồn Java tối ưu hoàn chỉnh.
Giải thích lý do tối ưu hoạt động chính xác.
MINH CHỨNG THỰC TẾ (LOG CHAT MẪU)
Lượt Chat 1 - Sinh mã nguồn
User
(Prompt ở Bước 1)
AI
import java.time.Duration;
import java.time.LocalDateTime;
import java.util.*;

public class FraudDetector {

    public List<Transaction> detectFraud(List<Transaction> transactions) {

        List<Transaction> suspicious = new ArrayList<>();

        if (transactions == null || transactions.isEmpty()) {
            return suspicious;
        }

        for (Transaction tx : transactions) {

            if (tx == null || tx.getCardId() == null) {
                continue;
            }

            if (tx.getAmount() > 100_000_000) {
                suspicious.add(tx);
            }
        }

        for (int i = 0; i < transactions.size(); i++) {

            Transaction t1 = transactions.get(i);

            for (int j = i + 1; j < transactions.size(); j++) {

                Transaction t2 = transactions.get(j);

                if (!Objects.equals(t1.getCardId(), t2.getCardId())) {
                    continue;
                }

                if (t1.getLocation() == null ||
                    t2.getLocation() == null) {
                    continue;
                }

                long minutes =
                    Math.abs(
                        Duration.between(
                            t1.getTransactionTime(),
                            t2.getTransactionTime())
                            .toMinutes()
                    );

                if (minutes < 10 &&
                    !t1.getLocation().equals(t2.getLocation())) {

                    suspicious.add(t1);
                    suspicious.add(t2);
                }
            }
        }

        return suspicious;
    }
}
AI phân tích
Độ phức tạp thời gian: O(N²)
Do sử dụng hai vòng lặp lồng nhau để so sánh các giao dịch.
Lượt Chat 2 - Kiểm toán hiệu năng
User
(Prompt ở Bước 2)
AI
Phân tích
Thuật toán hiện tại có độ phức tạp O(N²).

Khi có 10,000 giao dịch:

Số phép so sánh xấp xỉ:

10,000 × 10,000
= 100,000,000

Đây là bottleneck lớn của hệ thống.
Đề xuất
1. Gom nhóm theo cardId bằng HashMap.

2. Sắp xếp từng nhóm theo transactionTime.

3. Chỉ kiểm tra hai giao dịch liền kề.

Độ phức tạp:

Grouping:
O(N)

Sorting:
O(NlogN)

Scanning:
O(N)

Tổng:
O(NlogN)
Mã nguồn tối ưu cuối cùng
import java.time.Duration;
import java.util.*;
import java.util.stream.Collectors;

public class FraudDetector {

    public List<Transaction> detectFraud(List<Transaction> transactions) {

        Set<Transaction> suspicious = new HashSet<>();

        if (transactions == null || transactions.isEmpty()) {
            return new ArrayList<>();
        }

        for (Transaction tx : transactions) {

            if (tx == null || tx.getCardId() == null) {
                continue;
            }

            if (tx.getAmount() > 100_000_000) {
                suspicious.add(tx);
            }
        }

        Map<String, List<Transaction>> grouped =
                transactions.stream()
                        .filter(Objects::nonNull)
                        .filter(t -> t.getCardId() != null)
                        .collect(Collectors.groupingBy(
                                Transaction::getCardId));

        for (List<Transaction> group : grouped.values()) {

            group.sort(
                    Comparator.comparing(
                            Transaction::getTransactionTime));

            for (int i = 0; i < group.size() - 1; i++) {

                Transaction current = group.get(i);
                Transaction next = group.get(i + 1);

                if (current.getLocation() == null ||
                    next.getLocation() == null) {
                    continue;
                }

                long minutes =
                        Duration.between(
                                current.getTransactionTime(),
                                next.getTransactionTime())
                                .toMinutes();

                if (minutes < 10 &&
                    !current.getLocation()
                            .equals(next.getLocation())) {

                    suspicious.add(current);
                    suspicious.add(next);
                }
            }
        }

        return new ArrayList<>(suspicious);
    }
}
Kết quả kiểm toán
Tiêu chí	Phiên bản đầu	Phiên bản tối ưu
Thuật toán	So sánh mọi cặp	Group + Sort
Độ phức tạp	O(N²)	O(NlogN)
10,000 giao dịch	~100 triệu phép so sánh	Thấp hơn rất nhiều
Khả năng mở rộng	Trung bình	Tốt

Kết luận: Quy trình 2 bước giúp AI không chỉ sinh mã nguồn đúng nghiệp vụ mà còn tự kiểm toán và tối ưu hiệu năng, đưa thuật toán từ O(N²) xuống O(NlogN), phù hợp với yêu cầu hệ thống FinTech có dữ liệu lớn.