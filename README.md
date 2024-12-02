# 🛫 AirReservationService
> **항공편 예약 시스템**

---
## 🗓 제작 기간 
- 2024년 11월 28일 ~ 12월 1일 (총 4일)
<br></br>
## 📝구현 상세
- 로그인(관리자/일반회원), 회원가입, ID찾기
- 항공편 데이터(CRUD), 여객기 데이터(CRUD), 여행국 데이터(CRUD), 국가 간 비행거리 및 소요시간 자동 계산 트리거 구현
<br></br>
## 🛠 개발 환경
- `Java`
- `JDK-21.0.4`
- **IDE** : Eclipse
- **Database** : Oracle DB(11g Express Edition)
- **Developer** : sqldeveloper
<br></br>
## 📌주요 기능
1. 사용자가 항공편 예약을 쉽게 할 수 있는 시스템 구현
2. 관리자 모드를 통해 고객, 항공기, 국가, 항공편 관리 가능
3. 사용자와 관리자가 모두 사용할 수 있는 직관적인 UI 제공
4. **MVC 아키텍처**를 통한 코드 구조화
<br></br>
## 💻 실행화면
<details>
  <summary>로그인 화면</summary>
  <img src="일반고객 로그인.png" alt="sample image">
</details>
<br></br>
<details>
  <summary>항공편 예매 화면</summary>
    <img src="항공편예매.png" alt="sample image">
</details>
<br></br>
<details>
  <summary>회원가입 화면</summary>
    <img src="회원가입.png" alt="sample image">
</details>
<br></br>
<details>
  <summary>고객정보 출력 화면</summary>
    <img src="고객정보출력.png" alt="sample image">
</details>
<br></br>
## 🚧 **핵심 트러블 슈팅**

- _**예약 시 고객 예약 합계 증가 문제**_
  - 예약 테이블에 데이터가 추가될 때마다 고객 예약 합계를 자동으로 증가시키는 트리거를 구현하여 문제 해결
  - 트리거 코드:
    ```sql
    CREATE OR REPLACE TRIGGER COUNT_UP_TRG
    AFTER INSERT ON BOOKING
    FOR EACH ROW
    BEGIN
        UPDATE CUSTOMER SET C_COUNT = C_COUNT + 1 WHERE NO = :NEW.CUSTOMER_NO;
    END;
    /
    ```

    <br></br>

- _**예약 삭제 시 고객 예약 합계 감소 문제**_
  - 예약 테이블에서 데이터가 삭제될 때 고객 예약 합계를 감소시키는 트리거를 추가하여 문제 해결
  - 트리거 코드:
    ```sql
    CREATE OR REPLACE TRIGGER COUNT_DOWN_TRG
    AFTER DELETE ON BOOKING
    FOR EACH ROW
    BEGIN
        UPDATE CUSTOMER SET C_COUNT = C_COUNT - 1 WHERE NO = :OLD.CUSTOMER_NO;
    END;
    /
    ```
    
<br></br>

- _**항공기 추가 시 좌석 자동 생성 문제**_
  - 항공기 정보를 추가할 때 좌석 정보를 자동으로 생성하는 트리거를 작성하여 관리의 편의성 제공
  - 트리거 코드:
    ```sql
    CREATE OR REPLACE TRIGGER PLANE_INSERT_TRG
    AFTER INSERT ON PLANE
    FOR EACH ROW
    DECLARE
        X1 NUMBER;
        X2 NUMBER;
    BEGIN
        FOR i IN 0 .. :NEW.ROWX - 1 LOOP
            FOR j IN 1 .. :NEW.COLY LOOP
                X1 := ASCII('A') + TRUNC(i / 26);
                X2 := ASCII('A') + MOD(i, 26);
                INSERT INTO SEATS VALUES (
                    TO_CHAR((SELECT NVL(MAX(NO), 0) + 1 FROM SEATS), 'FM000000'),
                    :NEW.NO,
                    CHR(X1) || CHR(X2),
                    TO_CHAR(j, 'FM00')
                );
            END LOOP;
        END LOOP;
    END;
    /
    ```
<br></br>
- _**항공편 가격 및 도착 시간 자동 계산 문제**_
  - 국가 간 거리를 기준으로 항공편의 가격을 자동 계산하는 트리거를 구현하여 사용자가 입력할 필요 없이 자동으로 설정되도록 개선
  - 또한 출발 시간에 소요 시간을 더하여 도착 시간을 자동으로 계산하는 트리거 구현
  - 가격 및 도착 시간 자동 계산 트리거 코드:
    ```sql
    CREATE OR REPLACE TRIGGER FLIGHT_PRICE_TRIGGER
    BEFORE INSERT ON FLIGHT
    FOR EACH ROW
    DECLARE
        SHOUR NUMBER(4.2);
    BEGIN
        SELECT HOUR INTO SHOUR FROM COUNTRY C WHERE C.NO = :NEW.ARRIVAL_COUNTRY_NO;
        :NEW.PRICE := SHOUR * 150000;
    END;
    /
    
    CREATE OR REPLACE TRIGGER FLIGHT_ARRIVAL_TRIGGER
    BEFORE INSERT ON FLIGHT
    FOR EACH ROW
    DECLARE
        SHOUR NUMBER(4.2);
    BEGIN
        SELECT HOUR INTO SHOUR FROM COUNTRY C WHERE C.NO = :NEW.ARRIVAL_COUNTRY_NO;
        :NEW.ARRIVAL_HOUR := :NEW.DEPARTURE_HOUR + (SHOUR / 24);
    END;
    /
    ```
<br></br>
- _**예약 시 좌석 자동 배정 문제**_
  - 예약이 입력되거나 업데이트될 때 입력된 좌석에 맞춰 좌석 번호를 자동으로 업데이트하는 트리거 구현
  - 트리거 코드:
    ```sql
    CREATE OR REPLACE TRIGGER BOOKING_SEATS_TRG
    BEFORE INSERT OR UPDATE ON BOOKING
    FOR EACH ROW
    DECLARE
        SNO CHAR(6);
    BEGIN
        SELECT NO INTO SNO FROM SEATS 
        WHERE ROWX = SUBSTR(:NEW.SEAT, 1, 2) 
        AND COLY = SUBSTR(:NEW.SEAT, 3) 
        AND PLANE_NO = (SELECT PLANE_NO FROM FLIGHT WHERE NO = :NEW.FLIGHT_NO);
        :NEW.SEATS_NO := SNO;
    END;
    /
    ```
<br></br>
- _**예약 시 코드 자동 생성 문제**_
  - 예약 데이터를 삽입할 때 예약 코드(`CODE`)를 자동으로 생성하도록 트리거를 작성하여 관리 편의성 제공
  - 트리거 코드:
    ```sql
    CREATE OR REPLACE TRIGGER BOOKING_INSERT_TRG
    BEFORE INSERT ON BOOKING
    FOR EACH ROW
    BEGIN
        :NEW.CODE := :NEW.GROUP_NO || '-' || :NEW.CUSTOMER_NO || '-' || :NEW.FLIGHT_NO;
    END;
    /
    ```
<br></br>
## 💡 추가 구현
- **VIEW를 이용한 통합 정보 제공**
  - 모든 예약 정보를 한눈에 볼 수 있도록 여러 테이블을 조인한 `BOOKING_JOIN_ALL_VIEW` 뷰 작성
  - 뷰 정의 코드:
    ```sql
    CREATE OR REPLACE VIEW BOOKING_JOIN_ALL_VIEW AS
    SELECT DISTINCT B.CUSTOMER_NO, GROUP_NO, B.CODE, C.NAME, P.NAME AS PLANE_NAME, DC.NAME AS DEP_COUNTRY, AC.NAME AS ARV_COUNTRY, PRICE, DEPARTURE_HOUR, ARRIVAL_HOUR, SEAT, BOOKING_DATE
    FROM BOOKING B
    JOIN FLIGHT F ON B.FLIGHT_NO = F.NO
    JOIN CUSTOMER C ON B.CUSTOMER_NO = C.NO
    JOIN COUNTRY DC ON DC.NO = F.DEPARTURE_COUNTRY_NO
    JOIN COUNTRY AC ON AC.NO = F.ARRIVAL_COUNTRY_NO
    JOIN PLANE P ON P.NO = F.PLANE_NO;
    /
    ```
