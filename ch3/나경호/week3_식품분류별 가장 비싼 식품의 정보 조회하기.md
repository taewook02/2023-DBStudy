# [식품분류별 가장 비싼 식품의 정보 조회하기](https://school.programmers.co.kr/learn/courses/30/lessons/131116)

**MySQL**

### **문제 설명**

다음은 식품의 정보를 담은 `FOOD_PRODUCT` 테이블입니다. `FOOD_PRODUCT` 테이블은 다음과 같으며 `PRODUCT_ID`, `PRODUCT_NAME`, `PRODUCT_CD`, `CATEGORY`, `PRICE`는 식품 ID, 식품 이름, 식품코드, 식품분류, 식품 가격을 의미합니다.

| Column name | Type | Nullable |
| --- | --- | --- |
| PRODUCT_ID | VARCHAR(10) | FALSE |
| PRODUCT_NAME | VARCHAR(50) | FALSE |
| PRODUCT_CD | VARCHAR(10) | TRUE |
| CATEGORY | VARCHAR(10) | TRUE |
| PRICE | NUMBER | TRUE |

---

### 문제

`FOOD_PRODUCT` 테이블에서 식품분류별로 가격이 제일 비싼 식품의 분류, 가격, 이름을 조회하는 SQL문을 작성해주세요. 이때 식품분류가 '과자', '국', '김치', '식용유'인 경우만 출력시켜 주시고 결과는 식품 가격을 기준으로 내림차순 정렬해주세요.

---

### 예시

`FOOD_PRODUCT` 테이블이 다음과 같을 때

| PRODUCT_ID | PRODUCT_NAME | PRODUCT_CD | CATEGORY | PRICE |
| --- | --- | --- | --- | --- |
| P0018 | 맛있는고추기름 | CD_OL00008 | 식용유 | 6100 |
| P0019 | 맛있는카놀라유 | CD_OL00009 | 식용유 | 5100 |
| P0020 | 맛있는산초유 | CD_OL00010 | 식용유 | 6500 |
| P0021 | 맛있는케첩 | CD_SC00001 | 소스 | 4500 |
| P0022 | 맛있는마요네즈 | CD_SC00002 | 소스 | 4700 |
| P0039 | 맛있는황도 | CD_CN00008 | 캔 | 4100 |
| P0040 | 맛있는명이나물 | CD_CN00009 | 캔 | 3500 |
| P0041 | 맛있는보리차 | CD_TE00010 | 차 | 3400 |
| P0042 | 맛있는메밀차 | CD_TE00001 | 차 | 3500 |
| P0099 | 맛있는맛동산 | CD_CK00002 | 과자 | 1800 |

SQL을 실행하면 다음과 같이 출력되어야 합니다.

| CATEGORY | MAX_PRICE | PRODUCT_NAME |
| --- | --- | --- |
| 식용유 | 6500 | 맛있는산초유 |
| 과자 | 1800 | 맛있는맛동산 |

---
## 문제풀이
```SQL

/* 틀린 풀이 */ 
SELECT CATEGORY, MAX(PRICE) AS MAX_PRICE, PRODUCT_NAME
FROM FOOD_PRODUCT
WHERE CATEGORY IN ('과자', '국', '김치', '식용유')
GROUP BY CATEGORY
ORDER BY MAX_PRICE DESC
```

- 틀린 이유: 
1. MAX(PRICE)를 바로 SELECT 할 경우 가장 높은 가격을 뽑기만 하고, 같은 행에 있는 값들은 관련이 없는 값이 SELECT 된다.
2. 때문에 가장 큰 값을 식별할 수 있는 값으로 서브 쿼리를 통해 가져오고 그 값과 그 값에 해당하는 행의  CATEGORY, PRODUCT_NAME을 가져오도록 한다.

- 아래 사진에서 확인할 수 있듯 전혀 다른 PRODUCT_ID를 가지게 된다는게 문제이다.
  
<img width="738" alt="스크린샷 2023-07-18 오후 1 09 27" src="https://github.com/Hoya324/2023-DBStudy/assets/96857599/1e82748e-6a06-44e8-8472-dd248b9d278f">

<img width="728" alt="스크린샷 2023-07-18 오후 1 10 24" src="https://github.com/Hoya324/2023-DBStudy/assets/96857599/57578174-9009-4e0c-8c69-000d9af29a49">

```sql
/* 정답은 맞지만 잘못된 풀이 */
SELECT CATEGORY, PRICE AS MAX_PRICE, PRODUCT_NAME
FROM FOOD_PRODUCT
WHERE CATEGORY IN ('과자', '국', '김치', '식용유') AND
    PRICE IN (SELECT MAX(PRICE)
    FROM FOOD_PRODUCT
    GROUP BY CATEGORY)
ORDER BY MAX_PRICE DESC
```

- 잘못된 이유
1. 가장 높은 가격의 PRICE를 식별 값으로 사용했을 때 만약 다른 CATEGORY의 값 중 같은 값이 있다면 문제가 생길 수 있다.
2. 이를 방지하기 위해 서브쿼리에서 CATEGORY를 함께 SELECT 해주었다.

<img width="736" alt="스크린샷 2023-07-18 오후 1 12 04" src="https://github.com/Hoya324/2023-DBStudy/assets/96857599/e02a5aa7-9a76-4fa0-8de8-862d866b2db6">

```sql
/* 옳은 풀이 */
SELECT CATEGORY, PRICE AS MAX_PRICE, PRODUCT_NAME
FROM FOOD_PRODUCT
WHERE CATEGORY IN ('과자', '국', '김치', '식용유') AND
    (CATEGORY, PRICE) IN (SELECT CATEGORY, MAX(PRICE)
    FROM FOOD_PRODUCT
    GROUP BY CATEGORY)
ORDER BY MAX_PRICE DESC
```
