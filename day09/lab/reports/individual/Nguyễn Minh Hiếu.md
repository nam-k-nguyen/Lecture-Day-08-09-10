# Báo Cáo Cá Nhân — Lab Day 09: Multi-Agent Orchestration

**Họ và tên:** Nguyễn Minh Hiếu  
**Vai trò trong nhóm:** Trace & Docs Owner  
**Ngày nộp:** 14/04/2026  
**Độ dài yêu cầu:** 500–800 từ

---

## 1. Tôi phụ trách phần nào? (100–150 từ)

Trong dự án lab Day 09 này, tôi chịu trách nhiệm chính về việc xây dựng hệ thống đánh giá hiệu năng (Evaluation) và hoàn thiện toàn bộ bộ hồ sơ kỹ thuật cho nhóm.

**Module/file tôi chịu trách nhiệm:**
- File chính: `eval_trace.py`
- Functions tôi implement: `analyze_traces()`, `compare_single_vs_multi()`, và các module tự động tổng hợp metrics từ trace JSON.
- Phụ trách toàn bộ thư mục `docs/` bao gồm: `system_architecture.md`, `routing_decisions.md`, `single_vs_multi_comparison.md`.

**Cách công việc của tôi kết nối với phần của thành viên khác:**

Công việc của tôi là tổng hợp code của cả nhóm. Tôi nhận file `graph.py` từ anh Nam (Supervisor Owner) và các worker từ Phúc, Tú Nam, Hữu Hưng để chạy đánh giá. Tôi sử dụng các file trace sinh ra từ code của anh Quân (MCP Owner) để viết tài liệu `routing_decisions.md`. Nếu tôi không hoàn thành việc phân tích trace, nhóm sẽ không có số liệu để so sàng với Day 08.

**Bằng chứng:**
Tôi đã trực tiếp viết code tính toán `routing_distribution` và `avg_confidence` trong `eval_trace.py` để chuyển đổi từ các file log thô sang báo cáo định lượng.

---

## 2. Tôi đã ra một quyết định kỹ thuật gì? (150–200 từ)

**Quyết định:** 
Tôi quyết định triển khai cơ chế **Automated Trace Analysis** trong file `eval_trace.py` thay vì kiểm tra thủ công từng file JSON trong thư mục `artifacts/traces/`.

**Lý do:**
Khi làm việc với Multi-Agent, số lượng trace sinh ra rất lớn. Nếu kiểm tra thủ công, tôi rất dễ bỏ sót các lỗi logic như Supervisor route nhầm hoặc Worker phản hồi chậm. Việc viết code để tự động bóc tách `route_reason` và `workers_called` giúp tôi có cái nhìn tổng quát về độ phủ của từng worker.

**Trade-off đã chấp nhận:**
Tôi đã phải dành thêm thời gian ở Sprint 4 để debug logic tính toán trong `eval_trace.py` thay vì viết tài liệu ngay lập tức. Điều này làm tăng áp lực thời gian cuối ngày nhưng bù lại, các con số trong `group_report.md` hoàn toàn chính xác và có bằng chứng cụ thể.

**Bằng chứng từ code:**
Đoạn code tôi dùng để bóc tách lịch sử thực thi của worker (ví dụ minh họa):
```python
def analyze_traces(trace_dir):
    # logic duyệt file và tổng hợp metrics
    for trace in all_traces:
        route = trace.get("supervisor_route", "unknown")
        stats["routing_distribution"][route] = stats["routing_distribution"].get(route, 0) + 1
```

---

## 3. Tôi đã sửa một lỗi gì? (150–200 từ)

**Lỗi:** `UnicodeEncodeError` khi in các icon đặc biệt trên terminal Windows.

**Symptom:**
Khi chạy `python graph.py` để sinh dữ liệu trace, script bị crash ngay lập tức tại các dòng print có chứa các icon đặc biệt như icon "▶" hoặc "⚠️". Điều này khiến nhóm không thể có file trace để làm bằng chứng cho tài liệu.

**Root cause:**
Terminal mặc định của Windows không hỗ trợ mã hóa UTF-8 cho các ký tự icon này, dẫn đến việc crash script do lỗi mã hóa (`cp1252`).

**Cách sửa:**
Tôi đã thay đổi các ký tự icon này sang ký tự ASCII tiêu chuẩn (ví dụ: `▶` sửa thành `>`). Việc này đảm bảo script có thể thực thi thành công trên tất cả máy tính của các thành viên trong nhóm mà không bị dừng đột ngột.

**Bằng chứng trước/sau:**
- Trước: Crash với lỗi `UnicodeEncodeError`.
- Sau: Script thực thi mượt mà, sinh ra 15 file trace JSON trong thư mục artifacts.

---

## 4. Tôi tự đánh giá đóng góp của mình (100–150 từ)

**Tôi làm tốt nhất ở điểm nào?**
Tôi làm tốt ở việc bao quát hệ thống và chuyển đổi các khái niệm kỹ thuật khô khan thành tài liệu kiến trúc dễ hiểu. Việc hoàn thiện bộ hồ sơ `docs/` giúp nhóm có cái nhìn rõ ràng về những gì mình đã xây dựng.

**Tôi làm chưa tốt hoặc còn yếu ở điểm nào?**
Tôi còn hơi chậm trong việc hiểu sâu về cấu trúc của `MCP Server`. Ở Sprint 3, tôi đã mất khá nhiều thời gian để tìm hiểu cách bóc tách `mcp_tools_used` từ `AgentState`.

**Nhóm phụ thuộc vào tôi ở đâu?**
Nhóm phụ thuộc vào tôi để hoàn thành `eval_trace.py` và chuẩn bị các file log. Nếu không có phần này, nhóm sẽ không thể nộp bài đúng hạn (trước 18:00) với đầy đủ dữ liệu grading.

---

## 5. Nếu có thêm 2 giờ, tôi sẽ làm gì? (50–100 từ)

Nếu có thêm thời gian, tôi sẽ thử triển khai **LLM-as-a-judge** trong `eval_trace.py` để chấm điểm Accuracy tự động thay vì chỉ nhìn vào Confidence. Hiện tại, một số câu hỏi phức tạp (multi-hop) vẫn cần kiểm tra tay kết quả để đảm bảo độ chính xác tuyệt đối.

---
