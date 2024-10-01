# 1. 조건에 맞는 사용자와 총 거래금액 조회하기
```SQL
SELECT 
    U.USER_ID,
    U.NICKNAME,  
    SUM(B.PRICE) AS TOTAL_SALES --이름 수정
FROM 
    USED_GOODS_BOARD B
JOIN 
    USED_GOODS_USER U
ON 
    B.WRITER_ID = U.USER_ID
WHERE 
    B.STATUS = 'DONE'  -- 거래가 완료된 것만 
GROUP BY 
    U.USER_ID, U.NICKNAME  
HAVING 
    TOTAL_SALES >= 700000  -- 총 거래 금액이 70만 원 이상인 사람만
ORDER BY 
    TOTAL_SALES ASC;  -- 총 거래 금액을 기준으로 오름차순 정렬 
```
- 서로 다른 이름의 칼럼인 WRITTER_ID와 USER_ID를 기준으로 데이터를 병합하는 것이 핵심
- USER_ID와 NICKNAME 별로 그룹바이 한 후 SELECT의 SUM을 활용하여 전체 판매량을 집계하는 것
- HAVING을 통해 그룹바이 한 결과의 조건을 다시 재지정

# 2. 업그레이드 할 수 없는 아이템 구하기
```SQL
SELECT 
    I.ITEM_ID, 
    I.ITEM_NAME, 
    I.RARITY
FROM 
    ITEM_INFO I
INNER JOIN 
    (
        SELECT T.ITEM_ID  -- PARENT_ITEM_ID가 NULL인 경우만 선택
        FROM ITEM_TREE T
        WHERE T.PARENT_ITEM_ID IS NULL
    ) AS Filter
ON 
    I.ITEM_ID = Filter.ITEM_ID  -- 필터링된 결과와 조인
ORDER BY 
    I.ITEM_ID DESC;
```
- 서브쿼리를 통해 부모가 없는, 즉 더이상 업그레이드 할 수 없는 피쳐를 먼저 선택한 후 INNER JOIN하여 ID가 겹치는 아이템에 대해서 ITEM_NAME과 RARITY를 가져오는 코드
- 이론적으로는 되어야? 맞지만 어째선지 자꾸 오류가 뜨는데 이유를 모르겠다.

# 3. 조건에 맞는 개발자 찾기
```SQL
SELECT 
    D.ID,
    D.EMAIL,
    D.FIRST_NAME,
    D.LAST_NAME
FROM 
    DEVELOPERS D
JOIN 
    SKILLCODES S
ON 
    (D.SKILL_CODE & S.CODE) = S.CODE  -- 비트 연산으로 스킬 확인
WHERE 
    S.NAME IN ('Python', 'C#')  -- Python 또는 C# 
ORDER BY 
    D.ID ASC;  -- ID 기준으로 오름차순 정렬
```
- 코드의 핵심은 ON을 활용한 비트 연산 부분이다.SKILL_CODE와 CODE를 비트 연산했을 때 같은 위치에 둘다 1이 있을경우(이진수에서) AND 연산은 1을 반환한다. 이런식으로 연산을 했을 때, CODE와 같다면 해당 스킬이 있다고 간주된다.