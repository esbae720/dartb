# 1. ROOT 아이템 구하기
```sql
SELECT ITEM_INFO.ITEM_ID, ITEM_INFO.ITEM_NAME
FROM ITEM_INFO
JOIN ITEM_TREE ON ITEM_INFO.ITEM_ID = ITEM_TREE.ITEM_ID
WHERE ITEM_TREE.PARENT_ITEM_ID IS NULL
ORDER BY ITEM_INFO.ITEM_ID ASC;
```

- ROOT아이템을 찾기 위해선 parent item id가 결측치인 값들을 골라내는 것이 이 문제의 핵심이다.
- item_info와 item_tree를 조인하여 최종적인 결과값을 도출해낼 수 있다.

# 2. 노선 별 평균 역 사이 조회하기
```sql
SELECT
    ROUTE,
    CONCAT(ROUND(SUM(D_BETWEEN_DIST), 1), 'km') AS TOTAL_DIST.
    CONCAT(ROUND(AVG(D_BETWEEN_DIST), 2), 'km') AS AVERAGE_DIST
FROM
    SUBWAY_DISTANCE
GROUP BY
    ROUTE
ORDER BY
    SUM(D_BETWEEN_DIST) DESC;
```

- select 해줄때 concat함수를 써서 거리의 합과 km을 합쳐 주어 새로운 변수인 total_dist와 average_dist를 만드는게 핵심이다.

- groupby룰 통해 루트별로 묶어주면 문제가 풀린다.

# 3. 해비유저가 소유한 장소
```sql
SELECT ID, NAME, HOST_ID
FROM PLACES
WHERE HOST_ID IN (
    SELECT HOST_ID
    FROM PLACES
    GROUP BY HOST_ID
    HAVING COUNT(*) >= 2
)
ORDER BY ID;
```
- where절을 통해 조건을 정의하고 그 안에 서브쿼리를 두어  호스트 아이디로 그룹바이 한 후 하나의 호스트 아디이에 두 개 이상의 레코드가 있는 피쳐를 필터링 한다.

- having은 그룹바이 한 호스트 아이디에 대하여 필터링을 해주는 함수이다.

- 서브쿼리르 통해 레코드가 2개 이상인 데이터를 뽑아내는게 이 문제의 핵심    
