# smart-parking-backend
Node.js기반 스마트 파킹 시스템을 위한 MQTT·MongoDB·REST API 기반 백엔드

# -# Smart Parking System - Backend (Server & Communication)

본 프로젝트는 YOLO 기반 차량 인식 결과를 실시간으로 수집하고,  
주차면 상태 및 불법 주정차 정보를 안정적으로 저장·가공하여  
모바일 애플리케이션(Flutter)에 제공하는 백엔드 시스템이다.

IoT 환경에 적합한 MQTT 기반 메시징과 MongoDB, REST API 구조를 적용하여  
실시간성, 확장성, 유지보수성을 동시에 확보하는 것을 목표로 한다.

---

## 1. 시스템 아키텍처

- Raspberry Pi (YOLO Detection)
- MQTT Publisher (Node.js)
- HiveMQ Cloud (Broker)
- MQTT Subscriber (Node.js)
- MongoDB Atlas
- REST API Server (Express)
- Flutter Client
<img width="2289" height="1396" alt="architecture" src="https://github.com/user-attachments/assets/8ebfc483-3e27-4d4e-94a9-146ecce511ef" />

---

## 2. 데이터 흐름 (Data Flow)

1. YOLO가 차량 및 불법 주정차를 탐지  
2. Publisher가 MQTT 메시지 발행  
3. HiveMQ Broker가 메시지 전달  
4. Subscriber가 수신 후 데이터 정제  
5. MongoDB에 저장  
6. REST API가 DB 조회  
7. Flutter 앱이 API 호출하여 화면 갱신  

![DataFlow](images/dataflow.png)

---

## 3. 데이터 모델

### latest (슬롯별 현재 상태)
```json
{
  "slot": 1,
  "status": 1,
  "confidence": 0.95,
  "updatedAt": "2025-01-01T12:00:00"
}
```
### changes (상태 변경 이력)
```
{
  "slot": 1,
  "status": 0,
  "changedAt": "2025-01-01T12:10:00"
}
```
illegal_latest (현재 불법 주정차 스냅샷)
```
{
  "count": 2,
  "cars": [
{ "id": 0, "x": 320, "y": 210, "duration": 30 }
  ]
}
```

4. 문제점
1) 실시간 데이터 전달 지연

HTTP 폴링 방식은 빈번한 요청으로 네트워크 비용 증가 및 지연 발생

2) 데이터베이스 저장 용량 증가

모든 MQTT 메시지를 저장하면 데이터가 급격히 증가

3) 클라이언트 MQTT 직접 연결의 복잡성

Flutter 앱에서 MQTT를 직접 처리하면 보안 및 관리 난이도 상승

5. 해결 방법
MQTT 기반 Pub/Sub 구조 도입

IoT 환경에 최적화된 경량 프로토콜 사용

Publisher와 Subscriber 분리

HiveMQ Cloud를 이용해 TLS 보안 및 연결 관리

MongoDB 컬렉션 분리 설계

latest : 현재 상태만 저장

changes : 변경 발생 시만 기록

illegal_latest : 불법 주정차 스냅샷

REST API 계층 추가

MQTT → DB → REST API → Client 구조

Flutter는 HTTP만 사용

6. 결과

주차 상태 실시간 반영

DB 저장 용량 감소

Flutter 앱 구조 단순화

기능 확장에 유리한 구조 확보

7. 사용 기술

Node.js / Express

MQTT / HiveMQ Cloud

MongoDB Atlas

AWS EC2

Flutter
