# 아이템 24 - 일관성 있는 별칭 사용하기

## 별칭은 타입스크립트가 타입을 좁히는 것을 방해한다. 따라서, 변수에 별칭을 사용할 때는 일관되게 사용해야한다.

```typescript
interface Coordinate {
  x: number;
  y: number;
}

interface BoundingBox {
  x: [number, number];
  y: [number, number];
}

interface Polygon {
  exterior: Coordinate[];
  holes: Coordinate[];
  bbox?: BoundingBox;
}

function isPointInPolygon(polygon : Polygon, pt : Coordinate){
	const box = polygon.bbox; // undefined or Polygon 
    if(polygon.bbox){
    	// 위의 조건을 통과했으므로, 여기서부터 polygon.bbox의 타입은 Polygon으로 추론됨
        if(pt.x < box.x[0]) || pt.x > box.x[1]){ //box는 여전히 undefined로 추론될 수 있기에 오류 발생
        	//....
            return false;
         }
    }
}
```
따라서 box 별칭을 사용하던가 polygon.bbox를 사용하던가 둘 중 하나만 사용해서 일관성을 유지하도록 코드를 바꾸어야한다.

 

polygon.bbox를 사용하면 코드가 너무 길어지고, box 변수를 사용하면 하나의 값을 표현하는데에 2가지가 쓰인다.(box, polygon.bbox)

 

<b>비구조화 문법(destructing)을 사용하면 더 간결하게 표현할 수 있다.</b>

```typescript
function isPointInPolygon(polygon : Polygon, pt : Coordinate){
	const {bbox} = polygon;  //destructing..
    if(bbox){
        if(pt.x < bbox.x[0]) || pt.x > bbox.x[1]){ 
        	//....
            return false;
         }
    }
}
```
속성보다 지역변수를 활용하면 타입 정제를 더 믿을 수 있다.

```typescript
function fn(p : Polygon){ //Polygon의 bbox를 제거 }

// 속성을 이용하는 경우
polygon.bbox // BoundingBox | undefined
if(polygon.bbox){
	polygon.bbox // BoundingBox
    fn(polygon); //polygon.bbox가 undefined으로 변경됨
    polygon.bbox // 하지만 여전히 BoundingBox로 추론됨 
}

// 지역변수을 이용하는 경우
const {bbox} = polygon; // BoundingBox | undefined
if(bbox){
    fn(polygon); //polygon.bbox가 undefined가 됨
    bbox // BoundingBox | undefined로 추론. bbox의 타입은 그대로 유지됨
}
```