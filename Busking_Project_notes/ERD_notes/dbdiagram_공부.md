# dbdiagram 공부
- 복합 UNIQUE 제약조건은 dbdiagram에서는 지원 안 됨
- 다형성 관계 (Polymorphic Association)는 하나의 FK로 두 테이블을 구분할 수 없음 → `post_type + post_id` 조합으로 구분
- `CHECK (rating BETWEEN 1 AND 5)` → dbdiagram에서는 지원 안 되지만 SQL에서는 꼭 넣어야 함
