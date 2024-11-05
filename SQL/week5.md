# 1. 성분으로 구분한 아이스크림 총 주문량

```sql
SELECT 
    I.INGREDIENT_TYPE,
    SUM(F.TOTAL_ORDER) AS TOTAL_ORDER
FROM 
    FIRST_HALF F
JOIN 
    ICECREAM_INFO I ON F.FLAVOR = I.FLAVOR
GROUP BY 
    I.INGREDIENT_TYPE
ORDER BY 
    TOTAL_ORDER ASC;
```

1.
FIRST_HALF 테이블과 ICECREAM_INFO 테이블을 FLAVOR 필드를 기준으로 조인하여 아이스크림 맛에 대한 성분 유형 가져오기

2.
성분 유형(INGREDIENT_TYPE)별로 TOTAL_ORDER의 합계를 계산

3.
INGREDIENT_TYPE별로 그룹화하여 합계를 구하기

4.
TOTAL_ORDER 기준으로 오름차순으로 정렬

# 2. 즐겨찾기가 가장 많은 식당 정보 출력하기

```sql
SELECT 
    R.FOOD_TYPE,
    R.REST_ID,
    R.REST_NAME,
    R.FAVORITES
FROM 
    REST_INFO R
WHERE 
    R.FAVORITES = (
        SELECT MAX(FAVORITES)
        FROM REST_INFO
        WHERE FOOD_TYPE = R.FOOD_TYPE
    )
ORDER BY 
    R.FOOD_TYPE;
```

1.
FOOD_TYPE, REST_ID, REST_NAME, FAVORITES 컬럼을 REST_INFO 테이블에서 선택

2.
서브쿼리를 사용하여 각 음식 종류별로 FAVORITES가 가장 높은 레코드만 필터링 할 때, FOOD_TYPE별로 가장 높은 즐겨찾기 수를 MAX함수를 통해 필터링

3.
oreder by를 통해 food_type을 기준으로 정렬

# 3. 조건에 맞는 사원 정보 조회하기

```sql
WITH TotalScores AS (
    SELECT 
        G.EMP_NO,
        SUM(G.SCORE) AS TOTAL_SCORE
    FROM 
        HR_GRADE G
    WHERE 
        G.YEAR = 2022
    GROUP BY 
        G.EMP_NO
)
SELECT 
    T.TOTAL_SCORE AS SCORE,
    E.EMP_NO,
    E.EMP_NAME,
    E.POSITION,
    E.EMAIL
FROM 
    TotalScores T
JOIN 
    HR_EMPLOYEES E ON T.EMP_NO = E.EMP_NO
WHERE 
    T.TOTAL_SCORE = (SELECT MAX(TOTAL_SCORE) FROM TotalScores)
ORDER BY 
    E.EMP_NO;
```

1. 
with절을 사용해서 임시테이블을 만드는데 hr_grade 테이블에서 사원의 상반기와 하반기의 점수를 합산하여 total_score라는 새로운 열을 만든다.

2.
totalscores와 hr_employees테이블을 조인하여 hr_employees테이블에 있는 사원정보들을 가져온다.

3.
totalscores테이블에서 가장 높은 total_score을 가진 사원을 where문을 통해 선택한다.
