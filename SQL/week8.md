# 1. PIVOT과 UNPIVOT

 - PIVOT: 행을 열로 바꾸고 지정된 칼럼의 각 행의 속성값들이 새로운 칼럼이 된다.

 - UNPIVOT: 열을 행으로 바꾸고 칼럼이 각 행의 속성값이 된다.

 - PIVOT을 하는 궁극적인 이유는 데이터를 잘 시각화하기 위함

- 이 과정에서 행을 열로 바꾸는게 좋으면 PIVOT이고 그 반대면 UNPIVOT이다.

- EX)계절별 연도별 데이터 구성

## 1-1. 문제풀이

```SQL
WITH RankedOccupations AS (
    SELECT 
        Name,
        Occupation,
        ROW_NUMBER() OVER (PARTITION BY Occupation ORDER BY Name) AS RowNumber
    FROM Occupations
)
SELECT 
    MAX(CASE WHEN Occupation = 'Doctor' THEN Name ELSE NULL END) AS Doctor,
    MAX(CASE WHEN Occupation = 'Professor' THEN Name ELSE NULL END) AS Professor,
    MAX(CASE WHEN Occupation = 'Singer' THEN Name ELSE NULL END) AS Singer,
    MAX(CASE WHEN Occupation = 'Actor' THEN Name ELSE NULL END) AS Actor
FROM RankedOccupations
GROUP BY RowNumber
ORDER BY RowNumber;
```
- 각 직업을 기준으로 이름을 정렬하고 각 행에 고유한 번호 부여

- MAX함수: 특정 직업의 이름만 가져오기

- CASE문: 특정직업에 해당하는 이름만 선택하고 아닌경우에 NULL을 반환

- 같은 ROWNUMBER을 가진 이름들을 각 직업 열로 정렬

# 2. 성능 최적화

1. 좌변을 연산하지 않을 것

- 예를들어 2024-06-01과 같은 날짜형 데이터에서 연도를 가져오고 싶을 때 SELECT YEAR(date)와 같은 쿼리를 사용한다면 데이터를 변형하는 연산을 수행하여 인덱스를 사용할 수 없게 되어 작업량이 늘어남

- 형태를 유지하면서 데이터를 찾아야 하는데, 2021년의 데이터를 가져오고 싶다면 1월1일부터 12월31의 데이터를 가져오면 변형을 하지 않고서도 가능

2. OR 대신 UNION 사용

- OR은 한 번의 스캔으로 모든 조건을 확인해야 하므로 불필요한 데이터까지 다량으로 검색하게 됨(여러 값을 동시에 찾아야 함)

- UNION은 각 조건에 대한 쿼리를 별도로 실행하고, 결과를 합쳐주기 때문에 쿼리를 독립적으로 최적화할 수 있다.

3. 필요한 ROW와 COLUMN만 선택하기

- 최대한 데이터의 수를 줄여서 필요한 부분만 가져오기

4. 분석함수를 활용하기

분석함수는 ROW별로 세부적인 계산을 가능하게 하는데, 집계함수와 달리 사전에 데이터를 그룹화할 필요가 없어서 실행시간을 크게 단축시킨다.

5. 와일드 카드는 끝에 작성하기

- 와일드카드가 앞에 있으면 모든 가능한 문자열 조합을 일일히 검색하기 때문에 비효율적이다.

6. 계산값을 미리 저장해두고, 나중에 조회할 것

- 계산값을 별도의 테이블에 저장하여 필요할 때 바로 값을 가져오기

# 2-1. 문제풀이

```SQL
UPDATE customers c
SET
    avg_order_value = (
        SELECT AVG(od.quantity * od.unit_price)
        FROM order_details od
        JOIN orders o ON od.order_id = o.order_id
        WHERE o.customer_id = c.customer_id
    ),
    total_spent = (
        SELECT SUM(od.quantity * od.unit_price)
        FROM order_details od
        JOIN orders o ON od.order_id = o.order_id
        WHERE o.customer_id = c.customer_id
    ),
    num_orders = (
        SELECT COUNT(DISTINCT o.order_id)
        FROM orders o
        WHERE o.customer_id = c.customer_id
    ),
    repurchase_rate = (
        SELECT
            COUNT(DISTINCT CASE WHEN od.product_id IN (
                SELECT product_id
                FROM order_details
                JOIN orders o ON order_details.order_id = o.order_id
                WHERE o.customer_id = c.customer_id
                GROUP BY order_details.product_id
                HAVING COUNT(order_details.product_id) > 1
            ) THEN od.product_id END) * 1.0 / COUNT(DISTINCT od.product_id)
        FROM order_details od
        JOIN orders o ON od.order_id = o.order_id
        WHERE o.customer_id = c.customer_id
    );
```

- 평균 주문 금액: order_details와 orders을 조인하여 주문 세부 정보를 가져오고 수량과 가격을 곱하여 금액을 구함

- 총 지출금액:
위와 거의 동일하되 sum을 사용

- 주문횟수:
distinct를 사용하여 중복되지 않은 주문의 개수를 셈

4. 재구매 비율:
- having count를 통해 특정 고객이 2번 이상 주문한 제품 목록을 가져오고 case when 을 통해 재구매한 제품을 카운트, 마지막으로 재구매한 제품 수를 전체 재품수로 나누어 비율을 계산

추가질문 1: 정답 쿼리에서 repurchase_rate를 구할 때 사용한 HAVING COUNT(order_details.product_id) > 1의 의미는 무엇인가요? 같은 제품을 2번 이상 구매한 경우를 필터링 하기 위해

2. 이 문제에서 사용될 수 있는 성능을 최적화하기 위한 방법은 무엇일까요? 적절한 인덱스를 생성해서 고객별 데이터를 빠르게 조회할 수 있도록 하고, 중간 테이블을 만들어서 재구매한 재품 리스트를 미리 계산하여 사용할 수 있다.또한 distint는 추가적인 정렬 작업을 필요로 하기 때문에 성능 저하를 유발하므로 그룹화와 카운트를 사용할 수 있다.