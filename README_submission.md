# Lab 17 – Submission notes

## Phân tích benchmark

Không có layer có hit rate thấp nhất: tất cả đều 100% — short-term E01/E10 (2/2), long-term E02/E03/E08/E09 (4/4), episodic E04/E05 (2/2), semantic E06/E11 (2/2), mixed E07 (1/1). Query nhiều token nhất là E03, “Minh còn open loop hay deadline nào chưa hoàn thành?”, với 753 tokens.

E07 kết hợp **long-term** và **semantic**: evidence bắt buộc là preference **Python** của Minh và **Idempotency-Key** của payment-retry policy. Benchmark giảm token trung bình 14,19% so với full source context. No-memory giảm 81,8% nhưng hit rate chỉ 18,2% (2/11), vì gần như không retrieve context; giảm token không đồng nghĩa evidence đúng.

## Reflection

Layer quan trọng nhất là **long-term**, bao phủ E02, E03, E08, E09: preference qua session, open loop, conflict/recency và user isolation. Zep Context Block quản lý graph, thread, facts/episodes, provenance và cross-session retrieval theo `user_id`, nên ít phải tự ghép dữ liệu; đổi lại phụ thuộc cloud và latency trung bình 524,6 ms. Redis + Qdrant kiểm soát cache/TTL, vector store và hạ tầng cục bộ, nhưng phải tự xây ingestion, đồng bộ, ranking, isolation và xóa liên store.

Guardrail chống memory poisoning: chỉ durable-ingest khi opt-in, minimize PII, lưu provenance/timestamp/user scope, và không diễn giải retrieved text thành instruction hoặc quyền mới. Heartbeat chỉ deduplicate, đánh dấu stale và tạo recap an toàn; không được tự ghi policy/quyền vào memory. Xóa dữ liệu phải xóa Zep + Redis theo user và verify mọi store.

E08 thể hiện scope-specific conflict: Python còn đúng cho demo cá nhân ORCHID-27, nhưng thông tin mới hơn quy định BLUEBIRD-42 phải dùng TypeScript/NestJS; không được áp preference chung sai project. E10 cho thấy compaction giữ durable constraint `REVIEW-DEADLINE-1600` trong summary/notes dù raw turn cũ bị evict, nên deadline vẫn retrieve được.
