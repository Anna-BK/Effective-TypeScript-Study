### 🎯 아이템 31 타입 주변에 null 값 배치하기

### 📕 strictNullChecks 설절은 통한 null 또는 undefined 체크.

🔖 최소값 최대값 계산
```javascript
function extent(nums: number[]) {
    let min, max;
    for(const num of nums) {
        if(!min) {
            min = num;
            max = num;
        }else {
            min = Math.min(min, num);
            max = Math.max(max, num);   // 오류 발생 코드.
        }
    }
    return [min, max];
}
```
최소값이나 최대값이 0인 경우 잘못된 결과를 반환할 수 있다.
nums 배열이 비어 있다면 함수는 [undefined, undefined]를 반환 한다.

🔖 단일 객체 사용 방식
```javascript
function extent(nums: number[]){
    let result: [number, number] | null = null;
    for(const num of nums) {
        if(!result) {
            result = [num, num];
        }else {
            result = [Math.min(num, result[0]), Math.max(num, result[1])];
        }
    }
    return result;
}
```
###📌 요약
* 한 값의 null 여부가 다른 값의 null 여부에 암시적으로 관련되도록 설계하면 안 됩니다.
* API 작성 시에는 반환 타입을 큰 객체로 만들고 반환 타입 전체가 null이거나 null이 아니게 만들어야 합니다.
* 클래스를 만들 때는 필요한 모든 값이 준비되었을 때 생성하여 null이 존재하지 않도록 하는 것이 좋습니다.
* strictNullChecks를 설정하면 코드에 많은 오류가 표시되겠지만, null 값과 관련된 문제점을 찾아낼 수 있기 때문에 반드시 필요합니다.
