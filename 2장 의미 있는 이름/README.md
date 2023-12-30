# 2장 의미 있는 이름
<hr>

#### 의도를 분명히 밝혀라
> "의도가 분명하게 이름을 지으라"
```
public List<int[]> getThem() {
    List<int []> list1 = new ArrayList<int[]>();
    for (int[] x : theList)  
        if (x[0] == 4)  
            list1.add(x);
    return list1
}
```
* 위와 같은 정보가 드러나지 않는다. 하지만 정보 제공은 **충분히 가능했었다**
* 지뢰찾기 게임을 만든다고 가정할때 theList는 게임판이라는 사실을 알 수 있다 각 칸은 단순 배열로 표현, 배열에서 0번째 값은 칸 상태를 뜻함, 값 4는 깃발이 꽂힌 상태를 가리킨다고 하고 코드를 수정
```
public List<int[]> getFlaggedCells() {
    List<int[]> flaggedCells = new ArrayList<int[]>();
    for (int[] cell : gameBoard)
        if (cell[STATUS_VALUE] == FLAGGED)
            flaggedCells.add(cell);
    return flaggedCells;
}
```
* 코드의 단순성은 변하지 않았지만 코드는 더욱 명확해졌다
* 한 걸음 더 나아가 int 배열을 사용하는 대신 칸을 간단한 **클래스**로 만들어도 가능
```
public List<Cell> getFlaggedCells() {
    List<Cell> flaggedCells = new ArrayList<Cell>();
    for (Cell cell : gameBoard)
        if (cell.isFlagged())
            flaggedCells.add(cell);
    return flaggedCells;
}
```
<br>

#### 그릇된 정보를 피하라
* 나름대로 널리 쓰이는 의미가 있는 단어를 다른 의미로 사용해도 안 된다
* 서로 흡사한 이름을 사용하지 않도록 주의한다
* 이름으로 그릇된 정보를 제공하는 최악은 **소문자 L이나 대문자 O변수** 이다(소문자 L은 숫자 1처럼 보이고 대문자 O는 숫자 0처럼 보인다)
<br>

#### 의미 있게 구분하라
* 연속된 숫자를 덧붙이거나 불용어(ex : Info, Data는 a, an, the)를 추가하는 방식은 부적절
<br>

#### 발음하기 쉬운 이름을 사용하라
```
class DtaRcrd102 {
    private Date genymdhms;
    private Date modymdhms;
    private final String pszqint = "102";
    /* ... */
};
```
* 위의 코드보다 아래 코드가 지적인 대화가 가능함
```
class Customer {
    private Date generationTimestamp;
    private Date modificationTimestamp;
    private final String recordId = "102";
    /* ... */
};
```
<br>

#### 이름 생성
* **클래스(객체) 이름** : 명사나 명사구가 적합
* **메서드 이름** : 동사나 동사구가 적합
* 적절한 '프로그래머 용어'가 없다면 문제영역에서 이름을 가져온다

