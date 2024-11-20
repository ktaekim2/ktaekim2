# 예약 시스템 설계

## 슬롯 기반 예약 관리(slot-based) vs 동적 예약 관리(on-demand)
이번 예약 시스템 설계에서 고려한 두 가지 방법론이다.

### 슬롯 기반 예약 관리
미리 단위별 예약 가능한 row를 생성하는 방식.
- 장점
  - 데이터 상태 관리가 명확함
- 단점
  - 저장 공간과 성능 부담
  - 유연성 낮음(시간 단위 고정)

### 동적 예약 관리
예약 시 동적으로 row를 생성하고 관리하는 방식.
- 장점
  - 데이터 최소화
  - 유연성 높음(시간 단위, 조건 등 변경 가능)
- 단점
  - 예약 가능 여부 판단 로직 복잡함
  - 동시성 충돌 관리 필요

## 그 외 예약 설계 방법론
- 캘린더 기반(calendar-based)
- 큐 기반(queue-based)
- 경매 기반(auction-based)
- 토큰 기반(token-based)
- 우선순위 기반(priority-based)
- 배치 처리(batch processing)
- 그래프 기반(graph-based)


## 동적 예약 관리에서 동시성 충돌 관리(트랜잭션 관리)
예약 시스템의 조건 등을 관리하는 테이블의 쓰기 발생 시 해당 레코드의 비관적 잠금(Pessimistic Lock)을 하여 다른 트랜잭션 접근을 막음.
데드락 방지를 위해 타임아웃 5초 설정하였음.

```java
    // service
    @Override
    @Transactional(timeout = 5)
    public ReservationTargetScheduleDTO updateSchedule(Long scheduleSeq, ReservationTargetScheduleDTO scheduleDTO) {
        ReservationTargetScheduleEntity entity = scheduleRepository.lockReservationTargetSchedule(scheduleSeq, "N")
                .orElseThrow(() -> new NoSuchElementException("해당 일정을 찾을 수 없습니다."));
        
        // 수정 로직...
    }

    // repository
    @Override
    @Transactional(timeout = 5)
    public Optional<ReservationTargetScheduleEntity> lockReservationTargetSchedule(Long scheduleSeq, String delYn) {
        QReservationTargetScheduleEntity targetScheduleEntity = QReservationTargetScheduleEntity.reservationTargetScheduleEntity;

        return Optional.ofNullable(
                queryFactory.selectFrom(targetScheduleEntity)
                        .where(
                                targetScheduleEntity.scheduleSeq.eq(scheduleSeq),
                                targetScheduleEntity.delYn.eq(delYn)
                        )
                        .setLockMode(LockModeType.PESSIMISTIC_WRITE) // 비관적 잠금 적용
                        .fetchOne()
        );
    }
```

동시 요청 테스트
```java
    @Test
    public void testPessimisticLocking() throws InterruptedException {
        Long scheduleSeq = 2L; // 테스트할 일정 시퀀스 ID

        Thread thread1 = new Thread(() -> {
            scheduleService.deleteSchedule(scheduleSeq);
        });

        Thread thread2 = new Thread(() -> {
            ReservationTargetScheduleDTO scheduleDTO = new ReservationTargetScheduleDTO();
            scheduleDTO.setStartDate(LocalDate.parse("2023-10-16"));
            scheduleDTO.setEndDate(LocalDate.parse("2023-10-21"));
            scheduleDTO.setDaysOfWeek("NNNNNNN");
            scheduleDTO.setAvailabilityType("UNAVAILABLE");
            // 기타 필드 설정
            scheduleDTO.setReason("두 번째 스레드 수정");

            scheduleService.updateSchedule(scheduleSeq, scheduleDTO);
        });

        thread1.start();
        Thread.sleep(100); // 첫 번째 스레드가 잠금을 획득하도록 약간 대기
        thread2.start();

        thread1.join();
        thread2.join();
    }
```